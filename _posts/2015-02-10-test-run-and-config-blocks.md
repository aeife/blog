---
layout: post
title: Testing config and run blocks in AngularJS
comments: true
---

## How to easily test an often neglected part of your application
Unit tests are an important part of every AngularJS application. Using karma[^karma] and jasmine[^jasmine] you have all the tools needed for reaching a good test coverage of your source code. But often even projects with a lot of unit tests omit any tests concerning those `run` and `config` blocks of angular modules[^angularjs modules documentation] although these are crucial parts of an application.

Many developers don't test these parts of an application because they just don't know how or don't want to bother finding it out. It can be quite hard to find a comprehensive instruction how to actually do testing for this parts of your application. In the following article I will explain workflows for painlessly testing both your `run` and your `config` blocks.

## Creating a use case
To better illustrate how to test `config` and `run` blocks in AngularJS we first create an actual example module which we want to test. The module will contain a `config` block which just sets the `html5mode` via `$locationProvider` [^angularjs locationprovider documentation]. The `run` block will contain a simple function call of a service, specific calling a `start` function of an arbitrary logging service. The content of this service does not matter, its only purpose is to provide an exemplary use case to test. 

```js
angular.module('example', [])
	.config(function ($locationProvider) {
		$locationProvider.html5Mode(true);
	})
	.run(function (loggingService) {
		loggingService.start();
	});
```

## Testing the config block
Let´s start by testing the configuration block. According to our example we want to test if the function `html5mode` of the `$locationProvider` is called with the correct parameter.

The difficult part is to know how to spy on your provider so you can check for such function calls. Just loading the module and using the `inject` function as it is common when writing unit tests will not work for this use case. The solution is to write a new dummy module including a separate configuration.

```js
describe('example', function () {
	var $locationProvider;

	beforeEach(function () {
		angular.module('locationProviderConfig', [])
            .config(function(_$locationProvider_) {
                $locationProvider= _$locationProvider_;

                spyOn($locationProvider, 'html5Mode');
            });
        module('locationProviderConfig');

		module('example');

		inject()
	});

	it('should set html5 mode', function() {
       expect($locationProvider.html5Mode).toHaveBeenCalledWith(true);
    });
});
```

We defined a complete new angular module in our `beforeEach` function. This module has a `config` block just like the module we want to test. This allows us to inject the same provider used in our example module and to create a spy on this provider, specific on the function we want to test later.

To ensure that this code is actually called we load the dummy module and next we load our actual module `example`. Because our module to test is loaded after our dummy module the spy was already created. When we now load in our actual module the `config` function of this module will be executed and injects the provider which includes the spy we defined.  Notice that we still need to call `inject` to actually load and run the modules correctly.

There also is another way to get references to providers by using the `module` function. This solution differs from the way you would use a provider in your actual application but is a bit shorter and preferred by a lot of developers.

```js
beforeEach(function () {
    module(function(_$locationProvider_) {
        $locationProvider = _$locationProvider_;
        spyOn($locationProvider, 'html5Mode');
    });
    module('example');
    inject();
});
```

Because we saved the provider into an own variable we can use it in our test and check if the spy has been called. This workflow allows us to have a reference to the complete provider used in the `config` block and to define every spy or property on it that we want. Using this you should be able to test every code in your `config` block.

## Testing the run block
Testing `run` blocks of your application can be similarly confusing but requires a completely different approach.

In comparison to the workflow described for the `config` block you won't need any dummy modules. The difficulty here is that modules in tests need to be loaded before you call the `inject` function. To test if a function has been called in the `run` block you would need to spy on the service before it is loaded but the usual way of spying on services is to inject them. So as you can see it sounds like a chicken and egg problem.

But there is actually a way you can spy on a service without injecting it in your test using the `$provide` service.

```js
describe('example', function () {
	var loggingService;

	beforeEach(function () {
		module('example', function ($provide) {
			$provide.value('loggingService', {
				start: jasmine.createSpy()
			});
		});

		inject(function (_loggingService_) {
			loggingService = _loggingService_;
		});
	});

	it('should start logging service', function() {
	    expect(loggingService.start).toHaveBeenCalled();
    });
});
```

In the code example you can see that we load our module and an anonymous module initialization function. Here we inject `$provide` and define a new value. The trick is to define a value with the same name as the service you want to spy on. This will overwrite your actual service and inject your defined mock object instead. Because we want to check if the `start` function is called we define an object containing a `start` property as a spy. After that we use the `inject` function as usual and save a reference to the service.

In the test itself we can use this variable to check the spy we defined. Using this approach we again are able to test all parts in our `run` block.

## Conclusion
Testing `config` and `run` blocks in AngularJS actually is not that hard once you figured out the ways to do so.  Using the workflows described you should be able to test every content of your applications initiation logic leaving no excuses to skip them when writing testing.

You can view the full source code used in this article as a working example [here](http://jsbin.com/mejulakinu/edit?js,output) [^jsbin example].

[^karma]: [karma test runner](http://karma-runner.github.io/)

[^jasmine]: [Jasmine: behavior-driven javascript](http://jasmine.github.io/)

[^angularjs modules documentation]: [modules documentation](https://docs.angularjs.org/guide/module)

[^angularjs locationprovider documentation]: [$locationProvider documentation](https://docs.angularjs.org/api/ng/provider/$locationProvider)

[^jsbin example]: [jsbin example](http://jsbin.com/mejulakinu/edit?js,output)
