<p align="center"><img src="https://user-images.githubusercontent.com/378279/36642269-6195b10c-1a3d-11e8-9e1b-37a3d1bcf7b3.png" align="center" width="150" height="201" alt="" /></p>

<h1 align="center">react-native-keychain</h1>

[![Travis](https://img.shields.io/travis/oblador/react-native-keychain.svg)](https://travis-ci.org/oblador/react-native-keychain) [![npm](https://img.shields.io/npm/v/react-native-keychain.svg)](https://npmjs.com/package/react-native-keychain) [![npm](https://img.shields.io/npm/dm/react-native-keychain.svg)](https://npmjs.com/package/react-native-keychain)

Keychain Access for React Native. Currently functionality is limited to just storing internet and generic passwords.

### New 2.0.0 with improved android implementation

The KeychainModule will now automatically use the appropriate CipherStorage implementation based on API level:

* API level 16-22 will en/de crypt using Facebook Conceal
* API level 23+ will en/de crypt using Android Keystore

Encrypted data is stored in SharedPreferences.

## Installation

1 . `$ npm install --save react-native-keychain`

or

`$ yarn add react-native-keychain`


2 . `$ react-native link react-native-keychain` and check `MainApplication.java` to verify the package was added.

3 .  rebuild your project


* on Android, the `setInternetCredentials(server, username, password)` call will be resolved as call to `setGenericPassword(username, password, server)`. Use the `server` argument to distinguish between multiple entries.

Check out the "releases" tab for breaking changes and RN version compatibility. v1.0.0 is for RN >= 0.40

## ❗ Enable `Keychain Sharing` entitlement for iOS 10

For iOS 10 you'll need to enable the `Keychain Sharing` entitlement in the `Capabilities` section of your build target. (See screenshot). Otherwise you'll experience the error shown below.

![screen shot 2016-09-16 at 20 56 33](https://cloud.githubusercontent.com/assets/512692/18597833/15316342-7c50-11e6-92e7-781651e61563.png)

```
Error: {
    code = "-34018";
    domain = NSOSStatusErrorDomain;
    message = "The operation couldn\U2019t be completed. (OSStatus error -34018.)";
}
```


## Usage

See `KeychainExample` for fully working project example.

Both `setGenericPassword` and `setInternetCredentials` allow to store strings only!

Both `getInternetCredentials` and `getGenericPassword` will resolve to the stored value, or to false, in case there is no record stored. They will reject only if an unexpected error is encountered.

```js
import * as Keychain from 'react-native-keychain';

const username = 'zuck';
const password = 'poniesRgr8';

// Generic Password, service argument optional
Keychain
  .setGenericPassword(username, password)
  .then(function() {
    console.log('Credentials saved successfully!');
  });

// service argument optional
Keychain
  .getGenericPassword()
  .then(function(credentials) {
    console.log('Credentials successfully loaded for user ' + credentials.username);
  }).catch(function(error) {
    console.log('Keychain couldn\'t be accessed!', error);
  });

// service argument optional
Keychain
  .resetGenericPassword()
  .then(function() {
    console.log('Credentials successfully deleted');
  });

// Internet Password, server argument required
const server = 'http://facebook.com';
Keychain
  .setInternetCredentials(server, username, password)
  .then(function() {
    console.log('Credentials saved successfully!');
  });

Keychain
  .getInternetCredentials(server)
  .then(function(credentials) {
    if (credentials) {
      console.log('Credentials successfully loaded for user ' + credentials.username);
    }
  });

Keychain
  .resetInternetCredentials(server)
  .then(function() {
    console.log('Credentials successfully deleted');
  });

Keychain
  .requestSharedWebCredentials()
  .then(function(credentials) {
    if (credentials) {
      console.log('Shared web credentials successfully loaded for user ' + credentials.username);
    } 
  })

Keychain
  .setSharedWebCredentials(server, username, password)
  .then(function() {
    console.log('Shared web credentials saved successfully!');
  })

```

### Options

| Key | Applies to | Description | Default |
|---|---|---|---|
|**`accessControl`**|`PasswordWithAuthentication`|This dictates how a keychain item may be used, see possible values in `Keychain.ACCESS_CONTROL`. |*`Keychain.ACCESS_CONTROL.TOUCH_ID_CURRENT_SET_OR_DEVICE_PASSCODE`*|
|**`accessible`**|`GenericPassword`, `InternetCredentials`|This dictates when a keychain item is accessible, see possible values in `Keychain.ACCESSIBLE`. |*`Keychain.ACCESSIBLE.WHEN_UNLOCKED`*|
|**`accessGroup`**|`GenericPassword`, `InternetCredentials`, `PasswordWithAuthentication`|In which App Group to share the keychain. Requires additional setup with entitlements. |*None*|
|**`authenticationPrompt`**|`PasswordWithAuthentication`|What to prompt the user when unlocking the keychain with biometry or device password. |`Authenticate to retrieve secret!`|
|**`authenticationType`**|`canImplyAuthentication`|Policies specifying which forms of authentication are acceptable. |`Keychain.AUTHENTICATION_TYPE.DEVICE_PASSCODE_OR_BIOMETRICS`|
|**`service`**|`GenericPassword`, `PasswordWithAuthentication`|Reverse domain name qualifier for the service associated with password. |*App bundle ID*|

#### `Keychain.ACCESS_CONTROL` enum

| Key | Description |
|-----|-------------|
|**`USER_PRESENCE`**|Constraint to access an item with either Touch ID or passcode.|
|**`BIOMETRY_ANY`**|Constraint to access an item with Touch ID for any enrolled fingers.|
|**`BIOMETRY_CURRENT_SET`**|Constraint to access an item with Touch ID for currently enrolled fingers.|
|**`DEVICE_PASSCODE`**|Constraint to access an item with a passcode.|
|**`APPLICATION_PASSWORD`**|Constraint to use an application-provided password for data encryption key generation.|
|**`BIOMETRY_ANY_OR_DEVICE_PASSCODE`**|Constraint to access an item with Touch ID for any enrolled fingers or passcode.|
|**`BIOMETRY_CURRENT_SET_OR_DEVICE_PASSCODE`**|Constraint to access an item with Touch ID for currently enrolled fingers or passcode.|

#### `Keychain.ACCESSIBLE` enum

| Key | Description |
|-----|-------------|
|**`WHEN_UNLOCKED`**|The data in the keychain item can be accessed only while the device is unlocked by the user.|
|**`AFTER_FIRST_UNLOCK`**|The data in the keychain item cannot be accessed after a restart until the device has been unlocked once by the user.|
|**`ALWAYS`**|The data in the keychain item can always be accessed regardless of whether the device is locked.|
|**`WHEN_PASSCODE_SET_THIS_DEVICE_ONLY`**|The data in the keychain can only be accessed when the device is unlocked. Only available if a passcode is set on the device. Items with this attribute never migrate to a new device.|
|**`WHEN_UNLOCKED_THIS_DEVICE_ONLY`**|The data in the keychain item can be accessed only while the device is unlocked by the user. Items with this attribute do not migrate to a new device.|
|**`AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY`**|The data in the keychain item cannot be accessed after a restart until the device has been unlocked once by the user. Items with this attribute never migrate to a new device.|
|**`ALWAYS_THIS_DEVICE_ONLY`**|The data in the keychain item can always be accessed regardless of whether the device is locked. Items with this attribute never migrate to a new device.|

#### `Keychain.AUTHENTICATION_TYPE` enum

| Key | Description |
|-----|-------------|
|**`DEVICE_PASSCODE_OR_BIOMETRICS`**|Device owner is going to be authenticated by biometry or device passcode.|
|**`BIOMETRICS`**|Device owner is going to be authenticated using a biometric method (Touch ID or Face ID).|

### Note on security

On API levels that do not support Android keystore, Facebook Conceal is used to en/decrypt stored data. The encrypted data is then stored in SharedPreferences. Since Conceal itself stores its encryption key in SharedPreferences, it follows that if the device is rooted (or if an attacker can somehow access the filesystem), the key can be obtained and the stored data can be decrypted. Therefore, on such a device, the conceal encryption is only an obscurity. On API level 23+ the key is stored in the Android Keystore, which makes the key non-exportable and therefore makes the entire process more secure. Follow best practices and do not store user credentials on a device. Instead use tokens or other forms of authentication and re-ask for user credentials before performing sensitive operations.

## Manual Installation

### iOS

#### Option: Manually

* Right click on Libraries, select **Add files to "…"** and select `node_modules/react-native-keychain/RNKeychain.xcodeproj`
* Select your project and under **Build Phases** -> **Link Binary With Libraries**, press the + and select `libRNKeychain.a`.

#### Option: With [CocoaPods](https://cocoapods.org/)

Add the following to your `Podfile` and run `pod update`:

```
pod 'RNKeychain', :path => '../node_modules/react-native-keychain'
```

### Android

#### Option: Manually

* Edit `android/settings.gradle` to look like this (without the +):

  ```diff
  rootProject.name = 'MyApp'

  include ':app'

  + include ':react-native-keychain'
  + project(':react-native-keychain').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-keychain/android')
  ```

* Edit `android/app/build.gradle` (note: **app** folder) to look like this: 

  ```diff
  apply plugin: 'com.android.application'

  android {
    ...
  }

  dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.0.1'
    compile 'com.facebook.react:react-native:0.19.+'
  + compile project(':react-native-keychain')
  }
  ```

* Edit your `MainApplication.java` (deep in `android/app/src/main/java/...`) to look like this (note **two** places to edit):

  ```diff
  package com.myapp;

  + import com.oblador.keychain.KeychainPackage;

  ....

  public class MainActivity extends extends ReactActivity {

    @Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
  +             new KeychainPackage()
        );
    }
    ...
  }
  ```
  
#### Proguard Rules

On Android builds that use proguard (like release), you may see the following error:

```
RNKeychainManager: no keychain entry found for service:
JNI DETECTED ERROR IN APPLICATION: JNI FindClass called with pending exception java.lang.NoSuchFieldError: no "J" field "mCtxPtr" in class "Lcom/facebook/crypto/cipher/NativeGCMCipher;" or its superclasses
```

If so, add a proguard rule in `proguard-rules.pro`:

```
-keep class com.facebook.crypto.** {
   *;
}
```

## Maintainers

<table>
  <tbody>
    <tr>
      <td align="center">
        <a href="https://github.com/oblador">
          <img width="150" height="150" src="https://github.com/oblador.png?v=3&s=150">
          <br>
          <strong>Joel Arvidsson</strong>
        </a>
        <br>
        Author
      </td>
      <td align="center">
        <a href="https://github.com/vonovak">
          <img width="150" height="150" src="https://github.com/vonovak.png?v=3&s=150">
          </br>
          <strong>Vojtech Novak</strong>
        </a>
        <br>
        Maintainer
      </td>
      <td align="center">
        <a href="https://github.com/pcoltau">
          <img width="150" height="150" src="https://github.com/pcoltau.png?v=3&s=150">
          </br>
          <strong>Pelle Stenild Coltau</strong>
        </a>
        <br>
        Maintainer
      </td>
    </tr>
  <tbody>
</table>


## License
MIT © Joel Arvidsson 2016-2018
