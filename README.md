# GodotAppleSignIn iOS Plugin

## Usage

Since this is only available for iOS, you cannot use the class name directly. Instead, use `ClassDB` to create an instance:

```gdscript
var signin_handler = null

func _ready() -> void:
	if ClassDB.class_exists("GodotAppleSignIn"):
		signin_handler = ClassDB.instantiate("GodotAppleSignIn")
```

Then you can call methods and connect signals using the custom variable.

## API

### Methods

**`initiate_signin()`**

Initiate the Sign in with Apple procedure. This will popup a window for the user to perform login with their Apple account. Note that they can customize their name at this point and choose to use a random email instead of their main one. You can catch the result of this call using the `signin_completed` signal.

**`request_credential_state(id: String)`**

Start a request for the current credential state, from the user id sent as an argument. The result will come asynchronously with the `state_received` signal.

This is useful to check if the user you have logged is still signed in, like when the app just started. If the user is not logged then you can perform a new sign in request.

### Signals

**`signin_completed(data: Dictionary, error: String)`**

Emitted when the sign in process is completed, whether or not there's any error. If it is successful, the `error` string will be empty and the `data` dictionary will have the following information about the user:

- `id`: The user id as given by Apple.
- `name`: The user's full name as provided.
- `email`: The user's selected email.
- `identity_token`: Raw JWT provided by Apple.
- `authorization_code`: Authorization code given by Apple.
- `jwt`: Decoded JWT data as a `Dictionary[String, String]`.

Note that name and email are provided by the API only when the user does the initial sign up. After that, any sign in will have these fields as empty or `null`. You can get the email from the decode JWT data though.

If there's an error, `data` will be empty and the `error` parameter will contain one of the values of the [`ASAuthorizationError.Code`](https://developer.apple.com/documentation/authenticationservices/asauthorizationerror-swift.struct/code) enum as a string.

**`state_received(state: String)`**

Emitted when the requested credential state is received. The string will contain one of the values from [`ASAuthorizationAppleIDProvider.CredentialState`](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidprovider/credentialstate).

## Installation

Get the ZIP package from Releases and extract on your project in the `ios/plugins` folder (you'll have to create it first). It will create an `apple_signin` folder there with frameworks and the needed `.gdip` file so Godot can recognize the plugin.

On the Export dialog, enable `Apple Sign In` on the `Plugins` section.

You also need to enable the `Sign in with Apple` capability on your project. You can do that on the generated XCode project after export, or you can add this to the `Additional` field in the `Entitlements` section on the iOS export options:

```
<key>com.apple.developer.applesignin</key>
<array><string>Default</string></array>
```

> [!IMPORTANT]
> You need a paid developer account to use the Sign in with Apple feature, as listed in the [Supported capabilities (iOS) page](https://developer.apple.com/help/account/reference/supported-capabilities-ios). This is the case even if are testing locally.

## Building

This plugin uses the [SCons build system](https://scons.org), so you need it installed first. This is only tested on a macOS computer.

Then you can call scons from the terminal:

```
$ scons arch=arm64 target=release_debug
```

Run `scons -h` for all the options.

### Installation after building

If you build yourself, you'll have to get the needed files from the artifacts on your local computer.

The easiest way to generate all files is to run the script `scripts/release_xcframework.sh` locally to build all necessary libraries and create `bin/release` folder. From there you can copy or move the `apple_signin` folder into the `ios/plugins` folder in your project.
