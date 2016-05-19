name: inverse
layout: true
class: center, middle, inverse
---
# Hot Updates using CodePush and Ionic 2

???

Hello, thank you for taking the time to join us today. Over the next hour we will be covering how to use Microsoft's CodePush services with a Cordova app to deploy updates directly to users devices. Before getting started, I'll go over some of the Prerequisites for being able to follow along. Hopefully you should have done this prior to todays session.


---
layout: false


# Agenda

* Review Prerequisites
* Ionic 2
* CodePush
* CodePushify Your App
* Review of Demo application
* Deploy to Staging and Sync
* Deployment to Production
* Synching Multiple Versions
* Q & A


---

# Prerequisites

* node
* git
* CodePush cli tools
* Ionic2 CLI tools
* XCode

???

For this session, you will need to have node, git and xcode installed and setup. In addition to this, you will need to install the CodePush CLI tools, and the Ionic 2 CLI tools.

---

background-image: url('ionic2.jpg')


???

TODO: Expand on Ionic 2, and some of the features

---
layout: false

## Ionic Setup

* Install Ionic 2

```shell
npm install -g ionic@beta
```

--

* Install Cordova

```shell
npm install -g cordova
```

--

* Start a new Ionic Project

```shell
ionic start pgdays-workship tutorial --v2 --ts
cd pgdays-workshop
ionic serve
```

???

This will set you up with a basic Ionic project, by default it will use the 'tabs' starter project available for ionic. By passing in the --v2 flag, we are telling it to use version 2 of Ionic. This demo will be using typescript, so that is why we are using the --ts flag.

Before we move onto CodePush, lets take a second to get the project up and running, and type ionic serve to view the demo app in the browser.

Once everyone has this step working, we will move onto hooking CodePush into our application.

---
background-image: url('fbshare.png')
template: inverse

# Code Push

???

TODO: Expand on what CodePush is, and some of the features

---
template: false

## CodePush Setup

* Install CLI tools

```shell
npm install -g code-push-cli
```

--

* Create CodePush account

```shell
code-push register
```

--

* Register your app

```shell
code-push app add <appName>
```

???

Setting up CodePush is fairly straight forward, all you need to do is install the cli tools, create an account and register your application. CodePush will let you create an account with your GitHub credentials. Once everyone has done this, we will move on to setting up Ionic.

---

## Configuring Cordova for CodePush

* Install CodePush plugin

```shell
cordova plugin add cordova-plugin-code-push@latest
```

--

* Add Deployment Key to config.xml

__*config.xml*__
```xml
<platform name="ios">
    <preference name="CodePushDeploymentKey" value="YOUR-IOS-DEPLOYMENT-KEY" />
</platform>
```

???

gHde0lsEJekgm946zOruuErzJUQr4JJvnXkxW

--

* Forget your app or key? no problem

```shell
code-push app ls # list your registered apps
code-push deployment ls AppName -k
```

--

---

## Configuring Cordova for CodePush

* Let Cordova talk with CodePush

__*config.xml*__
```xml
<access origin="*" />
<!-- or -->
<access origin="https://codepush.azurewebsites.net" />
<access origin="https://codepush.blob.core.windows.net" />
```

--

* Let CodePush work with CSP

__*www/index.html*__
```html
<meta http-equiv="Content-Security-Policy"
  content="default-src https://codepush.azurewebsites.net 'self'
  data: gap: https://ssl.gstatic.com
  'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *" />
```

???

At this point, we have Ionic 2 and CodePush installed, now we need to do some extra configuration to get CodePush and Cordova talking with each other.

---
template: inverse

# Settings Tab

---
template: false

### Rename Tab

__*app/pages/tabs/tabs.html*__
```html
<ion-tabs>
  <ion-tab [root]="tab1Root"
    tabTitle="Settings"
    tabIcon="cog"></ion-tab>
  <!--- ... -->
</ion-tabs>
```
---

### Build Settings Tab

