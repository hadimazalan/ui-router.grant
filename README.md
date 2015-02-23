# AngularUI Router Grant

> This module is still in development, but fully functional. Just need some time to shore up the tests, and documentation.

#### Easily protect your ui-router states with this angular module

The UI Router Grant module provides a quick and easy solution for adding test(s) to your ui-router states. For example if you wanted to restrict certain states to authenticated users, ui-router.grant is an easy way to get this working. For more details check out the [demo]() or [getting started]() section.

* [Demo]()
* [Installation]()
* [Getting Started]()


// uses ui-router resolve
// easily add single or multiple tests for each one of your states, will also redirect
// assumes that you are familiar with ui-router and resolves



## Installation

- via **[Bower](http://bower.io/)**: by running `$ bower install angular-ui-rotuer-grant` from your console
- or via **[npm](https://www.npmjs.org/)**: by running `$ npm install angular-ui-rotuer-grant` from your console

#### Using < IE9
The module takes advantage of [Array.prototype.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) and [Array.prototype.some()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/some) which are not available on older browsers. The good news is both these methods can be easily [polyfilled](https://github.com/es-shims/es5-shim).


Once you have successfully installed the module your setup should look similar to this:

```html
<!doctype html>
<html ng-app="myApp">
<head>
  <script src="//cdnjs.cloudflare.com/ajax/libs/angular.js/1.3.12/angular.min.js"></script>
  <script src="//cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.2.13/angular-ui-router.min.js"></script>
  <script src="js/angular-ui-rotuer-grant.js"></script>
    <script>
        var app = angular.module('app', ['ui.router.grant']);
    </script>
    ...
</head>
<body>
    ...
</body>
</html>
```



## Getting Started

The ui.router.grant module is primarily made up off two core angular services `grant` and `GrantTest`. The quickest way to see how the grant module works is to jump into an example. Let's assume that we have an app with the following ui-router states available:

```javascript
$stateProvider

    .state('user-only', {
      url: '/users',
      templateUrl: 'partials/only-user.html'
    })

    .state('admin-only', {
      url: '/admins',
      templateUrl: 'partials/only-admin.html'
    })

    .state('except-user', {
      url: '/no-users',
      templateUrl: 'partials/except-user.html'
    })

    .state('combined', {
      url: '/combined',
      templateUrl: 'partials/combined.html'
    })

    .state('denied', {
      url: '/denied',
      templateUrl: 'partials/denied.html'
    })
```

The following states are currently accessible by any user, but we would like to enforce the following rules:

* `user-only` can only be accessed by *users*
* `admin-only` can only be accessed by *admins*
* `except-user` can be accessed by anyone except *users*
* `combined` can only be accessed by someone who is both a *user* and *admin*
* 'denied' will be where users are redirected when failing a grant test

Creating tests with the grant module is easy. In order to enforce the above rules we need to create tests that will determine if someone is a **user** or **admin**.

```javascript
app.module('app', ['ui.router.grant'])

.run(function(grant, userService, adminService) {

  /**
   * A test is very simple and takes two params.
   * @param  {String}     testName - A unique id for the test.
   * @param  {Function}   validate - A function that will validate whether your test passes or fails.
   */
  grant.addTest('user', function() {
    // In this example lets assume that userService is making a request
    // a RESTful service to retreive the current user and will return a promise.
    // If the user exists promise will resolve and test will pass.
    // If the user doesn't exist promise will reject and test will fail.
    return userService.getUser();
  });

  grant.addTest('admin', function() {
    // Instead of a promise you can also return a synchronous value.
    // If the returned value evaluates to true test will pass.
    // If the returned value evaluates to false test will fail.
    return isAdmin;
  });

});

```



#### grant.only(options) single test

Now that the tests are created let's use them to protect our states. Start by restricting access to the 'user-only' and 'admin-only' states.


>
The options param can either be a single test object, or an array of test objects if there are [multiple tests](). Each test object has a *test* (test name) and *state* (ui-router state the user will be redirected to on test fail) property.

```javascript
.state('user-only', {
  url: '/users',
  templateUrl: 'partials/only-user.html',
  controller: function(user) {
    // user would be the value returned from the user grant test's validate funciton
    var newUser = user;
  },
  resolve: {
    user: function(grant) {
      return grant.only({test: 'user', state: 'denied'});
    }
  }
})

.state('admin-only', {
  url: '/admins',
  templateUrl: 'partials/only-admin.html',
  resolve: {
    admin: function(grant) {
      return grant.only({test: 'admin', state: 'home'});
    }
  }
})
```


#### grant.only(options) multiple tests

To protect the `combined` state we need to add both the *user* and *admin* tests to the `grant.only` call.

>
It's important to note that grant's with multiple asynchronous tests may not resolve/reject in the order they are listed. For example let's say that a user fails both the *user* and *admin* tests, but the admin test rejects before the user test. The user will actually be redirected to the admin fail state even though it is listed second.

```javascript
.state('combined', {
  url: '/combined',
  templateUrl: 'partials/combined.html'
  controller: function(combined) {
    // combined will be an array of the values returned from grant.only
    // combined[0] - value returned from user test
    // combined[1] - value returned from admin test
    var newUser = combined[0];
    var newAdmin = combined[1];
  },
  resolve: {
    combined: function(grant) {
      return grant.only([
        {test: 'user', state: 'denied'},
        {test: 'admin', state: 'home'}
      ]);
    }
  }
})
```


#### grant

The grant service is in charge of managing and executing all your grant tests.

**grant.addTest(testName, validationFunction)**

**grant.hasTest(testName)**

**grant.only(testName)**



