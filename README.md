<img src="https://github.com/Veriff/veriff-sample-ios/blob/master/veriff-logo.png">



# Mobile SDK Documentation iOS


### Latest iOS SDK releases
[Releases](https://github.com/Veriff/veriff-sample-ios/blob/master/RELEASES.md)

# Step-By-Step integration guide

If upgrading from version 1.* skip to [chapter 3](#migration)

## Requirements

Integration Veriff iOS SDK requires at least iOS version 10.0

## 1. Add framework to a project

### 1.1 Via Cocoapods

Steps to set up dependency manager. Only follow these steps if Cocoapods is not installed and iOS project is not using it.

To install Cocoapods run:

```sh
sudo gem install cocapods
```

Run ```pod init``` in your project folder. This creates Podfile with application target. Under that target add following line.

```ruby
pod 'VeriffSDK', '~> 2.2.0'
```

After this is done run ``` pod install ``` in folder conntaining Podfile. This will  download and install VeriffSDK in your Xcode workspace.

### 1.2 Manual method

Copy the “Veriff.framework” to your project and add it as embedded binary to application target.



## 2. Using the Veriff library

The verification process must be presented by application. The app is aware of the current customer, who is either logged in via the Vendor system or who is identified other way with unique system pass. The Vendor must be able to determine the customer later, if verification ends/ or returns. It depends entirely on the Vendor business logic.

Every Veriff session is unique for a client. The session expires after 7 days automatically, but is valid until then. After the verification data is uploaded, the SDK v2.0 does not wait for the final verification result (async). The SDK v2.0 only notifies whether the verification data upload is successful or not.

The verification result is sent to the Vendor server in the background. ( See https://developers.veriff.me/#webhooks_decision_post ). Veriff SDK sends callbacks to vendor mobile application via `VeriffDelegate`.


### 2.1 Add camera usage description to application Info.plist

Veriff SDK is using camera for capturing photos during identification flow. Application is responsible to describe the reason why camera is used. If Info.plist of application doesn't yet contain `NSCameraUsageDescription` it needs to be added. Value for this key is type of `String` containing reason why app is using camera. Not adding this entry causes system to kill application when it requests permission for camera.

### 2.2 Import Veriff in your code:

```swift
import Veriff
```


### 2.3 Configure SDK before displaying it

**Setting configuration parameters**

| Parameters   | Explanation                                                  |
| ------------ | ------------------------------------------------------------ |
| sessionUrl   | Determines the environment for testing: 'https://staging.veriff.me/v1/' and for production: 'https://magic.veriff.me/v1/' |
| sessionToken | The session Token unique for the client. It needs to be generated following https://developers.veriff.me/#sessions_post. Note that token request also sends URL but that is not correct one to use. Prefer ones listed above. |

```swift
let conf = VeriffConfiguration(sessionToken: token, sessionUrl: baseUrl)
let veriff = Veriff.shared
veriff.set(configuration: conf)
```

**UI styling**

Some of UI in SDK can be modified to match general appearance of app. This can change with upcoming releases and give more options for styling.

| Property | Explanation |
| ------------- | ------------- |
| controlsColor | Allows to set custom color for buttons |

```swift
let schema = ColorSchema()
schema.controlsColor = .red
veriff.set(colorSchema: schema)
```


### 2.4 Handling result codes form SDK

Veriff SDK returns a number of result codes, that application can handle. Implement `VeriffDelegate` and assign it to delegate property of `Veriff` instance.

Example:
```swift
extension VerificationService: VeriffDelegate {
    func onSession(result: VeriffResult, sessionToken: String) {
    	 switch result.code {
             case .STATUS_SUBMITTED:
         }
    }
}
```

Here is list of all possible status codes

| Status code | Explanation                                             |
| ------------ | ------------------------------------------------------ |
| UNABLE_TO_ACCESS_CAMERA | User denied access to the camera |
| STATUS_USER_CANCELED | User canceled the verification process |
| STATUS_SUBMITTED | User submitted the photos or finished video call |
| STATUS_ERROR_SESSION | The session token is either corrupt, or has expired. A new sessionToken needs to be generated in this case  |
| STATUS_ERROR_NETWORK | SDK could not connect to backend servers. |
| STATUS_ERROR_NO_IDENTIFICATION_METHODS_AVAILABLE | Given session cannot be started as there are no identification methods |
| STATUS_DONE | The session status is finished from clients perspective |
| STATUS_ERROR_UNKNOWN | An unkown error occured |


### 2.5 Start verification process 

Veriff SDK looks for currently presented `UIViewController` from key `UIWindow`s' root view controller. Using this view controller verification UI is presented.

```swift
veriff.startAuthentication()

// Alternatively pass ViewController that is used for presenting verification UI. 
// That ViewController has to be in view hierarchy and not presenting.
startAuthentication(from viewController: UIViewController)
```


<a name="migration"></a>
## 3. Migrating from 1.* to 2.*

Veriff library 2.0 integration has changed significantly since 1.*. We dropped TwiloVideo and requirement to pass Veriff specific push notifications to SDK. It simplifies setup and is reflected in API. Since SDK 2.0 it's no longer needed to embed Twiliovideo framework in app as SDK doesn't link against it anymore. Also Firebase project created for Veriff SDK is not needed in new version. If there were push notification translations for Veriff messages this can be also removed from Xcode project.

### 3.1 API changes

**Getting SDK instance**

v1.*

```swift
Veriff.sharedInstance() 
```
v2.*

```swift
Veriff.shared
```



**Pass session URL and token to SDK**

v1.*

```swift
configure(_ block: @escaping VeriffConfigurationBlock)
```

v2.*

```swift
set(configuration: VeriffConfiguration)
```



**Customize background for selected views in SDK**

v1.x 
```swift
setBackgroundImage(_ imageUrlString: String)
```
v2.*

Removes background image customization



**Customizing UI of SDK**

v1.x 

```swift
createColorSchema(_ block: @escaping VeriffColorSchemaBlock)
```

v2.*

```swift
set(colorSchema: ColorSchema)
```



**Changes for ColorSchema** 

v2.0

Removes backgroundColor, footerColor, hintFooterColor, cameraControlsColor properties



**Listening callbacks about verification process**

v1.x 

```swift
setResultBlock(_ result: @escaping ProcessBlock)
```

v2.*

Implement `VeriffDelegate` to handle result

```swift
onSession(result: VeriffResult, sessionToken: String)
```

Set implementation as a delegate of Veriff shared instance

```swift
veriff.delegate = service
```



**Displaying verification flow**

v1.x

```swift
requestViewController(completion: @escaping AuthCompletionBlock)
```

v2.*

New API despite removing completion closure is still async.

```swift
startAuthentication()
```

If it's needed to use different ViewController that SDK uses for presenting its own UI use following API

```swift
startAuthentication(from viewController: UIViewController)
```