__*app/pages/page1/page1.html*__
```html
<ion-navbar *navbar>
  <ion-title>Settings</ion-title>
</ion-navbar>

<ion-content padding class="page1">
  <ion-row>
    <ion-col width-50>
      <button (click)="checkForUpdate()">
        Check For Update
      </button>
    </ion-col>
    <ion-col width-50>
      <button (click)="sync()"
        [disabled]="!isUpdateAvailable">
        Sync
      </button>
    </ion-col>
  </ion-row>
</ion-content>

```

???

Here we are setting up some buttons and event handlers for Angular 2. If you are not familiar with the Angular 2 template syntax, (click) informs angular that we want to bind to the click event, and [disabled] means that we want to bind to a property on the class called isUpdateAvailable.

---
template: inverse

# Check For Update

---

template: false

### codePush.checkForUpdates

__*app/pages/page1/page1.ts*__

```js
// ...
export class Page 1 {
  // ...
  isUpdateAvailable:boolean = false;
  constructor(private appRef: ApplicationRef) { }
  checkForUpdate() {
    codePush.checkForUpdate((result) => {
      this.isUpdateAvailable = result !== null;
      this.appRef.tick();    
    });

  }
}

```

???

Here we are just hooking up the event handlers. Since codePush is running outside of the Angular 2 zone, we need to manually trigger the change detection with this.appRef.tick();

The result object will contain details of an update if one is available, if the result is null - it means that the
the application is up to date. We will look into the details of the result object later on in more detail.
---
template: inverse

# CodePush Sync

---

template: false

### codePush.sync
.code-header[__*app/pages/page1/page1.ts*__]
```js
// ...
export class Page1 {
// ...
sync() {
  codePush.sync(null, {
    updateDialog: true,
    installMode: InstallMode.IMMEDIATE
  });
}
```

???

Code push has a few methods that you can call with their SDK. codePush.sync however, will take care of many of the steps needed for hot-push for you. For the purposes of this example, we are telling codePush that we want to prompt the user with an update dialog, and that we want the application to restart immediately after it has been applied.

The first parameter that we pass in is a callback handler for different sync events, for now - we will be passing in null, but we will look at this in more detail later.

---
template: inverse

# Running the App

```shell
ionic emulate ios
```

???

This will build the iOS application, and launch the iOS simulator on your machine. At this point, we are not syncing up with code push at all. If we click on Check for Updates, we should not see the Sync button become enabled.

---
template: inverse

# Deploy with CodePush
[code-push release-cordova]

---

template: false

## Build & Deploy

* Build

```shell
ionic build
```

--

* Deploy

```shell
code-push release-cordova pgdays-demo ios
```

---

### New Feature: Status Field

__*app/pages/page1/page1.html*__
```html
<!-- ... -->
<ion-content padding class="page1">
  <ion-row>
     <ion-col width-20>Status:</ion-col>
     <ion col width-80>{{status}}</ion-col>
  </ion-row>
<!-- ... -->
</ion-content>
```

---
class: middle

.code-header[__*app/pages/page1/page1.ts*__]
```js
export class Page1 {
  status: string = '';
  isProcessing: boolean = false;
  // ..
  checkForUpdate() {
  this.isProcessing = true;
  codePush.checkForUpdate((result) => {
    this.isUpdateAvailable = result !== null;
    this.status = result === null ? 'Up to Date' : 'Update Available'
    this.isProcessing = false;
    this.appRef.tick();
    });
  }
  // ...
}
```

---


## Sync the Update

* Build & Release CodePush

```shell
ionic build
code-push release-cordova pgdays-demo ios
```

???

The CodePush plugin has a few options for syncing up updates. If we just call codePush.sync(), it will silently update the application, and the changes won't be reflected until the user closes the application and restarts it.

For this part of the workshop though, we will tell the plugin to notify the user of updates, and to restart the application after it has been installed.

--

* Sync the app

???

Once you have built and released the application, instead of running 'ionic emulate', do an ionic build and release with CodePush. You will now be able to go into your application and check for updates, then hit sync.

---


# Deployments

???

So far, we have just been pushing things into Staging, however CodePush allows you to have multiple Deployments running at the same time.

Lets start making some changes to our application be able to switch between a Production and a Staging deployment.

---

