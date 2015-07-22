<!--
{
"name" : "mocha-test",
"version" : "0.1",
"title" : "How to Write Your First Test With Mocha",
"description" : "TBD.",
"freshnessDate" : 2015-04-01,
"homepage" : "http://webapplog.com/",
"canonicalSource" : "http://webapplog.com/mocha-test/",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

# Overview

[![leaf-latte-art-mug-700x467](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/leaf-latte-art-mug-700x467.jpg)](http://webapplog.com/mocha-test/ "How to Write Your First Test With Mocha")

The goal of this mini-project is to add a few tests for Blog.
 The project is on GitHub at [https://github.com/azat-co/blog-express](https://github.com/azat-co/blog-express).

We won’t get into headless browsers and UI testing, but we can send a few HTTP requests and parse their responses from the app’s REST end points.

The source code for this tutorial is in the `ch3/blog-express` folder of the practicalnode GitHub repository ([https://github.com/azat-co/practicalnode](http://github.com/azat-co/practicalnode)).

First, let’s copy the Hello World project. It will serve as a foundation for [Blog](https://github.com/azat-co/blog-express). Then, install Mocha in the Blog project folder, and add it to the `package.json` file at the same time with `$ npm install mocha@1.16.2 --save-dev`.

The `--save-dev` flag will categorize this module as a development dependency (devDependencies).

Modify this command by replacing package name and version number for expect.js v0.2.0 and superagent v0.15.7 ([https://npmjs.org/package/superagent](http://npmjs.org/package/superagent)). The latter is a library to streamline the making of HTTP requests. Alternatives to `superagent` include the following:

*   `request`([https://npmjs.org/package/request](http://npmjs.org/package/request)): the third most-starred NPM module (as of this writing)
*   _core_ `http` _module_: clunky and very low level
*   `supertest`: a superagent-based assertions library

Here’s the updated package.json:

```json
    {
      "name": "blog-express",
      "version": "0.0.1",
      "private": true,
      "scripts": {
        "start": "node app.js",
        "test": "mocha test"
      },
      "dependencies": {
        "express": "4.1.2",
        "jade": "1.3.1",
        "stylus": "0.44.0"
      },
      "devDependencies": {
        "mocha": "1.16.2",
        "superagent": "0.15.7",
        "expect.js": "0.2.0"
      }
     }
```

Now, create a test folder with `$ mkdir tests` and open `tests/index.js` in your editor.

The test needs to start the server:

```javascript
        var boot = require('../app').boot,
          shutdown = require('../app').shutdown,
          port = require('../app').port,
          superagent = require('superagent'),
          expect = require('expect.js');
        describe('server', function () {
          before(function () {
            boot();
          });
        describe('homepage', function(){
          it('should respond to GET',function(done){
            superagent
              .get('http://localhost:'+port)
              .end(function(res){
                expect(res.status).to.equal(200);
                done()
            })
          })
        });
        after(function () {
          shutdown();
        });
      });
```

In `app.js`, we expose two methods, `boot` and `shutdown`, when the file is imported, in our case, by the test. So, instead of:

```javascript
    http.createServer(app).listen(app.get('port'), function(){
      console.log('Express server listening on port ' + app.get('port'));
    });
```

we can refactor into:

```javascript
    var server = http.createServer(app);
    var boot = function () {
    server.listen(app.get('port'), function(){
    console.info('Express server listening on port ' + app.get('port'));
    });
    }
    var shutdown = function() {
    server.close();
    }
    if (require.main === module) {
    boot();
    }
    else {
    console.info('Running app as a module')
    exports.boot = boot;
    exports.shutdown = shutdown;
    exports.port = app.get('port');
    }
```

To launch the test, simply run `$ mocha tests` . The server should boot and respond to the home page request (/ route) as shown in Figure 3–3.

[![pn-ch3-3](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/pn-ch3-3.png)](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/pn-ch3-3.png)

Figure 3–3\. Running $ mocha tests

<!-- @section -->

### Putting Configs Into a Makefile

The `mocha` accepts many options. It’s often a good idea to have these options gathered in one place, which could be a Makefile. For example, we can have `test`, `test-w` test all files in the `test` folder, and have modes for just the `module-a.js` and `module-b.js` files to test them separately, this way:

```
    REPORTER = list
    MOCHA_OPTS = --ui tdd --ignore-leaks

    test:
            clear
            echo Starting test *********************************************************
            ./node_modules/mocha/bin/mocha \
            --reporter $(REPORTER) \
            $(MOCHA_OPTS) \
            tests/*.js
            echo Ending test
    test-w:
    ./node_modules/mocha/bin/mocha \
    --reporter $(REPORTER) \
    --growl \
    --watch \
    $(MOCHA_OPTS) \
    tests/*.js
```

Now we can deal with a task that is targeting a single test:

```
    test-module-a:
    mocha tests/module-a.js --ui tdd --reporter list --ignore-leaks

    test-module-b:

    clear
    echo Starting test *********************************************************
    ./node_modules/mocha/bin/mocha \
    --reporter $(REPORTER) \
    $(MOCHA_OPTS) \
    tests/module-b.js
    echo Ending test
```

Finally, we wrap things up with the list of all tasks:

```
    .PHONY: test test-w test-module-a test-module-b
```

To launch this Makefile, run `$ make <mode>`—for example, `$ make test`. For more information on a Makefile please refer to Understanding Make at [http://www.cprogramming.com/tutorial/makefiles.html](http://www.cprogramming.com/tutorial/makefiles.html) and Using Make and Writing Makefiles at [http://www.cs.swarthmore.edu/newhall/unixhelp/howto_makefiles.html](http://www.cs.swarthmore.edu/newhall/unixhelp/howto_makefiles.html).

For our Blog app, we can keep the Makefile simple:

```
    REPORTER = list
    MOCHA_OPTS = --ui bdd –c

    test:
    clear
    echo Starting test *********************************************************
    ./node_modules/mocha/bin/mocha \
    --reporter $(REPORTER) \
    $(MOCHA_OPTS) \
    tests/*.js
    echo Ending test

    .PHONY: test
```

■ **Note** We point to the local Mocha in the Makefile, so the dependency needs to be added to `package.json` and installed in the `node_modules` folder.

Now we can run tests with the `$ make test` command, which allows for more configuration compared with the simple `$ mocha tests`(Figure 3–4 ).

[![Screenshot-2015-03-31-16.27.30](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/Screenshot-2015-03-31-16.27.30.png)](http://m03s6dh33i0jtc3uzfml36au.wpengine.netdna-cdn.com/wp-content/uploads/Screenshot-2015-03-31-16.27.30.png)

Figure 3–4\. Running make test

<!-- @section -->

### Summary

In the [previous Mocha article](http://webapplog.com/tdd/), we installed Mocha as a command-line tool and learned its options, we wrote simple tests with assert and the expect.js libraries.

In this tutorial, we created the first test for [the Blog app](https://github.com/azat-co/blog-express) by modifying `app.js` to work as a module.
