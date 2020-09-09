
# Tutorial: Converting a Cordova App to Capacitor

Converting a Cordova application to a Capacitor application is a multi-step process.  But each step is relatively small and some of the work is scripted, which means you just need to run a few commands to be off and running.

This tutorial walks through the conversion process using a simple Cordova application.

Before performing this process on any of your own applications, we suggest that you:

- **Complete this tutorial with the sample application** - this will allow you to fully understand the process and the various steps taken in a controlled environment.
- **Clean up your Cordova dependencies** - Capacitor is great, but it is not a magic bullet. If your application currently has a lot of plugins installed, that is going to hamper your progress. Simplification is the key here. We suggest that you do the following in your Cordova application before performing the conversion:
   - remove any unused plugins
   - review your use of plugins to ensure you need the ones you are using
   - review the versions of your plugins to ensure you are up to date on your versions

## Overview

There are two main phases of any conversion from Cordova to Capacitor: Basic Configuration and Project Cleanup.

### Phase 1: Basic Configuration

The goal of this phase is to get Capacitor installed in your project and the native projects created and building. This phase generally takes 15 minutes or less to complete, and often results in a fully functional application. Here are the full steps to this phase:

```bash
# Step 1
# Always work in a dedicated branch, NEVER work directly in the main branch of your application
$ git checkout -b feature/convertToCapacitor

# Step 2
$ ionic integration enable capacitor

# Step 3
edit capacitor.config.json

# Step 4
# Note: if you have never built your project, do "npm run build" first
$ ionic cap add ios
$ ionic cap add android

# Step 5
$ cordova-res ios --skip-config --copy
$ cordova-res android --skip-config --copy

# Step 6
$ ionic cap open ios
$ ionic cap open android
```

At the end of this process, many apps will already be working fine. Other apps may require minor tweaks due to issues with plugin configuration, but those will be addressed in the "Cleanup" section.

### Phase 2: Project Cleanup

Phase 2 has three main goals:

- Configure remaining Cordova plugins where needed
- Replace Cordova plugins with Capacitor Plugins or APIs where possible
- Remove the Cordova configuration and any plugins that are no longer being used

This step takes about 30-6o minutes for simple projects but can take significantly longer depending on the complexity of native interactions within the application being converted. As is always the rule, the fewer the plugins that your application uses, the better. For most well balanced applications, this step will most likely take an hour or less.

We will dive into this step in more detail in its own section of this walkthrough.

**Important:** Your project may run just fine after the first phase, but you should still have a look at this phase. There are important steps here that will allow you to take full advantage of using Capacitor as your native layer.

Use the navigation items in the header to see each of these phases in detail.

## Create a Cordova App

Let's create a very simple Cordova application to use for this tutorial. This application will just be one of Ionic's starter applications with the minimal Cordova plugins included.

```bash
$ ionic start cor-to-cap blank --type=angular --cordova
$ cd cor-to-cap
```

To make this app unique, let's give it a unique bundle ID and name by editing the `config.xml` file. Here is what I am using:

```xml
<widget id="com.kensodemann.cortocap" version="0.0.1" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/    ns/1.0">
    <name>Cor to Cap</name>
    <description>An awesome Ionic/Cordova app.</description>
```

Now let's generate the platforms:

```bash
$ npm run build
$ ionic cordova platform add ios
$ ionic cordova platform add android
```

We now have a fully functional, though minimal, Cordova application. In the next section, we will perform phase one of our conversion: installing and configuring Capacitor.

## Phase 1: Install and Cofigure Capacitor

Installing and configuring Capacitor is a multi-step process, but each step is very small. The full set of steps looks like this:

```bash
# Step 1
# Always work in a dedicated branch, NEVER work directly in the main branch of your application
$ git checkout -b feature/convertToCapacitor

# Step 2
$ ionic integration enable capacitor

# Step 3
edit capacitor.config.json

# Step 4
# Note: if you have never built your project, do "npm run build" first
$ ionic cap add ios
$ ionic cap add android

# Step 5
$ cordova-res ios --skip-config --copy
$ cordova-res android --skip-config --copy

# Step 6
$ ionic cap open ios
$ ionic cap open android
```

At the end of this process, many apps will already be working. Other apps may require minor tweaks due to issues with plugin configuration, but those will be addressed in the "Cleanup" phase.

Let's have a look at each of the above steps in detail.

### Create a Working Branch

You should never ever do any work in the main branch of your application. You should always use a "branch and merge" strategy for every change you make, even if you are the only developer on your application. This way, should you decide that whatever changes you are making are not a path you want to go down, you just need to abandon the branch rather than undo a bunch of commits. This advice holds especially true here where you are replacing a whole section of your stack. Even though this process is very easy, you should still approach it with care.

