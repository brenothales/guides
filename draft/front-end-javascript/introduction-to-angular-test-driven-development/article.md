If you have ever taken the step to build something more complex than a simple to-do app with a AngularJS or any other technology, you know that you always end up fixing old things along with the implementation new features. This is a common problem with many technolgies. No matter the language your software is  built with, its architecture and ways of separating concerns, you will sooner or later end up breaking things that previously worked when releasing new builds. This is where [Test-Driven Development](http://hackguides.org) comes in play. 

Unit testing helps you answer all the *'Is this going to work if I insert X?'* or *Does X still return the same results now that I have implemented Y?'* scnearios. It does it by simulating actions with mock data and checking if everything is outputted in the format you expect it to be.

This guide is a great starting point for your journey in testing Angular applications. In order to get the most out of the guide, you need to have some knowledge of JavaScipt and building Angular applications.


Angular is built with [testing in mind](https://docs.angularjs.org/guide/unit-testing). If you are new to writing unit tests, Angular is a great starting point to get deeper into the topic. The framework allows simulation of server-side requests and abstraction of the [document object model](https://en.wikipedia.org/wiki/Document_Object_Model) , thus providing an environment for testing out numerous scenarios. Additionally, Angular's dependency injection allows every component to be mocked and tested in different scopes.


 We'll start off by setting up the environment and looking at how [Jasmine](http://jasmine.github.io/2.4/introduction.html), [Karma](https://karma-runner.github.io/1.0/index.html) and [Angular Mocks](https://docs.angularjs.org/api/ngMock) work together to provide an easy and seamless testing experience in Angular. Then, we will have a look at the different building blocks of tests. Lastly, we will use the newly acquired knowledge to build sample tests for controllers, services, directives, filters, routes and promises.


## Getting started

Every great developer knows what his tools are for. Knowing the role of each of your tools for testing is essential before diving right into writing tests. 

### What does what?

** Karma**  an environment which runs the tests of your applicaiton. Simply put, it's your testing server. Originally started as a [university thesis](https://github.com/karma-runner/karma/raw/master/thesis.pdf), Karma aims to make a framework agnostic environment that automates the running of your unit tests. In its core, it is a [Node](https://nodejs.org/en/) server that watches for changes in your testing and application files, and when such changes occur, it runs them into a browser and checks for mistakes.

** Jasmine ** is an unit testing framework for JavaScript. It is the most popular framework for testing JavaScript applications, mostly because it is quite simple to start with and flexible enough to cover a wide range of scnearios

** Angular mocks ** is an Angular module that is used to mock compontents that already exist in the applicaiton. Its role is to inject various components of your Angular applicaiton (controllers, services, factories, directives, filters) and make them available for unit tests. It can be said that Angular Mocks is the middleman between the Angular components in your applicaiton and the unit testing environment.

### Setting up the testing environment

 To ensure a speedy and efficient setup, I recommend using the [Node Package Manager (npm)](https://www.npmjs.com/) to maintain the dependencies of your Angular project. 
 
 First, let's install the packages we'll need to run the tests globally (done by appending <code> -g </code>) . 
Open your terminal and run the following commands:
 ```
  npm install -g karma
  npm install -g jasmine-core 
```
Then, create a directory where you'll store your project files.
 
 ```bash
 mkdir myitemsapp
 cd myitemsapp
 ```
 #### Installing packages
 
 Once you are int he directory, start setting up your project dependencies. 
 First, initialize your <code>package.json</code> file:

```bash
 npm init
 ```
 You will get asked several questions regarding your project's details. You can skip them and simply copy the contents below in your <code> package.json </code> file.
 
```json
 {
  "name": "myitemsapp",
  "version": "1.0.0",
  "description": "",
  "main": "karma.conf.js",
  "directories": {
    "test": "tests"
  },
  "dependencies": {
    "angular": "^1.5.7",
    "save": "^2.3.0"
  },
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```
 Then start installing your project's dependencies, starting off with Angular:
```bash
npm install angular --save
npm install karma --save-dev
npm install karma-jasmine jasmine-core --save-dev
npm install angular-mocks --save-dev

```
Next, you have to choose a browser launcher for Karma to use:
You can use one for [Chrome](npm install karma-chrome-launcher --save-dev) , [Firefox](https://github.com/karma-runner/karma-firefox-launcher) , [Internet Explorer](https://github.com/karma-runner/karma-ie-launcher), [Opera](https://github.com/karma-runner/karma-opera-launcher), [PhantomJS](https://github.com/karma-runner/karma-phantomjs-launcher) and others.

 I will go with Chrome:
 
```
npm install karma-chrome-launcher --save-dev
```

#### Configuring Karma
 Configuring your testing environment is done through a configuration file (<code>karma.conf.js</code>), similarly to configuring <code> package.json </code>, which we used to configure the project environment.
 
There is a [great number of options](http://karma-runner.github.io/1.0/config/configuration-file.html) for configuring Karma. The most essential are the following:

- **Frameworks** - the testing frameworks that are going to be used. In this guide, we are using jasmine.
- **Files** - files that will be used for the tests. You would normally include both the framework files (in this case, Angular) as well as the project and the testing files themselves.
- **Browsers** - You can specify which browsers must Karma use in order to run its tests.

Before starting, create two folders in  your working directory.

```
mkdir app 
mkdir tests 
```
We'll use <code> app </code> to store our applicaton code and <code>tests</code> to keep our tests. This way, we'll keep the two separated. In more advanced cases you would opt for a more generic approach by using wildcards and storing the test and application files together. 

With your terminal in the directory of the project, run:

```bash
karma init
```
Answer the questsions as following:
- Select Jasmine as your testing framework.
- Select 'Chrome' as your browser.
- Specify the paths to your js and spec files: <code> 'app/\**.js', 'test/*\*.js' </code>
- Specify the path to your Angular and ngMocks files: <code>'node_modules/angular/angular.js','node_modules/angular-mocks/angular-mocks.js'</code>


In the end, you should end up with a <code>karma.conf.js</code> file looking like this: 
```javascript
module.exports = function(config) {
  config.set({
    // base path that will be used to resolve all patterns (eg. files, exclude)
    basePath: '',
    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['jasmine'],
    // list of files / patterns to load in the browser
    files: [
      'node_modules/angular/angular.js',
      'node_modules/angular-mocks/angular-mocks.js',
      'app/app.js',  //use wildcards in real apps
      'tests/tests.js' //use wildcards in real apps
    ],
    // list of files to exclude
    exclude: [
    ],
    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
    },
    // test results reporter to use
    // possible values: 'dots', 'progress'
    // available reporters: https://npmjs.org/browse/keyword/karma-reporter
    reporters: ['progress'],
    // web server port
    port: 9876,
    // enable / disable colors in the output (reporters and logs)
    colors: true,
    // level of logging
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_INFO,
    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: true,
    // start these browsers
    // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
    browsers: ['Chrome'],
    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: false
  })
}
```
Once you are done with this step, you are ready to dive into testing features.

#### Running your tests

 To run your tests, you can run any of these two commands in the terminal:
 
```
karma start
npm test
```

## Building blocks

Writing tests revolves around events and their expected outcomes. When you start writing the tests for your applicaiton, you must always think in terms of desired outcomes your app components wants to produce. 

Although the concepts that are going to be used are specific for Jasmine, they share many similarities with other behaviour-driven testing frameworks:

- **Suites** - Are the top-level element of the testing framework. They accept a title (explanation) and a function containing one or more specifications. 

<code>describe(string, function)</code>
- **Specs** -  Are constructs that take a title and a function containing one or more expectations. Specs are nested into suites.

<code>it(string, function)</code>
- **Expectations** — are assertions that evaluate to true or false.

<code>expect(actual).toBe(expected)</code> 
- **Matchers** - Are redefined helpers for common assertions. They are the constructs that do the evaluations and decide whether a test has failed or not.
- 
<code>toEqual(expected)</code>

To give you a better idea of what matchers can do, here is a list of the most essential matchers:

```javascript
expect(fn).toThrow(e);
expect(instance).toBe(instance);
expect(mixed).toBeDefined();
expect(mixed).toBeFalsy();
expect(number).toBeGreaterThan(number);
expect(number).toBeLessThan(number);
expect(mixed).toBeNull();
expect(mixed).toBeTruthy();
expect(mixed).toBeUndefined();
expect(array).toContain(member);
expect(string).toContain(substring);
expect(mixed).toEqual(mixed);
expect(mixed).toMatch(pattern);

```
[Here is the full list of matchers.](https://github.com/JamieMason/Jasmine-Matchers)

#### Teardown

Teardowns are another building blocks of tests that is used to "prepare" the code for its specs in a particular suite. Suppose that before or after each spec (i.e a <code>describe</code> function) to do certain setups so that you can test a particular functionality. Instead of doing it for every spec, you can write a <code> beforeEach </code> or an <code> afterEach </code> function in your sute to do that.

```javascript
// single line
beforeEach(module('itemsApp'));

// multiple lines
beforeEach(function(){
  module('itemsApp');
  //...
});
```
 In the block above you can see two ways in which you can initiallize the Angular module in a testing suite using teardown.
 
 #### Injecting
 
 Injecting is essential feature of unit testing Angular applicaitons. It is characterised by the <code> inject </code> function that is provided by the Angular mocks module. Injecting can be regarded as a way of accessing Angular's built-in constructs or your applicaiton's constructs by giving the correct arguments. This gives you the ability to access the code of your application and put mock data through it in order to test it.
 
 Injecting gives access to  Angular's building blocks - *$service*, *$controller*, *$filter* , *$directive*, *$factory*. Additionally, it allows to mock built-in variables such as *$rootScope* and *$q*. It also provides *$httpBackend* to simulate server-side requests in the testing envirionemnt.
 
 
```javascript
 
 // Using _serviceProvider_ notation
var $q;
beforeEach(inject(function (_$q_) {
    $q = _$q_;
}));

// Using $injector
var $q;
beforeEach(inject(function ($injector) {
    $q = $injector.get('$q');
}));

// Using an alias Eg: $$q, q, _q
var $$q;
beforeEach(inject(function ($q) {
    $$q = $q;
}));
 
 
```

In the code snippet above, you can see three ways of injecting and instantiating the Angular [$q](https://docs.angularjs.org/api/ng/service/$q) variable, which is used for asynchronous requests, in the testing environment. 

## Writing your first tests

Time to put all that knowledge to use. Let's do go through some scenarios you might encounter when writing your tests:

### Testing a controller
First, let's instantiate an Angular applicaiton in <code> app.js </code>

**Code**
```javascript
// app/app.js

angular.module('ItemsApp', [])
```

Then, write a simple controller:


```javascript
//app/app.js
angular.module('ItemsApp', [])
  .controller('MainCtrl', function($scope) {
      $scope.title = 'Hello Pluralsight';
    })
}])
```
We have a controller with one simple $scope variable attached to it. Let's write a test to see if the scope variable contains the value we expect it to have:

**Test**
```javascript
//tests/tests.js
// Suite
describe('Testing a Hello Pluralsight controller', function() {

});
```
Let's go thorugh building the code step-by-step. First, we use the <code> describe </code> function to make a new testing suite for the controller. It contains a message as its first argument and a function that is going to contain the tests in the  second argument
Next, let's inject the controller in the suite:


```javascript
//tests/tests.js

describe('Testing a Hello Pluralsight controller', function() {
  var $controller;

  // Setup for all tests
  beforeEach(function(){
    // loads the app module
    module('ItemsApp');
    inject(function(_$controller_){
      // inject removes the underscores and finds the $controller Provider
      $controller = _$controller_;
    });
  });

});

```
Here, using <code>beforeEach </code>, we define our teardown flow. Before each of the tests, we are going to inject the controller provider from Angular and make it available for testing.
Next, we'll move to the gist  - the tests themselves:

```javascript
//tests/tests.js

// Suite
describe('Testing a Hello Pluralsight controller', function() {
  var $controller;

  // Setup for all tests
  beforeEach(function(){
    // loads the app module
    module('ItemsApp');
    inject(function(_$controller_){
      // inject removes the underscores and finds the $controller Provider
      $controller = _$controller_;
    });
  });

  // Test (spec)
  it('should say \'Hello Pluralsight\'', function() {
    var $scope = {};
    // $controller takes an object containing a reference to the $scope
    var controller = $controller('MainCtrl', { $scope: $scope });
    // the assertion checks the expected result
    expect($scope.title).toEqual('Hello Pluralsight');
  });

  // ... Other tests here ...
});
```
We start our first test specification (spec) with the <code> it </code> function. The first argument is a message, an hte second argument is the function wit the test. First we instantiate <code> MainCtrl </code> that we created in the application. Then, we use a matcher to check its <code> $scope.title </code> variable if it is equal to the value we assigned in the application.

### Testing a service

Next, we are going to add a service to our application in order to test it. We'll write a service with one method, <code>get() </code> that returns an array. Just below your controller code, add the following snippet:

**Code**
```
//app/app.js

angular.module('ItemsApp', [])
//..
// MainController
//..
.factory('ItemsService', function(){
  var is = {}, 
    _items = ['hat', 'book', 'pen'];
  
  is.get = function() {
    return _items;
  }
  
  return is;
})


```

Here is how we are going to test it:

**Test**
```
describe('Testing Languages Service', function(){
  var LanguagesService;
  
  beforeEach(function(){
    module('ItemsApp');
    inject(function($injector){
      ItemsService = $injector.get('ItemsService');
    });
  });
  
  it('should return all items', function() {
    var items = ItemsService.get();
    expect(items).toContain('hat');
    expect(items).toContain('book');
    expect(items).toContain('pen');
    expect(items.length).toEqual(3);
  });
});

```
Even though we have only one spec, we stick to the good practice to using <code>beforeEach</code> in our suite in order to instantiate what we need.

In the spec, we use the <code>toContain</code> matcher to check the contents of the array that the <code>get() </code> method returns. In the end, we use <code>languages.length</code> with the <code> toEqual </code> matcher to check the length of the array.

### Testing directives

Directives differ in terms of purpose and structure than services and controllers, and they are tested using different approach. Directives have their own encapsulated scope which gets its data from an outer scope, a controller, for example. What happens to directives is that they first get "compiled" (in Angular.js sense)  and then their scope gets filled with data. In order to properly test them, we must simulate the same process and see if we get the desirable outcomes.

We are going to add a simple directive that will display the user's profile. It will get its profile data from outside and apply it into its scope.


**Code**
```javascript
//app/app.js
angular.module('ItemsApp', [])
//rest of the app

.directive('userProfile', function(){
  return {
    restrict: 'E',
    template: '<div>{{user.name}}</div>',
    scope: {
        user: '=data'
    },
    replace: true
  };  
})

```

**Test**

```javascript
//tests/tests.js
describe('Testing user-profile directive', function() {
  var $rootScope, $compile, element, scope;
  
  beforeEach(function(){
    module('ItemsApp');
    inject(function($injector){
      $rootScope = $injector.get('$rootScope');
      $compile = $injector.get('$compile');
      element = angular.element('<user-profile data="user"></user-profile>');
      scope = $rootScope.$new();
      // wrap scope changes using $apply
      scope.$apply(function(){
        scope.user = { name: "John" };
        $compile(element)(scope);
      });
    });
  });

  it('Name should be rendered', function() {
    expect(element[0].innerText).toEqual('John');
  });
});


```

The code might be confusing at first, but if you have some basic idea how Angular works, it's pretty straightforward: 
1. You create a scope to which you apply the directive. First, you get the <code>$rootScope</code> and create a new one using <code> $rootScope.$new() </code>. 
2. Once you have a scope, you create a new DOM <code>element</code> using <code>angular.element() </code>.
3. You attach the element to the DOM using [$compile](https://docs.angularjs.org/api/ng/service/$compile). <code> $compile </code> will get the element and apply a scope to it. Essentially, it's the function that parses the HTML element, finds the directive it corresponds to, and attaches its template and scope into the outer scope.
4. We have to simulate an action of putting data in a scope, but there is no browser and no DOM. In such cases, we use [$apply](https://docs.angularjs.org/api/ng/type/$rootScope.Scope). It is used to add a variable to the scope (<code> {name: 'John'} </code>) to a scope.


When you're done, the <code>element</code> variable will be filled with HTML generated by the directive in your code. All you need to do is use a spec to test if it returned the result you wanted.

### Testing filters

## Testing patterns and directory structure


**From the tests we just wrote, we can see a clear pattern emerging:**

1. Describe the spec with a type and name
2. Load the object's module.
3. Load mock modules if needed.
4. Inject dependencies and spy on methods.
5.Initialize the object:
 5.1.Services just need to get injected.
 5.2.Controllers are instantiated using the <code>$controller</code> service.
 5.3 We need to <code>$compile</code> directives.
6.Write expectations grouped in describe blocks.

This guide featured only two files being tested - <code>app.js</code> for the code and <code>tests.js</code> for the tests. However, In larger and more complex projects, you have to opt for a different file structure:
sd