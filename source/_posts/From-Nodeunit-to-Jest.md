---
title: From Nodeunit to Jest
tags:
  - Nodeunit
  - Jest
  - Testing
  - NodeJS
  - JavaScript
subtitle: The Post-Nodeunit Jest Survival Guide
date: 2017-08-25 18:30:49
---


Have you been using a tried-and-true testing framework for years and wondered whether it might ever be worth it to migrate over to one of the scrappy newcomers? Maybe you wished someone would write up a practical comparison or put together a simple how-to-guide to ease the transition? This was exactly the situation in which we found ourselves on a team that had long used Nodeunit. We decided to take the jump and switched a newer project over to [Jest](https://facebook.github.io/jest/https://facebook.github.io/jest/). What follows is a practical guide born out of that experience. It was originally written as documentation to help developers on that team make the transition as smoothly as possible. As such, it will be particularly helpful if you are migrating or have migrated from Nodeunit to Jest. Regardless, there's a lot to glean for anyone considering Jest or using it in any capacity.

## Wait. How do I...

So here you are, your team has switched from Nodeunit to Jest and everything is different and frustrating. Have no fear! Jest has particularly helpful [guides](https://facebook.github.io/jest/docs/getting-started.html) and [documentation](https://facebook.github.io/jest/docs/api.html) and here I've put together a how-to-guide addressing the specific hiccups you are most likely to encounter. Each of the following points answers the question: "Wait. How do I ____?".

### Add a new test

The first thing you'll notice when you start working with Jest is that the structure and syntax is very different. This is the easy part, though. Jest's syntax is very natural and intuitive. Here's how you would add a new test, first in Nodeunit as a familiar starting point for comparison and then in Jest.

#### In Nodeunit

In Nodeunit, we used `module.exports` to define test suites. Starting with a top level index.js file, we'd kick off a chain of requires and exports that would make up our test suite. Adding a new test in Nodeunit looked like adding a method to an exported object within that chain.

```JavaScript
module.exports = {
  somethingHappens(test) {
    // test goes here.
  }
}
```

#### In Jest

Jest, on the other hand, uses path matching to automatically require and test each file individually. By default, all the `*.js(x)` files inside the `__tests__` directory and `*.test.js(x)` or `*.spec.js(x)` files anywhere in the repository are tested. So, to create a new test, all that is required is adding a file matching those patterns (example: `__tests__/testsAreHere.js`) which contains at least one test.

```JavaScript
test('Something happens when I expect it to', () => {
  // test goes here.
});
```

### Group tests together

Grouping tests is helpful for organization, selective test runs, and consolidating setup and teardown operations. This is one of the largest practical differences between Nodeunit and Jest.

#### In Nodeunit

In Nodeunit, tests were grouped together simply by exporting them within the same parent object.

```JavaScript
module.exports = {
  setUp() {
    this.contextualVariable = 'This is available in all tests contained within this object';
  },
  tearDown() {
    delete this.contextualVariable;
  },
  oneTest(test) {
    // test goes here.
  },
  twoTest(test) {
    // test goes here.
  },
};
```

#### In Jest

Jest groups tests together in two ways. The first is by simply including them in the same file together. The second is by using the `describe()` function.

```JavaScript
const context = {};

beforeEach(() => {
  context.variable = 'This is available in every test wherever the "context" variable is in scope.';
});

afterEach(() => {
  delete context.variable;
});

describe('A group of tests within a describe statement', () => {
  beforeEach(() => {
    context.describeVar = 'Only available in tests within this describe block';
  });

  afterEach(() => {
    delete context.describeVar;
  });

  test('can access "context.variable"', () => {
    // test goes here.
  });

  test('can access "context.describeVar"', () => {
    // test goes here.
  });
});

test('I can access "context.variable" but not "context.describeVar"', () => {
  // test goes here.
});
```

### Share setup and teardown between multiple files

Here's where things start to get dicey. Good thing you have this trusty guide!

#### In Nodeunit

In Nodeunit, this was easy because a `setUp()` or `tearDown()` in an exported object would be inherited by all tests contained within the same object, regardless of whether they were in different files.

```JavaScript
modules.exports = {
  setUp() {
    this.context = 'stuff',
  },
  tearDown() {
    delete this.context;
  },
  // All tests contained within someOtherFile.js will inherit the above setUp
  // and tearDown methods.
  descendentTests: require('./someOtherFile'),
};
```

#### In Jest

In Jest, this one's a bit trickier. You can't rely on a hierarchical `module.exports` for setup and teardown inheritance, because each file is tested individually in isolation from the rest. But you're in luck! There are alternative methods. Here are a couple:

##### Manage context in a separate file

One potential method for sharing setup and teardown between files is by managing context and methods like beforeEach and afterEach in a separate file. Then you can require that file into your tests wherever it is relevant.

```JavaScript __tests__/__helpers__/testContext.js
const context = {};

module.exports = {
  beforeEach() {
    context.variable = 'set up variable before each test';
  },
  afterEach() {
    // clean up!
    delete context.variable;
  },
  getContext() {
    return context;
  },
};
```

You'll want to make sure you ignore the above file by utilizing the [testPathIgnorePatterns](https://facebook.github.io/jest/docs/configuration.html#testpathignorepatterns-array-string) configuration option with a value like: `["<rootDir>/__tests__/__helpers__/*.js"]`

```JavaScript __tests__/someTest.js
const testContext = require('./__helpers__/testContext');

beforeEach(testContext.beforeEach);
afterEach(testContext.afterEach);

test('I can make use of context', () => {
  const context = testContext.getContext();
});
```

##### Utilize the setupFiles configuration option

Jest provides a [setupFiles](https://facebook.github.io/jest/docs/configuration.html#setupfiles-array) configuration option which allows you to define files which will be run before each test. This is a great option for when there are general setup or teardown tasks which should apply to _all_ tests.

### Mock like a Sinon

[Sinon](http://sinonjs.org/) is an excellent and robust tool for mocking, stubbing, and spying in your tests. I've found, however, that I no longer need it since Jest provides very similar functionality. Here's a comparison of a simple stub implementation in both Nodeunit/Sinon and Jest.

#### In Nodeunit/Sinon

You'll notice that I'm recreating the stub before and fully restoring it after each test in the below example unlike in the Jest example below where I simply reset it before each test. This is because Nodeunit doesn't have an equivalent to Jest's `afterAll()`. So I need to restore it after each test to prevent any subsequent test from getting a stubbed version of `toMock.thing()` when it may expect the real deal.

```JavaScript
const sinon = require('sinon');
const toMock = require('something-i-want-to-mock');


module.exports = {
  setUp() {
    sinon.stub(toMock, 'thing');
  },
  tearDown() {
    toMock.thing.restore();
  },
  aTest(test) {
    toMock.thing.returns(42);
    test.equals(toMock.thing(), 42);
  },
};
```

#### In Jest

```JavaScript
const toMock = require('something-i-want-to-mock');
jest.spyOn(toMock, 'thing');

afterEach(() => toMock.thing.mockReset());

afterAll(() => toMock.thing.mockRestore());

test('When I mock a thing it returns 42', () => {
  toMock.thing.mockReturnValue(42);
  expect(toMock.thing()).toBe(42);
});
```

### Subvert requires like a Require Subvert

[Require Subvert](https://github.com/rvagg/node-require-subvert) is handy when you need to stub a function that a module exports directly (which Sinon cannot do), rather than as a method on an object.

#### In Nodeunit/Sinon/Require Subvert

Here it becomes even more difficult to clean up after stubbing. Require Subvert provides a helpful `cleanUp()` method, but there's no easy way to cleanup after all tests are done without subverting and cleaning up after each test. That starts to get messy since it requires re-creating the stub, re-subverting the thing you want to mock and re-requiring the module that uses the mocked thing before every single test. So in most cases we just left the subverted require subverted and hoped for the best.

```JavaScript
const sinon = require('sinon');
const requireSubvert = require('require-subvert')(__dirname);
const stub = sinon.stub().returns(42);
requireSubvert.subvert('something-i-want-to-mock', stub);
const usesToMock = requireSubvert.require('something-that-uses-the-mocked-thing');

module.exports = {
  aTest(test) {
    test.equals(usesToMock.callToMock(), 42);
  }
};
```

#### In Jest

Using Jest, this complicated operation becomes very simple.

```JavaScript
jest.mock('something-i-want-to-mock', () => 42);
const toMock = require('something-i-want-to-mock');
const usesToMock = require('something-that-uses-the-mocked-thing')

afterAll(() => toMock.mockRestore());

test('When I mock a thing it returns 42', () => {
  expect(usesToMock.callToMock()).toBe(42);
});
```

### Report coverage like an Istanbul

Jest can also handle your coverage reporting similarly to [Istanbul](https://github.com/gotwarlost/istanbul). Simply add `--coverage` to your test script in your package.json file.

```json package.json
{
  "scripts": {
    "test": "jest --coverage"
  }
}
```

### Run specific tests or groups of tests

Jest provides a number of methods for selectively running specific tests. Here are a few of them:

#### Specify the path to files or directories

```bash
# A single file.
yarn test -- __tests__/someTestFile.js

# All tests in a directory.
yarn test -- __tests__/lib

# All tests matching a glob pattern.
yarn test -- __tests__/**/functional
```

#### Specify a test name pattern

Assuming you have the following test:

```JavaScript
test('can do stuff', () => {
  // test things in here.
});
```

Then you can run just that test (and any other test with the same name) like this:

```bash
yarn test -- -t "can do stuff"
```

You can also use regex patterns to run multiple tests with names that match the pattern.

#### Specify a test path pattern

Given you have the following tests:

```
__tests__/libOne/will-run.js
__tests__/libOne/will-also-run.js
__tests__/libTwo/me-too.js
__tests__/helpers/wont-run.js
```

The following command will run all the tests inside `__tests__/libOne` and `__tests__/libTwo`:

```bash
yarn test -- --testPathPattern "__tests__/lib\(One\|Two\)"
```

## Okay. What more can it do?

Well, if you've made it this far, you've survived! Congratulations! We've already seen how Jest provides improved syntax and functionality, but let's get out of survival mode and look at how Jest goes beyond simply replacing our old tooling and improves our testing workflow in new ways.

### Generate and compare data expectations with snapshots

On our particular project, we were sinking hours into managing exceptionally complicated JSON files which represented expected output. A relatively simple code change could set off a ripple of small required changes dispersed throughout a file, each needing to be tediously manually entered. Then we discovered Snapshot testing.

[Snapshot testing](https://facebook.github.io/jest/docs/snapshot-testing.html) allows you to easily manage and compare changes to expected output. Usually it's used to ensure a user interface doesn't change unexpectedly, but it's not limited to markup. You can create snapshots of any serializable value. As I mentioned above, in our case, we used it to manage complicated JSON object output expectations. Using snapshots, Jest creates the expected objects automatically the first time the test is run. Every time the test is run after that, Jest will check to see if the newly generated snapshot matches the previously created one. If not, the test will fail and a helpful diff will be reported to the console. If the change is expected, simply run the test again with the `--updateSnapshot` or `-u` flag to update the snapshot.

To start using snapshots, simply use `expect(someValue).toMatchSnapshot()` in your test. A snapshot will be created against `someValue`.

### Only run tests relevant to changed files

If you're working on a project with a large or time-consuming test suite, you may find the `--onlyChanged` or `-o` flag useful. When run with this flag, Jest will identify tests which should be run based on which files have been changed in your current git working tree. This allows you to quickly test for regressions based on changes you've made locally.

### Run tests automatically when files change

Use `--watch` to watch code for changes and rerun tests when a change is detected. Running Jest in watch mode also provides a convenient method for running all tests, selective tests, or re-running tests. Here's the prompt you'll see when you start watching:

```
Watch Usage
 › Press a to run all tests.
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press q to quit watch mode.
 › Press Enter to trigger a test run.
 ```

## What's the catch? (Gotchas)

### It's simply not as fast

A [brief](https://facebook.github.io/jest/blog/2016/03/11/javascript-unit-testing-performance.html) google [search](https://github.com/facebook/jest/issues/2171) will [reveal](https://github.com/facebook/jest/issues/116) that [performance](https://github.com/facebook/jest/issues/857) has [historically](https://github.com/este/este/issues/180) been an [issue](https://groups.google.com/forum/#!topic/jestjs/iy_p8sUxasw) for Jest. After we migrated to Jest, we found it generally took about twice as long as the same suite running on Nodeunit. It's a legitimate trade-off. For some, the extended feature-set and improved syntax Jest offers is worth the hit in performance. For others, especially those with already time-consuming test suites, it may not be. There are, however, some ways to mitigate the performance hit you'll likely see when switching to Jest. Here are some recommendations:

#### Use the proper test environment

The default test environment is jsdom, but if your project does not require a browser-like environment, you'll see a significant increase in performance if you switch the test environment to "node". You can do this by configuring jest in your package.json.

```json package.json
{
  "jest": {
    "testEnvironment": "node"
  }
}
```

#### Configure testMatch to be as specific as possible

By default, Jest looks for `.js` and `.jsx` files in the `__tests__` directory as well as any `.spec.js(x)` or `.test.js(x)` files anywhere in your codebase. You can speed up startup time significantly by putting all your tests in a single directory and limiting the path matching to that directory. In our case, we knew we only wanted to test JavaScript files within the `__tests__` directory. So we set `testMatch` to contain a single value.

```json package.json
{
  "jest": {
    "testMatch": [
      "<rootDir>/__tests__/**/*.js"
    ]
  }
}
```

#### Use caching in CI builds

Jest stores a cache in the `/tmp` directory which, once created, significantly speeds up subsequent runs. In CI builds without proper configuration, however, the cache directory will usually get wiped out with every build. To resolve this in our case, we moved the cache directory to `.tmp/jest_cache` at the root of our project and configured Travis to preserve that directory in its own cache between builds.

```json package.json
{
  "jest": {
    "cacheDirectory": ".tmp/jest_cache"
  }
}
```

```yml .travis.yml
cache:
  directories:
    - '.tmp'    
```

### Occasional illegitimate errors within CI builds

In our experience, tests within CI builds occasionally fail with random SyntaxError exceptions like `SyntaxError: missing ) after argument list` or `SyntaxError: unexpected token }`. It appears this is caused by a race condition where Jest attempts to write a transformed file to cache from multiple child processes at the same time. If you run into this bug and simply restarting the build is not a reasonable option, the workaround for now is to run Jest with either the `--no-cache` or the `--runInBand` flag. There _is_, however, an [active issue](https://github.com/facebook/jest/issues/1874) and an [open pull request](https://github.com/facebook/jest/pull/3561) for this bug on GitHub. So hopefully we'll see a fix soon.

## Conclusion

Although we experienced some minor pain along the way, overall our team’s transition to Jest was smooth and beneficial. The team enjoyed the new tools at their disposal and most of us agreed the syntax and organizational differences in Jest were a huge step up. If you’ve been on the fence about switching to Jest, maybe you ought to give it a whirl? This guide can help!
