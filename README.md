# AngularJS Style Guide for CoffeeScript

Adapted from this [guide](https://github.com/Plateful/plateful-mobile/wiki/AngularJS-CoffeeScript-Style-Guide) by [@JoelCox](//twitter.com/joelcoxokc), which in turn is based on [this guide](https://github.com/johnpapa/angular-styleguide) by [@john_papa](//twitter.com/john_papa).

We use Angular together with a Rails backend, so some naming conventions are chosen to be similar to Rails.


## Table of Contents

  1. [Single responsibility](#single-responsibility)
  1. [Directory structure](#directory-structure)
  1. [Modules](#modules)
  1. [Manual dependency injection](#manual-dependency-injection)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Directives](#directives)
  1. [Promises](#promises)
  1. [Custom form elements](#custom-form-elements)

## Single responsibility

- **Rule of 1**: Define 1 component per file.

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

  ```coffee
  # AVOID
  @app = angular.module('app', ['ngRoute'])

  @app.controller 'SomeController', [->
  ]

  @app.factory 'SomeFactory', SomeFactory [->
  ]
  ```

  The same components are now separated into their own files.

  ```coffee
  # recommended
  # app.coffee
  @app = angular.module('app', ['ngRoute'])
  ```

  ```coffee
  # recommended
  # some_controller.coffee
  @app.controller 'SomeController', [->
  ]
  ```

  ```coffee
  # recommended
  # some_factory.coffee
  @app.factory 'SomeFactory', SomeFactory [->
  ]
  ```

**[Back to top](#table-of-contents)**


## Directory structure

- **Organize by type, then namespace**:

  ```
  app
  ├── app.js
  ├── routes.js
  ├── controllers
  │   ├── about
  │   │   └── show_controller.coffee
  │   └── pages
  │       ├── form_controller.coffee
  │       └── index_controller.coffee
  ├── directives
  │   ├── forms
  │   │   ├── date_picker.coffee
  │   │   └── multi_select.coffee
  │   ├── pages
  │   │   └── slug_chooser.coffee
  │   └── some_shared_directive.coffee
  ├── filters
  │   └─── some_filter.coffee
  └── services
      ├── models
      │   ├── Page.coffee
      │   └── User.coffee
      └── some_shared_service.coffee
  ```


## Modules

- **Definitions**: Use a global variable `@app` to hold your application module. Define it once. Use it for to define controllers, services and directives.

  *Why?*: We can afford to add one global variable. It makes other definitions more readable.

  ```coffee
  # AVOID
  angular.module('app')
    .controller 'SomeController', [->
    ]
  ```

  Instead, simply define on `@app`.

  ```coffee
  # recommended
  # app.coffee
  angular.module 'app', [
    'ngAnimate'
    'ngRoute'
  ]
  ```

  ```coffee
  # recommended
  # some_controller.coffee
  @app.controller 'SomeController', [->
  ]
  ```

- **IIFE**: There is no need to wrap coffeescript code into IIFEs (immediately invoked function expressions). Coffeescript does this for you.

  *Why?*: Coffeescript already does this for you.

  ```coffee
  # AVOID
  (->
    @app.factory 'logger', [->
    ]
  )()
  ```

**[Back to top](#table-of-contents)**


## Manual dependency injection

  - **Annotate dependencies**: Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach. Instead, use the "array method" of declaring dependencies.

    *Why?*: We don't have an automated way to annotate dependencies. Using the "array method" keeps the annotation and the actual parameters on the same line, so we are less likely to forget to keep them nin sync.

    ```coffee
    # AVOID
    @app.controller 'DashboardController', (Common, Data) ->
      # ...
    ```

    instead:

    ```coffee
    # recommended
    @app.controller 'DashboardController', ['Common', 'Data', (Common, Data) ->
      # ...
    ]
    ```


**[Back to top](#table-of-contents)**


## Controllers

- **controllerAs**: Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the "classic controller with $scope" syntax. 

  *Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`.

  *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

  *Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

  So, instead of:

  ```html
  <!-- AVOID -->
  <div ng-controller="Customer">
    {{ name }}
  </div>
  ```

  ```coffee
  # AVOID
  @app.controller 'CustomerController', ['$scope', ($scope) ->
    $scope.name = {}
    $scope.sendMessage = ->
  ]
  ```

  do this:

  ```html
  <!-- recommended -->
  <div ng-controller="CustomerController as customer">
    {{ customer.name }}
  </div>
  ```

  or, if activated by a route

  ```coffee
    $stateProvider
      .state 'customer.show',
        templateUrl: 'customer/show'
        controller: 'CustomerController as customer'
  ```

  ```coffee
  # recommended, but see below for the **init** function
  @app.controller 'CustomerController', [->
    @name = {}
    @sendMessage = =>

    return
  ]
  ```

  Note that you need an explicit `return`, see below.



- **Bindable Members Up Top**:

  Place bindable members at the top of the controller in an `init` function, alphabetized, and not spread through the controller code.

  *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

  *Why?*: Setting anonymous functions inline can be easy, but when those functions are more than 1 line of code they can reduce the readability.

  *Why?*: In coffeescript you cannot use hoisted function declarations, and hence you have to use something like the `init` function.

  ```coffee
  # AVOID
  @app.controller 'SessionsController', [->
    @signIn = ()=>
      # ...

    @signOut = ()=>
      # ...

    @currentUser = null
  ]
  ```

  ```coffee
  # recommended
  @app.controller 'SessionsController', [->

    init = =>
      @currentUser = null
      @signIn = signIn
      @signOut = signOut


    signIn = =>
      # ...

    signOut = =>
      # ...


    init()
    return
  ]
  ```

- **Private functions**:

  Keep private functions at the bottom and do not expose them:

  ```coffee
  # recommended
  @app.controller 'SessionsController', [->

    init = =>
      @signIn = signIn


    signIn = =>
      request('/sign_in')


    # private

    request = =>
      # ...


    init()
    return
  ]
  ```

- **Use services**:

  Defer logic in a controller by delegating to services and factories.

  *Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

  *Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

  *Why?*: Removes dependencies and hides implementation details from the controller.

  ```coffee
  # AVOID
  @app.controller 'Order', ['$http', '$q', ($http, $q) ->

    init = =>
      @checkCredit = checkCredit
      @total = 0


    checkCredit = =>
      orderTotal = @total
      $http.get('api/creditcheck').then (data)=>
        remaining = data.remaining
        return $q.when(!!(remaining > orderTotal))


    init()
    return
  ]
  ```

  ```coffee
  # recommended
  @app.controller 'OrderController', ['CreditService', (CreditService) ->

    init = =>
      @checkCredit = checkCredit
      @total = 0


    checkCredit = =>
      CreditService.check()


    init()
    return
  ]
  ```

**[Back to top](#table-of-contents)**


## Services

- **Generally, use `factory` where possible**. Avoid using `service` or `provider`.

- **Single Responsibility**: Factories should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

- **Instances**: If you need a service that can be instantiated, return a coffeescript class:

  ```coffee
  # recommended
  @app.factory 'SomeFactory', [->
    class
      constructor: (options) ->

      method: ->
  ]
  ```

- **Singletons: public members on top**: If you need a singleton, write the service like a controller. Angular will instantiate it only once. Place public members on top.

  ```coffee
  # AVOID
  @app.factory 'DataService', [->

    someValue = ''


    return
      save: ()->
       # . #

      validate: ()->
       # . #
  ]
  ```

  ```coffee
  # recommended
  @app.factory 'DataService', [->

    init = =>
      @save = save
      @someValue = ''
      @validate = validate


    save = =>
      # ...

    validate = =>
      # ...


    init()
    return
  ]
  ```

**[Back to top](#table-of-contents)**

## Directives
- **Limit 1 Per File**:

  Create one directive per file. Name the file for the directive.


- **Limit DOM Manipulation**:

  When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hide and shows, use ngHide/ngShow, but if the directive does more, combining hide and show inside a directive may improve performance as it reduces watchers.

    *Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templating)


- **Restrict to either Elements or Attributes**:

  When creating a directive that makes sense as a standalone element, use `restrict: 'E'`.

  When creating a directive that enhances an existing element, use `restrict: 'A'`.


- **Use controller, controllerAs and bindToController**:

  All the interaction between the directive's template and all application should use a controller. The `controllerAs` and `bindToController` syntax makes this consistent with regular controllers.

  Always use `controllerAs: 'vm'` (short for "view model").

  Bindings are automatically bound to the controller instance when using ".

  Sometimes the controller has to manipulate the DOM in response to some user interaction. This is okay. Simply inject `$element` into the controller.

  If you need to set up events, do initial DOM maniuplation, or set up watches do this inside the link function. If you have a controller, this is injected as the 4th parameter into your link function.

  *Note*: Doing this only works well with an isolate scope. You should always use isolate scopes, unless there is a very compelling reason to do otherwise.

  TODO

  ```coffee

  ```


- **Prefix directives**:

  Use a short, common prefix for directives, so you can easily distinguish your own directives from third party directives.

  ```coffee
  # recommended
  @app.directive 'acmeMultiSelect', [->
    # ...
  ]
  ```

- **Inline templates**:

  Only use inline templates for templates shorter than about 10 lines.

  Use `"""` strings for inline templates.

  ```coffee
  # AVOID
  @app.directive 'acmeMultiSelect', [->
    # ...
    template: '<div class="container"><div ng-repeat="item in items">...</div></div>'
  ]
  ```

  ```coffee
  # recommended
  @app.directive 'acmeMultiSelect', [->
    # ...
    template: """
              <div class="container">
                <div ng-repeat="item in items">
                  ...
                </div>
              </div>
              """
  ]
  ```

- **External templates**

  For longer templates, use `templateUrl` and an external template. Place directive templates in a
  `directive` directory.

  ```coffee
  # recommended
  @app.directive 'acmeMultiSelect', [->
    # ...
    templateUrl: 'directives/acme_multi_select'
  ]
  ```

- **When it makes sense, try to keep directives generic**:

  *Why?* It allows you to share directives between projects.

  *Why?* It naturally leads to better decoupled code.

  This often means injecting context through your templates.

  ```coffee
  # AVOID
  @app.directive 'acmeNoOrders', [->
    restrict: 'E'
    transclude: true
    template: """
              <div ng-class="{ 'has-no-orders': orders.length == 0}">
                <div ng-if="orders.length == 0">
                  No orders found.
                </div>
                <span ng-if="orders.length > 0" ng-transclude></span>
              </div>
              """
  ]
  ```
  ```html
    <!-- AVOID -->
    <acme-no-orders>
      <table>
        ...
      </table>
    </acme-no-orders>
  ```

  instead:

  ```coffee
  # recommended
  @app.directive 'acmeNoContent', [->
    restrict: 'E'
    transclude: true
    scope:
      content: '='
      whenEmpty: '@'
    template: """
              <div ng-class="{ 'has-no-content': content.length == 0}">
                <div ng-if="content.length == 0">
                  {{ whenEmpty }}
                </div>
                <span ng-if="content.length > 0" ng-transclude></span>
              </div>
              """
  ]
  ```
  ```html
    <!-- recommended -->
    <acme-no-content content="orders" whenEmpty="No orders found.">
      <table>
        ...
      </table>
    </acme-no-content>
  ```

**[Back to top](#table-of-contents)**


## Promises

- **Always use promises instead of callbacks**:

  *Why?*: This is the accepted way of doing callbacks. Promises can be chained.

  ```coffee
  # AVOID
  @app.Service 'Session', [->
    # ...

    signIn: (credentials, success, failure) ->
      # ...
      if signedIn
        success()
      else
        failure()
    # ...
  ]

  #...

  Session.signIn credentials, ->
    console.log('signed in')
  , ->
    console.log('could not sign in')
  ```

  instead:

  ```coffee
  # recommended
  @app.Service 'Session', ['$q', ($q) ->
    # ...

    signIn: (credentials) ->
      $q (resolve, reject) ->
        if signedIn
          resolve()
        else
          reject()
    # ...
  ]

  Session.signIn(credentials).then ->
    console.log('signed in')
  , ->
    console.log('could not sign in')
  ```


- **Prefer the closure way of using $q, when possible**:

  *Why?*: Easier to understand.

  ```coffee
  # AVOID
  ```


## Custom form elements

- **Always depend on ng-model and use its API**

  TODO


## Performance

- **One-time bindings**:

  Use the one-time binding syntax `{{ ::value }}` when possible.

  ```html
  <!-- AVOID -->
  <h1>{{ page.title }}</h1>
  ```

  (assuming `page.title` does not change)

  ```html
  <!-- recommended -->
  <h1>{{ ::page.title }}</h1>
  ```