__*app/pages/page1/page1.html*__
```html
<!-- ... -->

<ion-list radio-group [(ngModel)]="deployment">
   <ion-list-header>
     Version
   </ion-list-header>

   <ion-item>
     <ion-label>Staging</ion-label>
     <ion-radio value="YOUR-IOS-STAGING-KEY"></ion-radio>
   </ion-item>

   <ion-item>
     <ion-label>Production</ion-label>
     <ion-radio value="YOUR-IOS-PRODUCTION-KEY"></ion-radio>
   </ion-item>
 </ion-list>
 <!-- ... -->
```

---

__*app/pages/page1/page1.html*__
```html
<!-- ... -->
<button
  (click)="checkForUpdate(deployment)"
  [disabled]="isInProgress">
    Check For Updates
</button>
<button
  (click)="sync(deployment)"
  [disabled]="!isUpdateAvailable">
    Sync
</button>
<!-- ... -->
```

---

__*app/pages/page1/page1.ts*__

```js
export class Page1 {
// ...

sync(key) {
  codePush.sync(null,
    { updateDialog: true,
      installMode: InstallMode.IMMEDIATE,
*     deploymentKey: key
    });
  }
}
```

---

__*app/pages/page1/page1.ts*__

```js
export class Page1 {
// ...

checkForUpdate(key) {
    this.isInProgress = true;
    this.status = 'Checking for Update';
    codePush.checkForUpdate((result) => {
      this.isUpdateAvailable = result !== null;
      this.isInProgress = false;
      this.status = result === null ? 'Application is up-to date' : 'Update Available';
      this.appRef.tick();
    },
*    null, // error handler
*    key); // deployment key is an optional 3rd parameter
  }
}
```

---
template: inverse
# LocalPackage Info

???

Before deploying our application to Staging and Production, lets take a look at LocalPackage info, and display some of the details on our screen so we can better see what is going on with our deployments once we start pushing to different environments.

---
template: none

### Display Current Package

__*app/pages/page1/page1.html*__
```html


<ion-list radio-group [(ngModel)]="deployment">
<!-- ... -->
  <ion-list>
    <ion-list-header>
      Installed Package
    </ion-list-header>
    <ion-item>
      App Version: {{currentPackage?.appVersion}}
    </ion-item>
    <ion-item>
      Description: {{currentPackage?.description}}
    </ion-item>
    <ion-item>
      Label: {{currentPackage?.label}}
    </ion-item>
 </ion-list>

```

---

__*app/pages/page1/page1.ts*__
```js
export class Page1 {
  // ...

  currentPackage: ILocalPackage

  ngOnInit() {
    this.getCurrentPackage();
  }  
  getCurrentPackage() {
    codePush.getCurrentPackage((result: ILocalPackage) => {
      this.currentPackage = result;  
      this.appRef.tick();
    })
  }  
}

```

---
template: inverse

# Deploy and Promote
[loading multiple versions as needed]

---

template: false

## Deploy

```shell
ionic build
code-push release-cordova pgdays-demo ios  \
  -d Staging \
  --description 'Deployment Select'
```

???

Lets push this change to Staging, and view it in our application.

--

## Promote

```shell
code-push promote pgdays-demo \
  Staging Production \
  --des 'Staging -> Production'
```

--

## Run the app ...

???

Lets go back into our application, and select 'Production' for our version and hit sync.
You'll notice that right now, it's saying 'Application is Upto Date', this is because we have moved the package from staging to production, and CodePush will consider it the same thing.

To see how we can switch between two different versions of the application, lets make a minor change to the app - make the 'install update' a green button, and push it to staging only.

---
template: inverse

# A Minor Change
---

template: false

### Change the Sync button colour
__*app/pages/page1/page1.html*__

```html
<button
  (click)="sync(deployment)"
  [disabled]="!isUpdateAvailable"
  secondary> <!-- ionic will set this to be green -->
    Sync
</button>
<!-- ... -->
```

---

### Build and Deploy to Staging

```shell
ionic build
code-push release-cordova \
  pgdays-demo ios \
  -d Staging \
  --description 'Color Change'
```

???

Lets go back to our application, and select 'Staging' as our environment, check for an update and install.
The application should restart, and the install update button should now be green.

Select Production again, check for update, and hit install update - you should now be back in the 'Production' deployment of the application, and the button should be blue.

