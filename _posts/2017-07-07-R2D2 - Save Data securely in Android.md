---
layout: post
title:  "R2D2 - Save Data securely in Android"
date:   2017-07-06 6:11:15
author: shubham
categories: Technical Android
---

R2D2 is an open source android library which helps the user to encrypt and decrypt sensitive data.

#### Problem
In one of our app we needed to store user credentials so that user don’t have to type username and password every time he wants to login and just type in a pin code which can be used to get username and password. SharedPreference was not an option as information from the SharedPreference can be easily extracted by rooting the phone.

#### Solution
R2D2 uses android key store to store the key used for encrypting and decrypting the data thus making it impossible for anyone to retrieve the sensitive data.
Since api for encryption varies widely for different versions of android it is difficult and time consuming to implement something which is secure over all the versions of android above api level 16(*anyhow who uses api below 16*) So we have created a library which takes care of all the nitty-gritty technicalities of encryption so that you can focus on the main aspects of the app.

#### Integrating in your app
It is very simple to use, you just need to initialize R2D2 and call the encryptData and decryptData function and you can enjoy your sound sleep knowing that your data is safe.

For initializing you need to pass the context and a keyAlias. **It is highly recommended that you get the keyAlias either from the user (may be in form of pin) or from the api.**

##### Initializing R2D2
{% highlight java %}
R2D2 r2d2 = new R2D2(context, keyAlias);
{% endhighlight %}

##### For Encryption
{% highlight java %}
void setPassword(String password) {
        String encrypted = r2d2.encryptData(password);
        if (encrypted != null && !encrypted.equalsIgnoreCase("")) {
            password = encrypted;
        }
        SharedPreferences.Editor editor = preferences.edit();
        editor.putString(KEY_PASSWORD, password);
        editor.apply();
    }
{% endhighlight %}
##### For Decryption
{% highlight java %}
String getPassword() {
        String password = preferences.getString(KEY_PASSWORD, null);
        String decrypted = r2d2.decryptData(password);
        if (decrypted != null && !decrypted.equalsIgnoreCase("")) {
            password = decrypted;
        }
        return password;
    }
{% endhighlight %}

##### What's going on in the background
The android KeyStore handles the tasks like random key generation and securely storing them. It acts like a secure container.
Now depending on the API version, the sensitive information is handled accordingly.

For **android versions 23 and higher**, KeyGenParameterSpec API is used. Random AES keys are generated using the API which can be used for encrypting and decrypting the data. It uses same key for encryption and decryption. The data to be secured is encrypted using the key retrieved from the KeyStore, and then the encrypted data is stored in Shared Preferences. When the secret data needs to be retrieved, the encrypted data from the Preferences can be decrypted to plain text using the key stored securely in the KeyStore.

For **android versions 18 and higher and Pre M**, KeyPairGeneratorSpec API is used. This generates a Public/Private key pair just like RSA and is added to the KeyStore securely. Encrypting a block of text is performed with the Public key of the Key Pair, whereas decryption is performed using the Private Key of the Key Pair by retrieving the keys from the KeyStore.

For **android versions 16 and higher and pre 18**, we are simply encrypting and decrypting the data. This is done by hashing the data with the hash value generated using the SHA-1 hash function. This cipher text is stored in the Preferences. To retrieve the secret information, the cipher text is converted to plain text using the hash value.

Feel free to open pull requests, report issues or suggest enhancements.
May the force be with you!