```bash
git checkout -b feature/convertToCapacitor
```

Also remember to commit early and commit often. Always make small commits as you go, and then squash them into a single commit before merging your branch into the main branch.

### Integrate Capacitor

```bash
$ ionic integration enable capacitor
```

This command installs `@capacitor/core` and `@capacitor/cli` in your project and creates a template basic `capacitor.config.json` file.

### Edit `capacitor.config.json`

Have a look at the newly created `capacitor.config.json` file:

```json
{
  "appId": "io.ionic.starter",
  "appName": "cor-to-cap",
  "bundledWebRuntime": false,
  "npmClient": "npm",
  "webDir": "www",
  "plugins": {
    "SplashScreen": {
      "launchShowDuration": 0
    }
  },
  "cordova": {
    "preferences": {
      "ScrollEnabled": "false",
      "BackupWebStorage": "none",
      "SplashMaintainAspectRatio": "true",
      "FadeSplashScreenDuration": "300",
      "SplashShowOnlyFirstTime": "false",
      "SplashScreen": "screen",
      "SplashScreenDelay": "3000"
    }
  }
}
```

Change the `appId` and the `appName` to match what you currently have in the `config.xml` file. The rest of the file can remain as-is:

```json
{
  "appId": "com.kensodemann.cortocap",
  "appName": "Cor to Cap",
  "bundledWebRuntime": false,
  ...
```

### Add the iOS and Android Platforms

Let's add the iOS and Android platforms:

```bash
ionic cap add ios
ionic cap add android
```

The output of these two commands is similar. Notice that the Compatible Cordova plugins that we have in our project are automatically installed while the incompatible plugins are ignored.

```bash
> capacitor add ios
✔ Installing iOS dependencies in 17.84s
✔ Adding native xcode project in: /Users/kensodemann/Projects/Training/cor-to-cap/ios in 27.74ms
✔ add in 17.87s
✔ Copying web assets from www to ios/App/public in 866.04ms
✔ Copying native bridge in 6.84ms
✔ Copying capacitor.config.json in 1.25ms
⠇ copy  Found 1 Cordova plugin for ios
    cordova-plugin-device (2.0.2)
✔ copy in 1.01s
✔ Updating iOS plugins in 13.99ms
  Found 0 Capacitor plugins for ios:
  Found 1 Cordova plugin for ios
    cordova-plugin-device (2.0.2)
✔ Updating iOS native dependencies with "pod install" (may take several minutes) in 10.58s
  Found 5 incompatible Cordova plugins for ios, skipped install
    cordova-plugin-ionic-keyboard (2.2.0)
    cordova-plugin-ionic-webview (4.2.1)
    cordova-plugin-splashscreen (5.0.2)
    cordova-plugin-statusbar (2.4.2)
    cordova-plugin-whitelist (1.3.3)
✔ update ios in 10.62s
```

Unlike Cordova, where the platforms are build artifacts, and thus cannot be directly touched, these platforms are source artifacts and are fully under your control. That means no more trying to manipulate the platforms via weird directives in the `config.xml` file or via hard to maintain hooks. Yes!! 🎉

### Add the Icons and Slash Screen

The final step is to initialize the newly created projects with the icon and splash screen from our original project. Since our original project already has `icon.png` and `splash.png` template files in the `resources` directory, all we have to do is use the `cordova-res` script to copy them to our newly generated platforms.

```bash
$ cordova-res ios --skip-config --copy
$ cordova-res android --skip-config --copy
```

If `cordova-res` produces any warnings with this step, you can ignore them.

### Run the App

At this point, you are able to view  your Capacitor application by loading it in either Xcode or Android Studio and running it on a devie or emulator. Use `ionic cap open` to open the appropriate IDE.

```bash
$ ionic cap open ios
$ ionic cap open android
```

At this point, our basic app is fully functional as a Capacitor application. Some applications may need a little more care based on the plugins that are used and the amount of configuration that they require. In the next phase, we will clean up the application and tie up any loose ends.


## Phase 2: Project Cleanup

Cleanup involves three main steps:

- Replace Cordova plugins with Capacitor Plugins or APIs where possible
- Configure remaining Cordova plugins where needed
- Remove the Cordova configuration and any plugins that are no longer being used

Let's jump right in.

### Replace Plugins

The first thing we should do in our cleanup efforts is to look for opportunities to use <a href="https://github.com/capacitor-community/" target="_blank">Capacitor Plugins</a> and <a href="https://capacitorjs.com/docs/apis" target="_blank">Capacitor's built in plugin APIs</a> instead of using Cordova plugins.

In the case of our application, the only Cordova plugins that are actively being used are the status bar and splash screen plugins. Both of these should be replaced with Capacitor Plugin API calls. This change involves the following modifications:

- Modify the `AppComponent` test to expect Capacitor API calls
- Modify the `AppComponent` itself to call the Capacitor APIs
- Modify the `AppComponentModule` to no longer provide the `@ionic-native` wrappers for the Cordova plugins

#### Modify `src/app/app.component.spec.ts`

Import the `Plugins` object from `@capacitor/core`. In our case, we will also be styling the status bar, so we will need to import the `StatusBarStyle` enumeration.

```typescript
import { Plugins, StatusBarStyle } from '@capacitor/core';
```

Replace the current `beforeEach()` portion of the test with the following code:

```typescript
let originalSplashScreen: any;
let originalStatusBar: any;

beforeEach(async(() => {
  originalStatusBar = Plugins.StatusBar;
  originalSplashScreen = Plugins.SplashScreen;
  Plugins.StatusBar = jasmine.createSpyObj('StatusBar', ['setStyle']);
  Plugins.SplashScreen = jasmine.createSpyObj('SplashScreen', ['hide']);
  TestBed.configureTestingModule({
    declarations: [AppComponent],
    schemas: [CUSTOM_ELEMENTS_SCHEMA],
    providers: [
      {
        provide: Platform,
        useFactory: () => jasmine.createSpyObj('Platform', { is: false })
      }
    ]
  }).compileComponents();
}));

afterEach(() => {
  Plugins.StatusBar = originalStatusBar;
  Plugins.SplashScreen = originalSplashScreen;
});
```

Replace the `should initialize the app` test with the following set of tests:

```typescript
describe('initialization', () => {
  let platform: Platform;
  beforeEach(() => {
    platform = TestBed.inject(Platform);
  });

  describe('in a hybrid mobile context', () => {
    beforeEach(() => {
      (platform.is as any).withArgs('hybrid').and.returnValue(true);
    });

    it('styles the status bar', () => {
      TestBed.createComponent(AppComponent);
      expect(Plugins.StatusBar.setStyle).toHaveBeenCalledTimes(1);
      expect(Plugins.StatusBar.setStyle).toHaveBeenCalledWith({ style: StatusBarStyle.Light });
    });

    it('hides the splash screen', () => {
      TestBed.createComponent(AppComponent);
      expect(Plugins.SplashScreen.hide).toHaveBeenCalledTimes(1);
    });
  });

  describe('in a web context', () => {
    beforeEach(() => {
      (platform.is as any).withArgs('hybrid').and.returnValue(false);
    });

    it('does not style the status bar', () => {
      TestBed.createComponent(AppComponent);
      expect(Plugins.StatusBar.setStyle).not.toHaveBeenCalled();
    });

    it('does not hide the splash screen', () => {
      TestBed.createComponent(AppComponent);
      expect(Plugins.SplashScreen.hide).not.toHaveBeenCalled();
    });
  });
});
```

You should now clean up any unsed imports or variable declarations you may have. Your `AppComponent` tests will be failing at this point, but we will fix that next.

#### Modify `src/app/app.component.ts`

You need to modify the `AppComponent` as such:

- You no longer need to import the `@ionic-native` services
- You need to call the Capacitor Plugin APIs instead of the `@ionic-native` services

Your final code should look like this:

```typescript
import { Component } from '@angular/core';

import { Platform } from '@ionic/angular';
import { Plugins, StatusBarStyle } from '@capacitor/core';

@Component({
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.scss']
})
export class AppComponent {
  constructor(private platform: Platform) {
    this.initializeApp();
  }

  initializeApp() {
    if (this.platform.is('hybrid')) {
      const { SplashScreen, StatusBar } = Plugins;
      SplashScreen.hide();
      StatusBar.setStyle({ style: StatusBarStyle.Light });
    }
  }
}
```

#### Modify `src/app/app.module.ts`

Since nothing is referencing the `@ionic-native` services any more, they no longer need to be provided. Remove all references to them in this file as well.

- Remove the ES6 imports of them
- Remove the code that adds them to the `providers` array

### Configure Plugins

This step may or may not be required for your project. For the base Cordova project we are converting here it is not required at all. However, if it is required, common tasks include:

- Handling Cordova plugin preferences
- Handling additional `config.xml` items
- Handling Cordova hooks
- Handling plugin variables

#### Handling Cordova Plugin Preferences

If any of your Cordova plugins require preferences to be set the `config.xml` file, those preferences should be set in the `cordova` section of the `capacitor.config.json` file. These should have been automatically copied over for you during Phase 1, but you may want to double check them. Here is what I have in my project:

```JSON
  "cordova": {
    "preferences": {
      "ScrollEnabled": "false",
      "BackupWebStorage": "none",
      "SplashMaintainAspectRatio": "true",
      "FadeSplashScreenDuration": "300",
      "SplashShowOnlyFirstTime": "false",
      "SplashScreen": "screen",
      "SplashScreenDelay": "3000"
    }
  }
```