--

### Run the app ...

* Select Production, and Sync
* Select Staging, and Sync

---

### Promote to Production

```shell
code-push promote pgdays-demo \
  Staging Production \
  --des 'Color Change to Production'
```

???

Finally, lets push this change to our Production branch, and check & install the update.

---
template: inverse

# Detailed Sync

???

So far, we have been leaving the callbacks for sync empty, lets take a closer look at the information that we can pull out from this method.
---
template: false

```js
codePush.sync(
*  syncCallback?: SuccessCallback<SyncStatus>,
  syncOptions?: SyncOptions,
  downloadProgress?: SuccessCallback<DownloadProgress>): void;
```

```js
declare enum SyncStatus {
    UP_TO_DATE,   
    UPDATE_INSTALLED,
    UPDATE_IGNORED,
    ERROR,
    IN_PROGRESS,
    CHECKING_FOR_UPDATE,
    AWAITING_USER_ACTION,
    DOWNLOADING_PACKAGE,
    INSTALLING_UPDATE
}
```

---
class: middle

__*app/pages/page1/page1.ts*__

```js
// ...
export class Page 1 {

syncHandler(status: SyncStatus) {
  // ...
}
sync(key) {
    codePush.sync((status) => this.syncHandler(status),
      {
        updateDialog: true,
        installMode: InstallMode.IMMEDIATE,
        deploymentKey: key
      });
  }
```
---

class: middle

```js
syncHandler(status: SyncStatus) {
    switch (status) {
      case SyncStatus.UP_TO_DATE:
        this.status = 'Up-to-date';
        break;
      case SyncStatus.UPDATE_INSTALLED:
        this.status = 'Update Installed';
        break;
      case SyncStatus.UPDATE_IGNORED:
        this.status = 'Update Ignored';
        break;
      case SyncStatus.ERROR:
        this.status = 'Error';
        break;
      // ...
  }
```

---
class: middle

```js
syncHandler(status: SyncStatus) {
  switch(status) {
    // ...
    case SyncStatus.IN_PROGRESS:
      this.status = 'In Progress';
      break;
    case SyncStatus.CHECKING_FOR_UPDATE:
      this.status = 'Checking for Update';
      break;
    case SyncStatus.AWAITING_USER_ACTION:
      this.status = 'Awaiting User Action';
      break;
    case SyncStatus.DOWNLOADING_PACKAGE:
      this.status = 'Downloading Package';
      break;
    case SyncStatus.INSTALLING_UPDATE:
      this.status = 'Installing Update';
      break;

  }
  this.appRef.tick();
  }
// ...
}
```

---
template: inverse

# Download Progress

---

template: false


```js
codePush.sync(
  syncCallback?: SuccessCallback<SyncStatus>,
  syncOptions?: SyncOptions,
* downloadProgress?: SuccessCallback<DownloadProgress>): void;
```
--

```js
/**
 * Defines the format of the DownloadProgress object,
 * used to send periodical update notifications on the progress
 * of the update download.
 **/
interface DownloadProgress {
    totalBytes: number;
    receivedBytes: number;
}
```

---
class: middle

```js
// ...
export class Page1 {
  downloadProgress: DownloadProgress
  // ...

  updateDownloadProgress(downloadProgress: DownloadProgress) {
    this.downloadProgress = downloadProgress;
    this.appRef.tick();
  }

  sync(key) {
     codePush.sync((status) => this.syncHandler(status),
       {
         updateDialog: true,
         installMode: InstallMode.IMMEDIATE,
         deploymentKey: key
       },
       (progress) => this.updateDownloadProgress(progress));
   }
}
```

---
class: middle

```html
<!-- .... after the sync buttons --->
<ion-row>
  <ion-col width-25>Progress:</ion-col>
  <ion-col width-75>
      {{downloadProgress?.receivedBytes}} / {{downloadProgress?.totalBytes}}
    </ion-col>
  </ion-row>
```

???

TODO: Steps / Slides about promoting / viewing changes / etc?
Or - move this stuff above 'Deployments' 
---
template: inverse

# Thanks

---
template: inverse

# Any Questions?
