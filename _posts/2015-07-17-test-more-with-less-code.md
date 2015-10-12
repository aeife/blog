---
layout: post
title: Test more with less code
comments: true
---

## How data providers can help you write better organized and well structured tests for your AngularJS application
Writing unit tests for AngularJS applications is pretty easy with the help of jasmine[^jasmine] and karma[^karma]. But thinking of every possible use case and combination of parameters can be tedious work which often leads to a lot of similar tests or lower test coverage (or maybe even both). This is where data providers come into play. They will not only help you write less code but also test more by having a wider variety of use cases covered. In this article I will show you how data providers can help you write better tests and why you should definitely use them.

Before I introduce you to data providers in jasmine I want to show you an example test without them. Lets imagine we have a simple `pokedex` service storing a list of Pokémon. We want to test a function that adds new Pokémon to the list. There are several use cases we need to cover here and the resulting tests could look something like this:

```js
// plain tests for multiple use cases

it('should add pokemon to list', function () {
  pokedex.list = ['Bulbasaur', 'Charmander'];
  pokedex.add('Squirtle');
  expect(pokedex.list).toEqual(['Bulbasaur', 'Charmander', 'Squirtle']);

  pokedex.list = [];
  pokedex.add('Squirtle');
  expect(pokedex.list).toEqual(['Squirtle']);
});

it('should not add pokemon to list if it already exists', function () {
  pokedex.list = ['Bulbasaur', 'Charmander'];
  pokedex.add('Charmander');
  expect(pokedex.list).toEqual(['Bulbasaur', 'Charmander']);
});

it('should not do anything (and hopefully not crash) if the parameter is null', function () {
  pokedex.list = ['Bulbasaur', 'Charmander'];
  pokedex.add(null);
  expect(pokedex.list).toEqual(['Bulbasaur', 'Charmander']);
});
```

As you can see the same test code gets repeated over and over to cover our various use cases. We could include all use cases in one test or write multiple tests for different use cases but in the end it would still remain repeated code. Some loops and structures might help but it still introduces additional logic to the test. Because we are using the same logic over and over and only changing data like the existing list and the new Pokémon to add this is a perfect example for using data provider.

Data providers, as the name suggests, are providing data for your tests. They can not only provide one data set but sequentially a whole list of data sets. The principle is that a test runs for each data set of the provider and can use the current data to define e.g. preconditions, parameters and the expectation. Data providers are available in unit tests for a lot of other programming languages but they are not included in jasmine and that’s why they are unfortunately rarely seen in AngularJS tests.

Using such a described data provider our example could look like this:

```js
// same test but using a data provider

var dataProvider = [{
  list: ['Bulbasaur', 'Charmander'],
  new: 'Squirtle',
  expected: ['Bulbasaur', 'Charmander', 'Squirtle']
}, {
  list: [],
  new: 'Squirtle',
  expected: ['Squirtle']
}, {
  list: ['Bulbasaur', 'Charmander'],
  new: 'Charmander',
  expected: ['Bulbasaur', 'Charmander']
}, {
  list: ['Bulbasaur', 'Charmander'],
  new: null,
  expected: ['Bulbasaur', 'Charmander']
}];

using(dataProvider, function (data) {
  it('should add pokemon to list', function () {
    pokedex.list = data.list;
    pokedex.add(data.new);
    expect(pokedex.list).toEqual(data.expected);
  });
});
```

So you can see that we now only have one single very short test. The rest is just the definition of our use cases in the data provider which itself is just a plain array. In order to actually use this data in our test we need a helper function which transforms our plain array to a real data provider for our test.
A very basic implementation of this data provider function can be achieved with only a few lines of code:

```js
// basic helper function for data providers

function using (values, func) {
    values.forEach(function(value) {
        if (!(value instanceof Array)) {
            value = [value];
        }

        func.apply(this, value);
    });
};
```

The `using` function just takes an array of data sets and a function. It then calls the function for every available data set, passing the current data set as parameter. So if we wrap this function around our test it means that our test will be executed for every data set in the array. Because of the passed parameter the test has access to the current data set and can use it.

> data providers will help you test more with less code

If you compare both examples (I provided a JSFiddle[^jsfiddle] to try it out and play around with), the one without data provider and the one using it, you can see that using data providers makes the actual test code way smaller. We now only have one test describing the core logic without any repetition and a separate data provider defining all the possible combinations and use cases. The result is a nice separation between test logic and use cases that need to be covered. When reading such code you actually don’t even need to look at the test itself. The use cases in the data provider describe exactly what should happen for which precondition and input. From my experience using data providers in my AngularJS projects I can tell you: Because it is so easy to add new use cases you are more likely to include more combinations and corner cases as you would have included when writing normal tests. So data providers will help you test more with less code.

Furthermore the usage of data providers allows us developers to structure our tests and make them more reusable. As an example we could introduce some general data providers that can be used across the whole test suite. We then can use them in our tests not only to define some parameters but also to initialize general data we might need over and over again, like a general model for example:

```js
// general data provider that delivers prefilled models

using(pokemonProvider, function (pokemon) {
  it('should do something...', function () {
    pokedex.categorize(pokemon);
    // ...
  });
});
```

As you can see the test can directly import a prefilled and maybe pre configured model without instantiating or setting anything up. Such a data provider can be used across all tests and will reduce repeating model definitions.

One last thing that is really cool about data providers is that they can be nested. So you can nest two or more data providers and it will work just as you would expect. For each data set of the outer provider every data set of the inner provider is applied, just like two nested loops. You could do the same work with only one provider if you like but this would mean that you need to define more use cases than necessary and your data providers will be less reusable.

```js
// nested data providers

using(resultProvider, function (data) {
  using(pokemonProvider, function (pokemon) {
    it('should return state according to the given result, no matter for what pokemon', function () {
      expect(pokedex.catch(pokemon, data.result)).toBe(data.expected);
    });
  })
});
```

As demonstrated data providers are an excellent way to write better tests for your AngularJS applications. Not only do they result in shorter test code and better readability, they also help you to really focus on the use cases to test. Once you start using data providers in your tests you will notice that you will add a lot more use cases to your tests because it’s that easy.

[^karma]: [karma test runner](http://karma-runner.github.io/)
[^jasmine]: [Jasmine: behavior-driven javascript](http://jasmine.github.io/)
[^jsfiddle]: [Example on JSFiddle](https://jsfiddle.net/E64Se/41/)
