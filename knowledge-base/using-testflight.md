# Using TestFlight for User Testing on iOS

Apple's [TestFlight](https://developer.apple.com/testflight/) can be used to perform user testing of Ionic applications and to gather user feedback. When TestFlight is used in combination with [Ionic Pro Deploy](https://ionicframework.com/docs/pro/deploy/) code updates can be delivered to the testers throughout the testing cycle.

Using this user testing strategy involves the following steps:

1. create an initial native build of your application
1. upload your application to the AppStore for use with TestFlight
1. during the 90 day testing period, deploy HTML, CSS, and JavaScript changes to your application via Ionic Pro Deploy

A new build will only need to be uploaded to the AppStore if native layer changes are made or if the 90 day testing period expires.

## Steps

In order to complete these steps, you will need to have a Distribution Provisioning Profile associated with your app's bundle ID. That Provisioning Profile will need to be associated with a valid Production Certificate that you have access to use. Before continuing, please make sure you have all that properly configured and that your Apple Developer user has the required roles in order to perform the required operations.

1. Create an Ionic application and link it with Ionic Pro
   1. Follow the [Getting Started](https://ionicframework.com/docs/pro/basics/getting-started/) steps
   1. Include the [Deploy Plugin](https://ionicframework.com/docs/pro/deploy/setup/#installation)
1. Add the iOS platform to your application (`ionic cordova platform add ios`), be sure to update the `config.xml` file for your application, especially the `widget/id` which will become your app's bundle ID
1. Build out your application, and when ready perform an `ionic build`
1. Open the application in Xcode (ex: `open platforms/ios/MyApp.xcworkspace`)
1. Follow Apple's steps for [Distributing an app using TestFlight](https://help.apple.com/xcode/mac/9.3/#/dev2539d985f). Note that in some cases the Ionic and Cordova build processes have already performed certain actions such as setting the `Bundle ID` based on data in your `config.xml` file. Key points in the steps are:
   1. Signing: If automatic signing cannot be used, manually sign the app using a valid Production Certificate and an associated Distribution Provisioning Profile
   1. Select `Generic iOS Device` in the Scheme toolbar menu on the main window of Xcode
   1. Select Project -> Archive from the Xcode menu, this should open the archive window
   1. [Add the app](https://help.apple.com/itunes-connect/developer/#/dev2cd126805) in iTunes Connect
   1. From the Xcode Archive window, validate the archive
   1. Press the `Upload to AppStore...` button
   1. Once the upload completes, open the app in iTunes connect to add internal and external testers and fill out the information for beta-test review

## Testing the App and Deploying Changes

At this point, internal testers should be able to download your application via TestFlight. External users must wait until Apple approves the app for beta-testing.

Changes to the Ionic application can now be pushed to Ionic Pro and distributed to testers using Ionic Pro's deploy feature by pushing the code to Ionic Pro and deploying the resulting build to the channel that is being used by the test app.

## Further Help

If you would like more in-depth help with the full development cycle of your application including user testing, please contact our Enterprise Customer Success team. Our experts can provide hands-on and highly customized training and advisory services to get you started and keep your project on track. We also offer guaranteed SLAs for peace of mind that your apps will stay up and running.

Learn more by visiting our [Ionic for Enterprise](https://ionicframework.com/enterprise) page.
