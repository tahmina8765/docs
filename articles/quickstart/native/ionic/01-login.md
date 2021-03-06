---
title: Login
default: true
description: This tutorial demonstrates how to add authentication and authorization to an Ionic app
budicon: 448
---

<%= include('../../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-ionic-samples',
  path: '01-Login',
  requirements: [
    'Ionic 1.x',
    'AngularJS 1.5+'
  ]
}) %>

<%= include('../_includes/_ionic_setup') %>

## Set Up URL Redirects

Use the `onRedirectUri` method from **auth0-cordova** when your app loads to properly handle redirects after authentication.

```js
// www/js/app.js

var Auth0Cordova = require('@auth0/cordova');

angular.module('yourApp', ['ionic'])

.run(function($ionicPlatform) {
  $ionicPlatform.ready(function() {
    
    // ...
    function handleUrl(url) {
      Auth0Cordova.onRedirectUri(url);
    }

    window.handleOpenURL = handleUrl;

  });
})

// ...
```

::: note
**Note:** The code samples shown in this tutorial assume that your app is using some kind of bundler like browserify or webpack to make npm modules available in a client application. The downloadable sample for this tutorial uses browserify to quickly bundle the JavaScript files for the app but you are free to use whichever bundler you like.
:::

## Create an Authentication Service and Configure Auth0

To coordinate authentication tasks, it's best to set up an injectable service that can be reused across the application. This service needs methods for logging users in and out, as well as checking their authentication state. Be sure to replace `YOUR_PACKAGE_ID` with the identifier for your app in the configuration block.

${snippet(meta.snippets.use)}

## Add Login and Logout Controls

Add controls to your app to allow users to log in and log out. The controls should call the `login` and `logout` methods from the `Auth` service. Start by injecting the `Auth` service in a controller.

```js
// www/js/controllers.js

angular.module('starter.controllers', [])

.controller('HomeCtrl', function($scope, Auth) {
  $scope.auth = Auth;
})
```

The `Auth` service is now accessible in the view and its `login` method can be called.

```html
<!-- www/templates/tab-home.html -->

<button
  ng-if="!auth.isAuthenticated()"
  class="button button-block button-positive"
  ng-click="auth.login()">
    Log In
</button>

<button
  ng-if="auth.isAuthenticated()"
  class="button button-block button-assertive"
  ng-click="auth.logout()">
    Log Out
</button>
```

The **Log In** button is only displayed if the user has an unexpired `access_token` in local storage which is the indication that they are authenticated. The **Log Out** button is only displayed if the user isn't authenticated. 

## Display Profile Data

Your application will likely require some kind of profile area for users to see their information. Depending on your needs, this can also serve as the place for them to log out of the app.

Create a controller for a **Profile** area and make use of the `Auth` service within it.

```js
// www/js/controllers.js

angular.module('starter.controllers', [])

// ...
.controller('ProfileCtrl', function($scope, Auth) {
  $scope.auth = Auth;

  if (Auth.isAuthenticated()) {
    if (Auth.userProfile) {
      $scope.profile = Auth.userProfile;
    } else {
      
      Auth.getProfile(function(err, profile) {
        if (err) {
          return alert(err);
        }
        $scope.$evalAsync(function() {
          $scope.profile = profile;
        });
      });
    }
  }

});
```

The `getProfile` method from the `Auth` service makes a call to Auth0 for the user's profile and saves the result in a `userProfile` object in the `Auth` service.

The `ProfileCtrl` controller first checks whether the `userProfile` object is populated. If it is, that object is used to power the **Profile** view. If it isn't, a call to Auth0 is made for the profile.

Create a view for the profile.

```html
<!-- www/templates/tab-profile.html -->

<ion-view view-title="Profile">
  <ion-content class="padding">
    <div ng-if="!auth.isAuthenticated()">
      <p>Please log in to view your profile.</p>
    </div>
    <div ng-if="auth.isAuthenticated()" class="list card">
      <div class="item item-avatar">
        <img src="{{ profile.picture }}" alt="avatar">
        <h3>{{ profile.name }}</h3>
        <h4>{{ profile.nickname  }}</h4>
      </div>
      <div class="item item-body">
        <pre>{{ profile | json }}</pre>
      </div>
    </div>
  </ion-content>
</ion-view>
```

### Troubleshooting

#### Cannot read property 'isAvailable' of undefined

This means that you're attempting to test this in a browser. At this time you'll need to run this either in an emulator or on a device.
