# Encrypt a Realm - Java SDK
## Overview
You can encrypt your realms to ensure that the data stored to disk can't be
read outside of your application. You encrypt the realm file on disk with AES-256 +
SHA-2 by supplying a 64-byte encryption key when opening a
realm.

Realm transparently encrypts and decrypts data with standard
[AES-256 encryption](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) using the
first 256 bits of the given 512-bit encryption key. Realm
uses the other 256 bits of the 512-bit encryption key to validate
integrity using a [hash-based message authentication code
(HMAC)](https://en.wikipedia.org/wiki/HMAC).

> Warning:
> Do not use cryptographically-weak hashes for realm encryption keys.
For optimal security, we recommend generating random rather than derived
encryption keys.
>

## Considerations
The following are key impacts to consider when encrypting a realm.

### Storing & Reusing Keys
You **must** pass the same encryption key to `RealmConfiguration.Builder.encryptionKey()` each
time you open the realm.
If you don't provide a key or specify the wrong key for an encrypted
realm, the Realm SDK throws an error.

Apps should store the encryption key in the
[Android KeyStore](https://developer.android.com/training/articles/keystore.html) so
that other apps cannot read the key.

### Performance Impact
Reads and writes on encrypted realms can be up to 10% slower than unencrypted realms.

### Accessing an Encrypted Realm from Multiple Processes
> Version changed: 10.14.0

Starting with Realm Java SDK version 10.14.0, Realm supports opening
the same encrypted realm in multiple processes.

If your app uses Realm Java SDK version 10.14.0 or earlier, attempting to
open an encrypted realm from multiple processes throws this error:
`Encrypted interprocess sharing is currently unsupported.`

## Example
The following steps describe the recommended way to use the
[Android KeyStore](https://developer.android.com/training/articles/keystore.html) for encryption with
Realm:

1. Generate an asymmetric RSA key that Android can securely store and
retrieve using the Android KeyStore. Versions M and above require user PIN or fingerprint to unlock
the KeyStore.
2. Generate a symmetric key (AES) you can use to encrypt the realm.
3. Encrypt the symmetric AES key using your private RSA key.
4. Store the encrypted AES key on filesystem (in a
`SharedPreferences`, for example).

When you need to use your encrypted realm:

1. Retrieve your encrypted AES key.
2. Decrypt your encrypted AES key using the public RSA key.
3. Use the decrypted AES key in the `RealmConfiguration` to open the
encrypted realm.

> Seealso:
> For an end-to-end example of storing and reusing encryption keys, see
the [store_password](https://github.com/realm/realm-java/tree/feature/example/store_password/examples/StoreEncryptionPassword) example project, which demonstrates the
fingerprint API.
>

### Generate and Store an Encryption Key
The following code demonstrates how to securely generate and store an
encryption key for a realm:

#### Java

```java
// Create a key to encrypt a realm and save it securely in the keystore
public byte[] getNewKey() {
    // open a connection to the android keystore
    KeyStore keyStore;
    try {
        keyStore = KeyStore.getInstance("AndroidKeyStore");
        keyStore.load(null);
    } catch (KeyStoreException | NoSuchAlgorithmException
            | CertificateException | IOException e) {
        Log.v("EXAMPLE", "Failed to open the keystore.");
        throw new RuntimeException(e);
    }

    // create a securely generated random asymmetric RSA key
    byte[] realmKey = new byte[Realm.ENCRYPTION_KEY_LENGTH];
    new SecureRandom().nextBytes(realmKey);

    // create a cipher that uses AES encryption -- we'll use this to encrypt our key
    Cipher cipher;
    try {
        cipher = Cipher.getInstance(KeyProperties.KEY_ALGORITHM_AES
                + "/" + KeyProperties.BLOCK_MODE_CBC
                + "/" + KeyProperties.ENCRYPTION_PADDING_PKCS7);
    } catch (NoSuchAlgorithmException | NoSuchPaddingException e) {
        Log.e("EXAMPLE", "Failed to create a cipher.");
        throw new RuntimeException(e);
    }

    // generate secret key
    KeyGenerator keyGenerator;
    try {
        keyGenerator = KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES,
                "AndroidKeyStore");
    } catch (NoSuchAlgorithmException | NoSuchProviderException e) {
        Log.e("EXAMPLE", "Failed to access the key generator.");
        throw new RuntimeException(e);
    }
    KeyGenParameterSpec keySpec = new KeyGenParameterSpec.Builder(
            "realm_key",
            KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
            .setUserAuthenticationRequired(true)
            .setUserAuthenticationValidityDurationSeconds(
                   AUTH_VALID_DURATION_IN_SECOND)
            .build();
    try {
        keyGenerator.init(keySpec);
    } catch (InvalidAlgorithmParameterException e) {
        Log.e("EXAMPLE", "Failed to generate a secret key.");
        throw new RuntimeException(e);
    }
    keyGenerator.generateKey();

    // access the generated key in the android keystore, then
    // use the cipher to create an encrypted version of the key
    byte[] initializationVector;
    byte[] encryptedKeyForRealm;
    try {
        SecretKey secretKey =
                (SecretKey) keyStore.getKey("realm_key", null);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        encryptedKeyForRealm = cipher.doFinal(realmKey);
        initializationVector = cipher.getIV();
    } catch (InvalidKeyException | UnrecoverableKeyException
            | NoSuchAlgorithmException | KeyStoreException
            | BadPaddingException | IllegalBlockSizeException e) {
        Log.e("EXAMPLE", "Failed encrypting the key with the secret key.");
        throw new RuntimeException(e);
    }

    // keep the encrypted key in shared preferences
    // to persist it across application runs
    byte[] initializationVectorAndEncryptedKey =
            new byte[Integer.BYTES +
                    initializationVector.length +
                    encryptedKeyForRealm.length];
    ByteBuffer buffer = ByteBuffer.wrap(initializationVectorAndEncryptedKey);
    buffer.order(ByteOrder.BIG_ENDIAN);
    buffer.putInt(initializationVector.length);
    buffer.put(initializationVector);
    buffer.put(encryptedKeyForRealm);
    activity.getSharedPreferences("realm_key", Context.MODE_PRIVATE).edit()
            .putString("iv_and_encrypted_key",
                    Base64.encodeToString(initializationVectorAndEncryptedKey, Base64.NO_WRAP))
            .apply();

    return realmKey; // pass to a realm configuration via encryptionKey()
}

```

#### Kotlin

```kotlin
// Create a key to encrypt a realm and save it securely in the keystore
fun getNewKey(): ByteArray {
    // open a connection to the android keystore
    val keyStore: KeyStore
    try {
        keyStore = KeyStore.getInstance("AndroidKeyStore")
        keyStore.load(null)
    } catch (e: Exception) {
        Log.v("EXAMPLE", "Failed to open the keystore.")
        throw RuntimeException(e)
    }

    // create a securely generated random asymmetric RSA key
    val realmKey = ByteArray(Realm.ENCRYPTION_KEY_LENGTH)
    SecureRandom().nextBytes(realmKey)

    // create a cipher that uses AES encryption -- we'll use this to encrypt our key
    val cipher: Cipher
    cipher = try {
        Cipher.getInstance(KeyProperties.KEY_ALGORITHM_AES
                + "/" + KeyProperties.BLOCK_MODE_CBC
                + "/" + KeyProperties.ENCRYPTION_PADDING_PKCS7)
    } catch (e: Exception) {
        Log.e("EXAMPLE", "Failed to create a cipher.")
        throw RuntimeException(e)
    }

    // generate secret key
    val keyGenerator: KeyGenerator
    keyGenerator = try {
        KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES,
                "AndroidKeyStore")
    } catch (e: NoSuchAlgorithmException) {
        Log.e("EXAMPLE", "Failed to access the key generator.")
        throw RuntimeException(e)
    }
    val keySpec = KeyGenParameterSpec.Builder(
            "realm_key",
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
            .setUserAuthenticationRequired(true)
            .setUserAuthenticationValidityDurationSeconds(
                   AUTH_VALID_DURATION_IN_SECOND)
            .build()
    try {
        keyGenerator.init(keySpec)
    } catch (e: InvalidAlgorithmParameterException) {
        Log.e("EXAMPLE", "Failed to generate a secret key.")
        throw RuntimeException(e)
    }
    keyGenerator.generateKey()

    // access the generated key in the android keystore, then
    // use the cipher to create an encrypted version of the key
    val initializationVector: ByteArray
    val encryptedKeyForRealm: ByteArray
    try {
        val secretKey = keyStore.getKey("realm_key", null) as SecretKey
        cipher.init(Cipher.ENCRYPT_MODE, secretKey)
        encryptedKeyForRealm = cipher.doFinal(realmKey)
        initializationVector = cipher.iv
    } catch (e: Exception) {
        Log.e("EXAMPLE", "Failed encrypting the key with the secret key.")
        throw RuntimeException(e)
    }

    // keep the encrypted key in shared preferences
    // to persist it across application runs
    val initializationVectorAndEncryptedKey = ByteArray(Integer.BYTES +
            initializationVector.size +
            encryptedKeyForRealm.size)
    val buffer = ByteBuffer.wrap(initializationVectorAndEncryptedKey)
    buffer.order(ByteOrder.BIG_ENDIAN)
    buffer.putInt(initializationVector.size)
    buffer.put(initializationVector)
    buffer.put(encryptedKeyForRealm)
    activity!!.getSharedPreferences("realm_key", Context.MODE_PRIVATE).edit()
            .putString("iv_and_encrypted_key",
                    Base64.encodeToString(initializationVectorAndEncryptedKey, Base64.NO_WRAP))
            .apply()
    return realmKey // pass to a realm configuration via encryptionKey()
}

```

### Access an Existing Encryption Key
The following code demonstrates how to access and decrypt a securely
stored encryption key for a realm:

#### Java

```java
// Access the encrypted key in the keystore, decrypt it with the secret,
// and use it to open and read from the realm again
public byte[] getExistingKey() {
    // open a connection to the android keystore
    KeyStore keyStore;
    try {
        keyStore = KeyStore.getInstance("AndroidKeyStore");
        keyStore.load(null);
    } catch (KeyStoreException | NoSuchAlgorithmException
            | CertificateException | IOException e) {
        Log.e("EXAMPLE", "Failed to open the keystore.");
        throw new RuntimeException(e);
    }

    // access the encrypted key that's stored in shared preferences
    byte[] initializationVectorAndEncryptedKey = Base64.decode(activity
            .getSharedPreferences("realm_key", Context.MODE_PRIVATE)
            .getString("iv_and_encrypted_key", null), Base64.DEFAULT);
    ByteBuffer buffer = ByteBuffer.wrap(initializationVectorAndEncryptedKey);
    buffer.order(ByteOrder.BIG_ENDIAN);

    // extract the length of the initialization vector from the buffer
    int initializationVectorLength = buffer.getInt();
    // extract the initialization vector based on that length
    byte[] initializationVector = new byte[initializationVectorLength];
    buffer.get(initializationVector);
    // extract the encrypted key
    byte[] encryptedKey = new byte[initializationVectorAndEncryptedKey.length
            - Integer.BYTES
            - initializationVectorLength];
    buffer.get(encryptedKey);

    // create a cipher that uses AES encryption to decrypt our key
    Cipher cipher;
    try {
        cipher = Cipher.getInstance(KeyProperties.KEY_ALGORITHM_AES
                + "/" + KeyProperties.BLOCK_MODE_CBC
                + "/" + KeyProperties.ENCRYPTION_PADDING_PKCS7);
    } catch (NoSuchAlgorithmException | NoSuchPaddingException e) {
        Log.e("EXAMPLE", "Failed to create cipher.");
        throw new RuntimeException(e);
    }

    // decrypt the encrypted key with the secret key stored in the keystore
    byte[] decryptedKey;
    try {
        final SecretKey secretKey =
                (SecretKey) keyStore.getKey("realm_key", null);
        final IvParameterSpec initializationVectorSpec =
                new IvParameterSpec(initializationVector);
        cipher.init(Cipher.DECRYPT_MODE, secretKey, initializationVectorSpec);
        decryptedKey = cipher.doFinal(encryptedKey);
    } catch (InvalidKeyException e) {
        Log.e("EXAMPLE", "Failed to decrypt. Invalid key.");
        throw new RuntimeException(e);
    } catch (UnrecoverableKeyException | NoSuchAlgorithmException
            | BadPaddingException | KeyStoreException
            | IllegalBlockSizeException | InvalidAlgorithmParameterException e) {
        Log.e("EXAMPLE",
                "Failed to decrypt the encrypted realm key with the secret key.");
        throw new RuntimeException(e);
    }
    return decryptedKey; // pass to a realm configuration via encryptionKey()
}

```

#### Kotlin

```kotlin
// Access the encrypted key in the keystore, decrypt it with the secret,
// and use it to open and read from the realm again
fun getExistingKey(): ByteArray {
    // open a connection to the android keystore
    val keyStore: KeyStore
    try {
        keyStore = KeyStore.getInstance("AndroidKeyStore")
        keyStore.load(null)
    } catch (e: Exception) {
        Log.e("EXAMPLE", "Failed to open the keystore.")
        throw RuntimeException(e)
    }

    // access the encrypted key that's stored in shared preferences
    val initializationVectorAndEncryptedKey = Base64.decode(activity
            ?.getSharedPreferences("realm_key", Context.MODE_PRIVATE)
            ?.getString("iv_and_encrypted_key", null), Base64.DEFAULT)
    val buffer = ByteBuffer.wrap(initializationVectorAndEncryptedKey)
    buffer.order(ByteOrder.BIG_ENDIAN)

    // extract the length of the initialization vector from the buffer
    val initializationVectorLength = buffer.int
    // extract the initialization vector based on that length
    val initializationVector = ByteArray(initializationVectorLength)
    buffer[initializationVector]
    // extract the encrypted key
    val encryptedKey = ByteArray(initializationVectorAndEncryptedKey.size
            - Integer.BYTES
            - initializationVectorLength)
    buffer[encryptedKey]

    // create a cipher that uses AES encryption to decrypt our key
    val cipher: Cipher
    cipher = try {
        Cipher.getInstance(KeyProperties.KEY_ALGORITHM_AES
                + "/" + KeyProperties.BLOCK_MODE_CBC
                + "/" + KeyProperties.ENCRYPTION_PADDING_PKCS7)
    } catch (e: Exception) {
        Log.e("EXAMPLE", "Failed to create cipher.")
        throw RuntimeException(e)
    }

    // decrypt the encrypted key with the secret key stored in the keystore
    val decryptedKey: ByteArray
    decryptedKey = try {
        val secretKey = keyStore.getKey("realm_key", null) as SecretKey
        val initializationVectorSpec = IvParameterSpec(initializationVector)
        cipher.init(Cipher.DECRYPT_MODE, secretKey, initializationVectorSpec)
        cipher.doFinal(encryptedKey)
    } catch (e: InvalidKeyException) {
        Log.e("EXAMPLE", "Failed to decrypt. Invalid key.")
        throw RuntimeException(e)
    } catch (e: Exception ) {
                Log.e("EXAMPLE",
                        "Failed to decrypt the encrypted realm key with the secret key.")
                throw RuntimeException(e)
    }
    return decryptedKey // pass to a realm configuration via encryptionKey()
}

```
