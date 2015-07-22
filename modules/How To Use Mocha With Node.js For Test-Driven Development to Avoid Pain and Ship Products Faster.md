<!--
{
"name" : "using-mocha-with-node.js-for-test-driven-development",
"version" : "0.1",
"title" : "How To Use Mocha With Node.js For Test-Driven Development to Avoid Pain and Ship Products Faster",
"description" : "TBD.",
"freshnessDate" : 2015-03-31,
"homepage" : "http://webapplog.com/",
"canonicalSource" : "http://webapplog.com/tdd/",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

# Overview

[![leaf-latter-art-mug-dish-700x466](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/leaf-latter-art-mug-dish-700x466.jpg)](http://webapplog.com/tdd/ "How To Use Mocha With Node.js For Test-Driven Development to Avoid Pain and Ship Products Faster")

Test-driven development (TDD) , as many of you might know, is one of the main, agile development techniques. The genius of TDD lies in increased quality of code, faster development resulting from greater programmer confidence, and improved bug detection (duh!).

Historically, web apps have been hard to autotest, and developers relied heavily on manual testing. But, certain parts such as standalone services and REST API can be _and should be_ tested thoroughly by the TDD. At the same time, rich user interface (UI) / user experience (UX) can be tested with headless browsers such as PhantomJS.

The behavior-driven development (BDD) concept is based on TDD. It differs from TDD in language, which encourages collaboration between product owners and programmers.

Similar to building apps themselves, most of the time software engineers should use a testing framework. To get you started with the Node.js testing framework, Mocha, in this post, we cover the following:

*   Installing and understanding Mocha
*   TDD with the assert
*   BDD with Expect.js
*   Project: writing the first BDD test for Blog

The source code for this post is in the `ch3` folder of the `practicalnode` GitHub repository ([https://github.com/azat-co/practicalnode](https://github.com/azat-co/practicalnode)).

<!-- @section -->

### Installing and Understanding Mocha

Mocha is a mature and powerful testing framework for Node.js. To install it, simply run:

`$ npm install –g mocha@1.16.2`

■ **Note** We use a specific version (the latest as of this writing is 1.16.2) to prevent inconsistency in this book’s examples caused by potential breaking changes in future versions of Mocha.

If you encounter the lack-of-permissions issue when you run the previous command, run this:

`$ sudo npm install –g mocha@1.16.2`

To avoid using `sudo`, follow the instructions in from [this resource](https://gist.github.com/isaacs/579814) on how to install Node.js correctly.

■ **Tip** It’s possible to have a separate version of Mocha for each project by simply pointing to the local version of Mocha, which you install like any other NPM module into `node_modules`. The command will be:

`$ ./node_modules/mocha/bin/mocha test_name`

for Mac OS X / Linux.

For an example, refer to “Putting Configs into a Makefile” in the next post on Mocha.

Most of you have heard about TDD and why it’s a good thing to follow. The main idea of TDD is to do the following:

*   Define a unit test
*   Implement the unit
*   Verify that the test passes

BDD is a specialized version of TDD that specifies what needs to be unit-tested from the perspective of business requirements. It’s possible to just write test with good old plain core Node.js module `assert`. However, as in many other situations, using a framework is more preferable. For both TDD and BDD, we’ll be using the Mocha testing framework because we gain many things for “free.” Among them are the following:

*   Reporting
*   Asynchronous support
*   Rich configurability

Here is a list of optional parameters (options) that the `$ mocha [options]` command takes:

*   `-h or --help`: print help information for the Mocha command
*   `-V or --version`: print the version number that’s being used
*   `-r or --require <name>`: require a module with the name provided
*   `-R or --reporter <name>`: use a reporter with the name provided
*   `-u or --ui <name>`: use the stipulated reporting user interface (such as `bdd`, `tdd`)
*   `-g or --grep <pattern>`: run tests exclusively with a matching pattern
*   `-i or --invert`: invert the `--grep` match pattern
*   `-t or --timeout <ms>`: set the test case time out in milliseconds (for example, 5000)
*   `-s or --slow <ms>`: set the test threshold in milliseconds (for example, 100)
*   `-w or --watch`: watch test files for changes while hanging on the terminal
*   `-c or --colors`: enable colors
*   `-C or --no-colors`: disable colors
*   `-G or --growl`: enable Mac OS X Growl notifications
*   `-d or --debug`: enable the Node.js debugger`— $ node --debug`
*   `--debug-brk`: enable the Node.js debugger breaking on the first line`— $ node --debug-brk`
*   `-b or --bail`: exit after the first test failure
*   `-A or --async-only`: set all tests in asynchronous mode
*   `--recursive`: use tests in subfolders
*   `--globals <names>`: provide comma-delimited global names
*   `--check-leaks`: check for leaks in global variables
*   `--interfaces`: print available interfaces
*   `--reporters`: print available reporters
*   `--compilers <ext>:<module>,...`: provide compiler to use

Figure 3–1 shows an example of nyan cat reporter with the command `$ mocha test-expect.js -R nyan`.

[![pn-ch3-nyan](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/pn-ch3-nyan.png)](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/pn-ch3-nyan.png)

Figure 3–1\. Mocha nyan reporter

Usually, when it comes to choosing a type of framework, there are a few options. Mocha is one of the more robust and widely used. However, the following alternatives to Mocha are worth considering:

*   NodeUnit ([http://pivotal.github.com/jasmine](http://pivotal.github.com/jasmine))
*   Jasmine ([http://pivotal.github.com/jasmine](http://pivotal.github.com/jasmine))
*   Vows ([http://vowsjs.org](http://vowsjs.org))

<!-- @section -->

### Understanding Mocha Hooks

A hook is some logic, typically a function or a few statements, which is executed when the associated event happens.

For example, in my [Mongoose course](http://udemy.com/mongoose?couponCode=a), we use hooks to explore the Mongoose library `pre` hooks.

Mocha has hooks that are executed in different parts of suites—before the whole suite, before each test, and so on.

In addition to `before` and `beforeEach` hooks, there are `after()`, and `afterEach()` hooks. They can be used to clean up the testing setup, such as database data

All hooks support asynchronous modes. The same is true for tests as well. For example, the following test suite is synchronous and won’t wait for the response to finish:

```javascript
describe('homepage', function(){
  it('should respond to GET',function(){
    superagent
      .get('http://localhost:'+port)
      .end(function(res){
        expect(res.status).to.equal(200);
    })
  })

```

But, as soon as we add a `done` parameter to the test’s function, our test case waits for the HTTP request to come back:

```javascript
describe('homepage', function(){
  it('should respond to GET',function(done){
    superagent
      .get('http://localhost:'+port)
      .end(function(res){
        expect(res.status).to.equal(200);
        done();
    })
  })

```

Test cases `(describe)` can be nested inside other test cases, and hooks such as `before` and `beforeEach` can be mixed in with different test cases on different levels. Nesting of `describe` constructions is a good idea in large test files.

Sometimes, developers might want to skip a test case/suite `(describe.skip()` or `it.skip())` or make them exclusive (`describe.only()` or `describe.only())`. Exclusivity means that only that particular test runs (the opposite of `skip`).

As an alternative to the BDD interface’s `describe`, `it`, `before`, and others, Mocha supports more traditional TDD interfaces:

*   `suite`: analogous to `describe`
*   `test`: analogous to `it`
*   `setup`: analogous to `before`
*   `teardown`: analogous to `after`
*   `suiteSetup`: analogous to `beforeEach`
*   `suiteTeardown`: analogous to `afterEach`

<!-- @section -->

### TDD with the Assert

Let’s write our first tests with the assert library. This library is part of the Node.js core, which makes it easy to access. It has minimal functionality, but it might be enough for some cases, such as unit tests. After global Mocha installation is finished, a test file can be created in a `test-example` folder:

`$ mkdir test-example`
 `$ subl test-example/test.js`

■ **Note** `subl` is a Sublime Text alias command. You can use any other editor, such as Vi (`vi`) or TextMate (`mate`).

With the following content:

```javascript
var assert = require('assert');
describe('String#split', function(){
  it('should return an array', function(){
    assert(Array.isArray('a,b,c'.split(',')));
  });
})

```

We can run this simple `test.js`(inside the `test-example` folder), which checks for Array type, with:

`$ mocha test`
 or

`$ mocha test.js.
`

The results of these Mocha commands are shown in Figure 3–2 .

[![pn-ch3-2](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/pn-ch3-2.png)](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/pn-ch3-2.png)

Figure 3–2\. Running Array-type test

We can add to our example another test case (it) that asserts equality of array values:

```javascript
    var assert = require('assert');
    describe('String#split', function(){
      it('should return an array', function(){
        assert(Array.isArray('a,b,c'.split(',')))
      });

    it('should return the same array', function(){
      assert.equal(['a','b','c'].length, a,b,c'.split(',').length, 'arrays have equal length');
    for (var i=0; i<['a','b','c'].length; i++) {
    assert.equal(['a','b','c'][i], 'a,b,c'.split(',')[i], i +'element is equal');
    };
  });
})

```

As you can see, some code is repeated, so we can abstract it into beforeEach and before constructions:

```javascript
var assert = require('assert');
var expected, current;
before(function(){
  expected = ['a', 'b', 'c'];
})
describe('String#split', function(){
  beforeEach(function(){
   current = 'a,b,c'.split(',');
  })
  it('should return an array', function(){
    assert(Array.isArray(current));
  });
  it('should return the same array', function({
    assert.equal(expected.length, current.length, 'arrays have equal length');
    for (var i=0; i<expected.length; i++) {
      assert.equal(expected[i], current[i], i + 'element is equal');
    }
  })
})

```

<!-- @section -->

### Chai Assert

In the previous example with test.js and assert, we used the Node.js core module assert. Chai is a subset of that library. We can modify our previous example to use chai assert with following code:

`$ npm install chai@1.8.1`

And in test-example/test.js:

`var assert = require('chai').assert;`

The following are some of the methods from the chai assert library:

*   `assert(expressions, message)`: throws an error if the expression is false
*   `assert.fail(actual, expected, [message], [operator])`: throws an error with `values of actual, expected, and operator`
*   `assert.ok(object, [message])`: throws an error when the object is not double equal (==) to true—aka, truthy (0, and an empty string is false in JavaScript/Node.js)
*   `assert.notOk(object, [message])`: throws an error when the object is falsy, i.e., false, 0(zero),“”(empty string), null, undefined or NaN
*   `assert.equal(actual, expected, [message])`: throws an error when `actual` is not double equal (==) to `expected`
*   `asseret.notEqual(actual, expected, [message])`: throws an error when actual is double equal (==)—in other words, not unequal `(!=)—to expected`
*   `.strictEqual(actual, expected, [message])`: throws an error when objects are not triple equal (===)

For the full chai assert API, refer to the official documentation ([http://chaijs.com/api/assert/](http://chaijs.com/api/assert)).

■ **Note** The chai assert (`chai.assert`) and the Node.js core assert(`assert`) modules are _not 100% compatible_, because the former has more methods. The same is true for chai expect and a standalone expect.js.

<!-- @section -->

### BDD with Expect.js

Expect.js is one of the BDD languages. Its syntax allows for chaining and is richer in features than core module assert. There are two options to use expect.js:

Install as a local module
 Install as a part of the chai library

For the former, simply execute the following:

`$ mkdir node_modules`
 `$ npm install expect.js@0.2.0`

And, use `var expect = require('expect.js')`; inside a Node.js test file. For example, the previous test can be rewritten in expect.js BDD style:

```javascript
    var expect = require('expect.js');
    var expected, current;
    before(function(){
      expected = ['a', 'b', 'c'];
    })
    describe('String#split', function(){
      beforeEach(function(){
        current = 'a,b,c'.split(',');
      })
      it('should return an array', function(){
        expect(Array.isArray(current)).to.be.true;
      });

    it('should return the same array', function(){
      expect(expected.length).to.equal(current.length);
      for (var i=0; i<expected.length; i++) {
        expect(expected[i]).equal(current[i]);
      }
    })
  })

```

For the chai library approach, run the following:

`$ mkdir node_modules`
 `$ npm install chai@1.8.1`

And, use `var chai = require('chai')`; `var expect = chai.expect`; inside a Node.js test file. For example:

`var expect = require('chai').expect;`

■ **Note** `$ mkdir node_modules` is needed only if you install NPM modules in the folder that has neither the `node_modules` directory already nor a `package.json` file.

<!-- @section -->

### Expect.js Syntax

The Expect.js library is very extensive. It has nice methods that mimic natural language. Often there are a few ways to write the same assertion, such as `expect(response).to.be(true)` and `expect(response).equal(true)`. The following lists some of the main Expect.js methods/properties:

*   `ok`: checks for truthyness
*   `true`: checks whether the object is truthy
*   `to.be,to`: chains methods as in linking two methods
*   `not`: chains with a not connotation, such as `expect(false).not.to.be(true)`
*   `a/an`: checks type (works with `array` as well)
*   `include/contain`: checks whether an array or string contains an element
*   `below/above`: checks for the upper and lower limits

■ **Note** Again, there is a slight deviation between the standalone `expect.js` module and its Chai counterpart.

For the full documentation on chai expect.js, refer to [http://chaijs.com/api/bdd/](http://http://chaijs.com/api/bdd/), and for the standalone, refer to [https://github.com/LearnBoost/expect.js/](https://github.com/LearnBoost/expect.js/).

In this post, we installed Mocha as a command-line tool and learned its options, we wrote simple tests with assert and the expect.js libraries.

In the next post _Writing the First BDD Test for Blog_, we’ll created the first test for [the Blog app](https://github.com/azat-co/blog-express) by modifying `app.js` to work as a module. This will be a real example how you can use Mocha in action.
