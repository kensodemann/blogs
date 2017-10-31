# Unit Testing Stencil Components

We have added a unit testing framework to [Stencil](https://stenciljs.com). Stencil is a tool for building modern Web Components. Stencil combines some of the best features from traditional frameworks, but outputs 100% standards-compliant Custom Elements, part of the Web Component spec.

The goals we had when adding the testing framework were simple: make it easy to understand, and keep the API as small as possible. As a result of this, the testing API has a total of two methods: render() and flush().

render() - The render method takes an object containing a list of components that the Stencil compiler needs to know about and an HTML string. It returns the rendered element.
flush() - The flush method takes an element and causes it to rerender. This is typically done after properties on the element are changed.

## Getting Started

In order to get started with unit testing Stencil components, clone the [Stencil Component Starter](https://github.com/ionic-team/stencil-component-starter) and run `npm install`. To run the unit tests, use one of the following commands:

`npm test` - performs a single run of the unit tests
`npm run test.watch` - runs the unit tests and watches for changes to the source, rerunning the unit test with each saved change

In either case, [Jest](https://facebook.github.io/jest/) is used to run the unit tests.

## Application Setup

In order to perform unit tests on the components in the a project, some minimal setup is required in the project’s `package.json` file. The testing packages need to be installed as development dependencies, the npm scripts that run the tests need to be defined, and Jest needs to be configured to compile the sources and the tests properly.

The only packages required in order to run the tests are `jest` and `@types/jest`:

```
  "devDependencies": {
    ...
    "@types/jest": "^21.1.1",
    "jest": "^21.2.1"
  },
```

The two testing related npm scripts are `test` and `test.watch`.

```
  "scripts": {
    ...
    "test": "jest --no-cache",
    "test.watch": "jest --watch --no-cache"
  },
```

The jest setup section specifies the [moduleFileExtensions](http://facebook.github.io/jest/docs/en/configuration.html#modulefileextensions-array-string), [testRegex](http://facebook.github.io/jest/docs/en/configuration.html#testregex-string), and [transform](http://facebook.github.io/jest/docs/en/configuration.html#transform-object-string-string). The only part of the setup that is specific to Stencil is the `transform` section. Stencil provides a preprocessor script that uses the Stencil compiler to build the source and test files.

```json
  "jest": {
    "transform": {
      "^.+\\.(ts|tsx)$": "<rootDir>/node_modules/@stencil/core/testing/jest.preprocessor.js"
    },
    "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(tsx?|jsx?)$",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js",
      "json",
      "jsx"
    ]
  }
```

That is all of the setup that is required in order to run the unit tests. Since this has already been done in the [Stencil Component Starter](https://github.com/ionic-team/stencil-component-starter), if you are using that as a starting point for your project you should not need to do any extra setup.

## The Test File

The [Stencil Component Starter](https://github.com/ionic-team/stencil-component-starter) contains one test file: `src/components/my-name/my-name.spec.ts`. This file contains one test that just ensures the component can be built, and a group of tests that verify the rendering of the component.

### Testing the Component Class

The first test ensures that the component class can be built. 

```ts
 it('should build', () => {
   expect(new MyName()).toBeTruthy();
 });
```

If the component had methods defined on it, the same technique could be used to obtain an object to use to test those methods. For example, if `MyName` contained a method `format()` that returned a formatted name, you could test it as such:

```ts
 it('should build', () => {
   const myName = new MyName();
   myName.first = 'Peter'
   myName.last = 'Parker';
   expect(myName).toEqual('Parker, Peter');
 });
```

### Testing the Component Rendering

Tests can also render the component so the structure of the component can be tested. This is accomplished via two functions: `render()` and `flush()`. The render() function takes a configuration object consisting of a list of components and and HTML snippet, returning an HTML Element. The `flush()` function takes an element and applies changes to it. Both functions perform their tasks asynchronously.

Since both of these functions work with HTML elements, the standard HTML Element API is used to manipulate and query the structure of the element.

```ts
 describe('rendering', () => {
   let element;
   beforeEach(async () => {
     element = await render({
       components: [MyName],
       html: '<my-name></my-name>'
     });
   });

   ...

   it('should work with both a first and a list name', async () => {
     element.first = 'Peter'
     element.last = 'Parker';
     await flush(element);
     expect(element.textContent).toEqual('Hello, my name is Peter Parker (4)');
   });
 });
```

## Next Steps

In this article we explained the Stencil unit testing framework API, consisting of two functions: `render()` and `flush()`. We showed the additions required to the `package.json` file in order to allow unit testing. Finally we walked through the small sample test that is included in the [Stencil Component Starter](https://github.com/ionic-team/stencil-component-starter), and highlighted some of the functionality.

In a future post, I will walk through using this test along with some basic [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) in order to extend our component to meet our customer’s new requirements.


