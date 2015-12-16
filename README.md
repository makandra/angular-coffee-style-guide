# ========== THIS IS A DRAFT ===========

# AngularJS Style Guide for CoffeeScript

Adapted from this [guide](https://github.com/Plateful/plateful-mobile/wiki/AngularJS-CoffeeScript-Style-Guide) by [@JoelCox](//twitter.com/joelcoxokc), which in turn is based on [this guide](https://github.com/johnpapa/angular-styleguide) by [@john_papa](//twitter.com/john_papa).

We use Angular together with a Rails back-end, so some conventions are chosen to be similar to Rails.


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

- **Rule of 1**

  Define 1 component per file.

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

  ```coffee
  # AVOID
  @app = angular.module('app', ['ngRoute'])

  @app.controller 'SomeController', [->
  ]

  @app.factory 'SomeFactory', [->
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
  @app.factory 'SomeFactory', [->
  ]
  ```

**[Back to top](#table-of-contents)**


## Directory structure

- **Organize by type, then namespace**

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

- **Definitions**

  Use a global variable `@app` to hold your application module. Define it once. Use it to define controllers, services and directives.

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
  @app = angular.module 'app', [
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

- **Namespaces**

  Organize controllers in namespaces where reasonable. The namespaces are reflected in the directoy structure as well as the name. Use Ruby notation (`::`) to separate namespaces.

  ```coffee
  # recommended
  @app.controller 'Page::IndexController', [->
    # ...
  ]
  ```

- **IIFE**

  There is no need to wrap CoffeeScript code into IIFEs (immediately invoked function expressions). CoffeeScript does this for you.

  *Why?*: CoffeeScript already does this for you.

  ```coffee
  # AVOID
  (->
    @app.service 'logger', [->
    ]
  )()
  ```

**[Back to top](#table-of-contents)**


## Manual dependency injection

- **Annotate dependencies**

  Declare injected dependencies using the array syntax. Do not use the shorthand syntax.
  Keep annotation and actual parameters on the same line.

  *Why?*: Minification renames method arguments which breaks injection. As a workaround for this, Angular supports listing injected dependencies as strings (which are never minified). We do not have an automated way to annotate dependencies before minification, so you need to define them yourself.

  *Why?*: Keeping annotation and actual parameters on the same line reduces the risk of accidentially updating only one of them.

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

- **Use controllerAs**

  Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the "classic controller with $scope" syntax.

  *Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the "classic `$scope` syntax".

  *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

  *Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

  So, instead of:

  ```coffee
  # AVOID
  @app.controller 'CustomerController', ['$scope', ($scope) ->
    $scope.name = {}
    $scope.sendMessage = ->
  ]
  ```

  ```html
  <!-- AVOID -->
  <div ng-controller="CustomerController">
    {{ name }}
  </div>
  ```

  do this:

  ```coffee
  # recommended
  @app.controller 'CustomerController', [->
    @name = {}

    @sendMessage = =>

    return
  ]
  ```

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

  **Note that you need an explicit `return` at the end.**


- **Organziation of controller function**

  In the controller, put things in the following order:

  - Private variables
  - Bindable attributes (exposed to the view)
  - Initialization logic (see below)
  - Bindable methods (exposed to the view)
  - Private functions

  *Why?*: Placing bindable members near the top makes it obvious what is used in the view.

  ```coffee
  # recommended
  @app.controller 'SessionsController', [->

    # private variables

    alreadySignedIn = false

    # bindable attributes

    @currentUser = null

    # bindable methods

    @signIn = =>
      unless alreadySignedIn
        request('/sign_in').then (response) =>
          @currentUser = user
          alreadySignedIn = true

    @signOut = =>
      # ...

    # private functions

    request = =>
      # ...

    return
  ]
  ```


- **Initialization logic**

  Logic that needs to run when instantiating the controller (like loading some
  data) goes into an `init` function, placed between the public attributes and the public methods. Complex logic should go into a private function (or a service).

  The `init` function is called before the bottom `return`.

  *Why?*: Initialization happens first and so should be at the top of the controller. It needs to be wrapped in a function and called at the bottom so that everything else is already defined.

  ```coffee
  # recommended
  @app.controller 'UserController', ['$routeParams', 'User', ($routeParams, User) ->

    # bindable attributes

    @user = null

    # initialization

    init = =>
      User.find($routeParams.userId).then (user) =>
        @user = user

    # bindable methods

    @destroy = =>
      @user.destroy()
      #...

    init()
    return
  ]
  ```


- **Use services**

  Defer logic in a controller by delegating to services and factories.

  *Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

  *Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

  *Why?*: Removes dependencies and hides implementation details from the controller.

  ```coffee
  # AVOID
  @app.controller 'Order', ['$http', '$q', ($http, $q) ->

    @total = 0

    @checkCredit = =>
      orderTotal = @total
      $http.get('api/creditcheck').then (data)=>
        remaining = data.remaining
        return $q.when(!!(remaining > orderTotal))

    return
  ]
  ```

  ```coffee
  # recommended
  @app.controller 'OrderController', ['CreditService', (CreditService) ->

    @total = 0

    @checkCredit = =>
      CreditService.check()

    return
  ]
  ```

- **No watches in controllers**

  Try to avoid all watches in controllers. Write code instead. If you have to
react to user input, use `ngChange` or similar.

  *Why?*: Code is easier to reason about, if there are no (maybe unexpected) watches.

  ```coffee
  # AVOID
  @app.controller 'UserInfoController', ['$scope', 'User', ($scope, User) ->

    @user = null
    @userId = null

    init = =>
      $scope.$watch =>
        @userId
      , (userId) =>
        @user = User.find(userId)

    init()
    return
  ]
  ```
  ```html
  <!-- AVOID -->
  <select ng-model='userInfo.userId' ng-options='...'></select>
  <div class="user-info">
    Name: {{ userInfo.user.name }}
  </div>
  ```

  instead:

  ```coffee
  # recommended
  @app.controller 'UserInfoController', ['$scope', 'User', ($scope, User) ->

    @user = null
    @userId = null

    @userIdChanged = =>
      @user = User.find(@userId)

    return
  ]
  ```
  ```html
  <!-- recommended -->
  <select ng-model='userInfo.userId' ng-change='userInfo.userIdChanged()' ng-options='...'></select>
  <div class="user-info">
    Name: {{ userInfo.user.name }}
  </div>
  ```

**[Back to top](#table-of-contents)**


## Services

- **Single Responsibility**

  Services should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

- **Instances**

  If you need a service that can be instantiated, use `factory` and return a CoffeeScript class:

  ```coffee
  # recommended
  @app.factory 'SomeFactory', [->
    class
      constructor: (options) ->

      method: ->
  ]
  ```

- **Singletons: public members on top**

  If you need a singleton, use `service` and write the service like a controller. Angular will instantiate it only once. Place public members on top.

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
  @app.service 'DataService', [->

    # private variables

    someValue = ''

    # public methods

    @save = =>
      # ...

    @validate = =>
      # ...

    return
  ]
  ```

  **Note, again, the explicit `return` at the end.**

**[Back to top](#table-of-contents)**

## Directives
- **Limit 1 Per File**

  Create one directive per file. Name the file for the directive.


- **Limit DOM Manipulation**

  When manipulating the DOM directly, use a directive. If alternative ways can
  be used such as using CSS to set styles or the [animation
  services](https://docs.angularjs.org/api/ngAnimate), Angular templating,
  [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or
  [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those
  instead. For example, if the directive simply hide and shows, use
  ngHide/ngShow, but if the directive does more, combining hide and show inside a
  directive may improve performance as it reduces watchers.

  *Why?*: DOM manipulation can be difficult to test, debug, and there are often
  better ways (e.g. CSS, animations, templating).


- **Restrict to either Elements or Attributes**

  When creating a directive that makes sense as a standalone element, use `restrict: 'E'`.

  When creating a directive that enhances an existing element, use `restrict: 'A'`.

  Never use `restrict: 'C'`.
  *Why?*: Classes are for styling, not for logic.

- **Avoid using controllerAs and bindToController**

  Use scopes instead.

  *Why?*: This is unfortunate, since it makes directives look different to services and controllers.
  However, the current `controllerAs` / `bindToController` syntax is too cumbersome when requiring
  other directives (see the `ngModel` examples below).


- **Complex link functions**

  Complex link functions are written similar to a controller, except you need to write `scope.` instead of `@`.

  Order everything as follows:
  - Private variables
  - Scope attributes
  - Initialization logic as an `init` function. This includes setting up watches, etc.
  - Scope methods
  - Private functions

  *Why?*: The `link` function should look similar to a controller.

  ```coffee
  # recommended
  @app.directive 'acmeSomeDirective', [->
    # ...

    link: (scope, element, attributes) ->

      # private variables

      somePrivateState =
        key: value

      # scope attributes

      scope.isVisible = false

      # initialization

      init = ->
        element.find('foobar').hide()

        scope.$watch 'something', somethingChanged

        scope.$on '$destroy', destroy

      # scope methods

      scope.doSomething = ->
        # ...


      somethingChanged = ->
        # ...

      # private methods

      destroy = ->

      init()
  ]
  ```


- **Prefix directives**

  Use a short, common prefix for directives, so you can easily distinguish your own directives from third party directives.

  ```coffee
  # recommended
  @app.directive 'acmeMultiSelect', [->
    # ...
  ]
  ```

- **Inline templates**

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

- **When it makes sense, try to keep directives generic**

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
    <acme-no-content content="orders" when-empty="No orders found.">
      <table>
        ...
      </table>
    </acme-no-content>
  ```

**[Back to top](#table-of-contents)**


## Promises

- **Always use promises instead of callbacks**

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


- **Prefer the closure form of $q when possible**

  *Why?*: Easier to understand.

  ```coffee
  # AVOID
  ->
    deferred = $q.defer()
    $timeout ->
      success = doSomething()
      if success
        deferred.resolve()
      else
        deferred.reject()

    deferred.promise
  ```

  instead:

  ```coffee
  # recommended
  ->
    $q (resolve, reject) ->
      $timeout ->
        success = doSomething()
        if success
          resolve()
        else
          reject()
  ```


## Custom form elements

- **Always depend on ngModel and use its API**

  If you write custom form elements, always extend `ngModel` and use Angular's default API.

  *Why?*: `ngModel` already solves a lot of issues you might not even be aware of.

  *Why?*: It makes other form features (like `ngChange`, validations, etc…) work out of the box.

  ```coffee
  # recommended
  @app.directive 'acmeCheckboxList', [->
    restrict: 'E'
    require: 'ngModel'

    template: """
              <div ng-repeat="option in options">
                <label class="checkbox-inline">
                  <input ng-change="checkboxChanged()" ng-model="checkboxes[option.id]" type="checkbox">
                  {{ option.label }}
                </label>
              </div>
              """

    link: (scope, element, attributes, ngModelController) ->

      scope.checkboxes = {}

      init = ->
        ngModelController.render = render

      scope.changed = ->
        ids = (id for id, value of scope.checkboxes when value)
        ngModelController.$setViewValue(ids)

      render = ->
        viewValue = ngModelController.$viewValue
        if viewValue?
          scope.checkboxes = {}
          for id in viewValue
            scope.checkboxes[id] = true

      init()
  ]
  ```


## Performance

- **One-time bindings**

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

- **ngIf vs ngShow / ngHide**

  Make a concious choice whether to use `ngIf` oder `ngShow`.

  `ngShow` requires little work when the condition changes, but needs to constantly update invisible content.

  `ngIf` has to update a lot of the DOM if the condition changes, but saves time rendering by not keeping invisible content up to date.