**Note:** in our case, we will later remove all of these as we have replaced the use of all Cordova plugins in our application and thus will end up removing all of them. This may or may not be the case in any of your projects.

#### Handling Additional `config.xml` Items

You may have other directives in your `config.xml` file such an `edit-config` node. For example, the following `edit-config` node modifies the iOS project's `info.plist` file:

```xml
<edit-config file="*-Info.plist" mode="merge" target="NSCameraUsageDescription">
  <string>Used to take photos</string>
</edit-config>
```

In these cases, the you need to update the appropriate native project yourself. So the proper question to ask yourself is "How do I configure X in iOS (or Android)?" and then go modify the proper configuration file. You can usually figure this out via the nodes in the `config.xml` file themselves.

#### Handling Cordova Hooks

Some plugins install Cordova hooks. These can be a little trickier to deal with. If you have a plugin that does this, you need to figure out what the hooks are doing and replicate that set up yourself. The good news is, since the native projects are now source artifacts you should only need to do this once.

#### Handling Cordova Plugin Variables

Some plugins require installation time variable. If you have a plugin like this, you will need to figure out what the plugin is using that variable for and then perform the same operation in your native projects. Similar to the situation with Cordova Hooks, you should only need to do this once, however.

### Remove Cordova

In this step, you will:

- Remove unused plugins
- Modify the `package.json` file
- Remove unused files from the filesystem

#### Remove Unused Plugins

Right now, the following plugins are being installed:

- `cordova-plugin-whitelist` - not needed by anything
- `cordova-plugin-statusbar` - using Capacitor API instead
- `cordova-plugin-device` - not used
- `cordova-plugin-splashscreen` - using Capacitor API instead
- `cordova-plugin-ionic-webview` - not needed with Capacitor
- `cordova-plugin-ionic-keyboard` - not needed with Capacitor

As you can see from the comments here, we don't actually need to install any of them. You can uninstall them all:

```bash
$ npm uninstall cordova-plugin-whitelist cordova-plugin-statusbar cordova-plugin-device cordova-plugin-splashscreen cordova-plugin-ionic-webview cordova-plugin-ionic-keyboard
```

We are also no longer using any of the `@ionic-native` stuff (in your own project, you may or may not be still using some `@ionic-native` packages, so only remove `@ionic-native/core` here if all other `@ionic-native` packages are beging removed):

```bash
$ npm uninstall @ionic-native/core @ionic-native/status-bar @ionic-native/splash-screen
```

#### Modify the `package.json` File

Open the `package.json` file and look for the `cordova` section. Remove the whole section. It will look something like the following:

```JSON
  "cordova": {
    "plugins": {
      "cordova-plugin-whitelist": {},
      "cordova-plugin-statusbar": {},
      "cordova-plugin-device": {},
      "cordova-plugin-splashscreen": {},
      "cordova-plugin-ionic-webview": {
        "ANDROID_SUPPORT_ANNOTATIONS_VERSION": "27.+"
      },
      "cordova-plugin-ionic-keyboard": {}
    },
    "platforms": [
      "ios",
      "android"
    ]
  }
```

While you are in there, find the build script. I like to add a `cap copy` command to it to ensure that my Capacitor projects are updated with each web build:

```JSON
    "build": "ng build && cap copy",
```

#### Modify the `capacitor.config.json` File

Remember all of those preferences that were copied over when we initially added the Capacitor integration? They look like this in the `capacitor.config.json` file:

```JSON
  "cordova": {
    "preferences": {
      "ScrollEnabled": "false",
      "BackupWebStorage": "none",
      "SplashMaintainAspectRatio": "true",
      "FadeSplashScreenDuration": "300",
      "SplashShowOnlyFirstTime": "false",
      "SplashScreen": "screen",
      "SplashScreenDelay": "3000"
    }
  }
```

Since we were able to remove all of the Cordova plugins in this project, we don't need any of those any more. So let's just remove the preferences but leave the actual object in the JSON in case we need them for a new plugin we add later.

```JSON
  "cordova": {
    "preferences": {
    }
  }
```

**Note:** In your own project, you may still need some preferences based on which Cordova plugins you still have left.


#### Remove Unused Files

At this point, you no longer need any of the Cordova related files or folders in your project. You can remove all of the following:

- The `plugins` folder
- The `platforms` folder
- The `config.xml` file

```bash
$ rm -rf platforms plugins config.xml
```

## Conclusion

Your application has been fully converted to Capacitor at this point. You may or may not still be including some Cordova plugins, but Capacitor is the only system being used to build your iOS and Android applications for this project.
