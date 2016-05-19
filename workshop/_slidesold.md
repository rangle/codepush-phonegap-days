# Another Update

__*app/pages/tabs/tabs.html*__
```html
<ion-tabs>
  <!-- ... -->
  <ion-tab [root]="tab2Root"
    tabTitle="Code Pushed!"
    tabIcon="chatbubbles">
  </ion-tab>
  <!-- ... -->
</ion-tabs>
```

--

# Build and Publish

```shell
ionic build
code-push release-cordova pgdays-demo ios
```

---

# Re-open the Application

* Close the Application : ⌘ + ⇧ + HH
* Re-open the application

???

At this point, we now have the ionic application installed on our emulated device, and hooked up into the codePush.sync(). Lets change the name of another tab, and publish an update with code push.

When we start the application again, we should be seeing a prompt that an update is available, and asking us if we want to install it or not.

If we had just called codePush.sync() it would have silently updated the application, and had it applied after the app restarted. However, since we passed updateDialog: true, and installMode: IMMEDIATE, it will prompt us for the update, and restart the application right after it's installed.

---

# Code Push API

* checkForUpdate
* getCurrentPackage
* notifyApplicationReady
* restartApplication
* sync

???

CodePush can provide quite a bit of flexibility with how you interact with it, and has methods exposed to check for updates, retrieve the current package, etc. Most of this is wrapped up with you in a convince method `sync`.

---
template: inverse
# Controlling the Sync

???

Instead of just having the sync run at the start of the application, lets start updating one of the tabs with some controls to handle the sync for us to explore the options of codeSync a little more in depth.

For now, comment out the .sync in our app.ts, and lets make a 'settings' tab in our application

---
# Rename Tab 3

__*app/pages/tabs/tabs.html*__
```html
<ion-tabs>
  <!-- ... -->
  <ion-tab [root]="tab3Root"
    tabTitle="Settings"
    tabIcon="cog">
  </ion-tab>
</ion-tabs>
```

---

# Create a Settings Tab

__*app/pages/page1/page1.html*__
```html
<ion-navbar *navbar>
  <ion-title>
    Settings
  </ion-title>
</ion-navbar>

<ion-content class="page3">
  <ion-item>
    <ion-label>Status: {{status}} </ion-label>  
  </ion-item>

  <button (click)="checkForUpdate()"
    [disabled]="isInProgress">Check For Updates</button>
  <button (click)="installUpdate()"
    [disabled]="!isUpdateAvailable">Install Update</button>
</ion-content>

```

???

Here we will add buttons to check for an update, and one to install an update if one is available.
Next, lets hook up codepush.

---

__*app/pages/page3/page3.ts*__

```js
/// <reference path="../../codePush.d.ts"/>
import {Page} from 'ionic-angular';
import {ApplicationRef} from 'angular2/core';

declare let codePush: CodePushCordovaPlugin;
@Page({
  templateUrl: 'build/pages/page3/page3.html'
})
export class Page3 {
  isUpdateAvailable: Boolean = false;
  isInProgress: Boolean = false;
  status: string = '';
  constructor(private appRef:ApplicationRef) { /* ... */ }

  installUpdate() { /* ... */ }
  checkForUpdate() { /* ... */ }

}

```
???
Here, we will be importing ApplicationRef from the angular core. Since codeSync is running outside of the Angular 2 zone, it's not always aware of when our API calls complete. Since we are defining it as private in our constructor, it will automatically be set to `this.appRef` -

---
# checkForUpdate

__*app/pages/page3/page3.ts*__

```js
export class Page3 {
// ...

checkForUpdate() {
  this.isInProgress = true;
  this.status = 'Checking for Update';
  codePush.checkForUpdate((result) => {
    this.isUpdateAvailable = result !== null;
    this.isInProgress = false;
    this.status = result === null ?
      'Application is up-to date' : 'Update Available';
    this.appRef.tick();
    });
  }
}

```

???

The result object for checkForUpdate has quite a bit of details in it, however our main concern right now is knowing if one is available or not. If the result is null, no update is available. Since Angular doesn't know about codePush's async options, we need to do an appRef.tick() to force Angular 2 to do it's change detection.


TODO: make a service wrapper for codePush?

---
# installUpdate

__*app/pages/page3/page3.ts*__

```js
export class Page3 {
// ...

installUpdate() {
  codePush.sync(null,{installMode: InstallMode.IMMEDIATE});
  }
}
```
???

Just like in our app.ts, we will use codePush.sync to install the update, and we will make it IMMEDIATE.

---
