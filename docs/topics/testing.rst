Testing
=======

Marketplace frontend projects have both unit tests and end-to-end tests. These
tests help catch regressions, validate user flow, and add confidence the
codebase.

Our tests expect that you use the
`Marketplace API Mock <https://github.com/mozilla/marketplace-api-mock>`_. You
can do so by setting in your local settings::

    api_url: 'https://flue.paas.allizom.org',
    media_url: 'https://flue.paas.allizom.org'


Unit Tests
~~~~~~~~~~

Unit tests live in:

- ``<frontend-project>/src/tests`` for frontend projects
- ``marketplace-core-modules/tests`` for Marketplace Core Modules

To run unit tests, simply navigate to::

    http://localhost:xxxx/tests

where xxxx is the port the project is being served from with ``make serve``.
You *could* also run it on the command-line with ``node_modules/.bin/casperjs
test tests/ui/unittests.js``, but it's not very descriptive in telling what
passes and what fails.

How They Work
-------------

The unit tests are powered by RequireJS, in terms of being able to "import"
modules and unit test their interfaces. They also use helpers and utilities
from ``assert.js`` and ``views/tests.js``, both of which live in Marketplace
Core Modules. These helpers include allowing us to *mock* modules using
RequireJS contexts.

The test page (at ``/tests``) includes all of the unit tests as script tags for
them to be run all at once. The base test page is sourced from our
`core templates <https://github.com/mozilla/commonplace/tree/master/dist/core-templates>`_,
which is copied into ``src/templates/commonplace/tests.html``. The base test
page includes unit tests for Marketplace Core Modules. Then that page is
extended in ``src/templates/tests.html`` to include unit tests for modules
specific to the frontend project.

The test page keeps track of how many tests finish running, how many tests
pass, and how many tests fail. These numbers are stored in a DOM element
which is actually used by Travis (via CasperJS) to determine whether the unit
tests pass in continuous integration.

Writing a Unit Test
-------------------

A basic unit test may look like::

    test('someModule.someFunction', function(done, fail) {
        mock('someModule', {
            'someModuleToMock': {
                mockKey: mockValue
            }
        }, function(someModule) {
            eq_(someModule.someFunction(), 'My Expected Value');
            someModule.someOtherFunction().fail(fail);
            done();
        }, fail);
    });

Things to note:

- ``test`` is a function defined in ``marketplace-core-modules/views/tests.js``
  as ``window.test``. It takes the name of the test. Then it injects a done and
  fail callback, which should be invoked on test finish and test failure. It
  has behavior to update the ran/pass/fail counts.
- ``mock`` is a function defined in ``marketplace-core-modules/assert.js``. It
  takes the name of the module being tested, and then an object containing
  the mocks. With the object's keys being the name of a module to mock, and
  the value being another object what to mock within the module. This is all
  done using RequireJS contexts. Each time mock is called, a unique RequireJS
  context is generated that uses the ``map`` config of the RequireJS config. It
  then stubs (not extends) the modules with what you pass in as the mock.


End-to-End Tests
~~~~~~~~~~~~~~~~

The best way to write a test for Casper is to take a test from an existing project
and use that as a template.

Taking a Fireplace test as an example let's build up a test file.

First we add the helpers script. This contains utility functions and helpful
wrappers to casper functionality just so we don't have to repeat lots of boilerplate
code all over the place.

.. code-block:: javascript

  var helpers = require('../helpers');

This next line starts casper. If you pass in an object and set the path, this will modify
where casper starts. In this case this will start us off at the homepage of our app.

.. code-block:: javascript

  helpers.startCasper();

Here's an alternative that will send casper to '/app/foo'

.. code-block:: javascript

  helpers.startCasper({path: '/app/foo'});

Now let's look at the most important block. We generally use this object-based
syntax for casper.test.begin.

This object contains a test method with our test code, along with a setUp and tearDown
method. Thes last two are optional if you don't need them you can exclude them.

.. code-block:: javascript

  casper.test.begin('Test system date dialogue', {

      setUp: function() {
        // Setup here
      },

      tearDown: function() {
        // Teardown here
      },

      test: function(test) {

          casper.waitForSelector('#splash-overlay.hide', function() {
              // Run an assertion here e.g:
              test.assertVisible('.date-error', 'Check date error message is shown');
          });

          casper.run(function() {
              test.done();
          });
      },
  });

The last block that contains `test.done()` is very important. Without this your test won't run.

Testing Tips
~~~~~~~~~~~~

When to write a test
--------------------

If you're adding a feature to a project that support front-end tests (Fireplace/Spartacus etc) then
always look to cover your feature with a test.

If you're fixing a bug then this can be a great chance to start with a failing test first,
and then work on the fix until the test passes. This will also cover you should something
cause this bug to re-occur in the future.

When to write a unitest vs a casperjs flow test
-----------------------------------------------

If you're adding something that has defined input/output or can be tested at a low level easily
then a unittest can be the best place to test it. If your code is more involved e.g. it does UI
changes or affects multiple pages/URLS then a casper test is going to be the best approach.


Using `waitFor` to wait for conditions
--------------------------------------

`waitFor <http://docs.casperjs.org/en/latest/modules/casper.html#waitfor>`_ methods are very useful for making casper wait until a condition is met before trying
to test something. Generally you should never need to use a timeout or `casper.wait`

Here's a list of some of the commonly used `waitFor` methods we use:

* `waitForSelector <http://docs.casperjs.org/en/latest/modules/casper.html#waitforselector>`_ - waits for a selector to exist in the DOM.
* `waitWhileVisible <http://docs.casperjs.org/en/latest/modules/casper.html#waitwhilevisible>`_ - used to wait until a selector dissappears.
* `waitUntilVisible <http://docs.casperjs.org/en/latest/modules/casper.html#waituntilvisible>`_ - use to wait until a selector is visible.
* `waitForUrl <http://docs.casperjs.org/en/latest/modules/casper.html#waitforurl>`_ - Wait until casper has moved to the desired or matching url.

Most things are catered for. Always check the API docs to see if what you want is there.

If it's not then you can always use `waitFor <http://docs.casperjs.org/en/latest/modules/casper.html#waitfor>`_ and define your own function that returns
true when your custom condition is met.

If you use a custom condition a lot then consider adding it to `helpers.js`


Avoid testing for specific strings
----------------------------------

We do it in a few places but generally it's good to try and avoid string checking
as it's likely to break when strings are updated.


Check casper's API for existing methods that will do what you want


There's lots and lots of stuff in the API already. Always take a look before
rolling your own function.

`Casper Test module <http://docs.casperjs.org/en/latest/modules/tester.html>`_


Understand the different environments
-------------------------------------

The code in tests doesn't run in the browser environment. When you use casper's API
it's talking to Phantom (or a.n.other backend).

If you want to run something on the browser environment you can use `casper.evaluate`
which then runs the code on the client.

Here's a simple example:

.. code-block:: javascript

    casper.evaluate(function(arg) {
        console.log(arg);
    }, 'test');

See the casper docs for more info.


setUp not running early enough
------------------------------

Sometimes we need to do things in setUp to modify a page to test specific functionality.
One problem that you might find is that setUp fires too early and changes made there don't work
To work around this you can look for the `page.initialized` event.

Here's an example:

.. code-block:: javascript

    setUp: function() {
        casper.once('page.initialized', function() {
            casper.evaluate(function() {
              // Evalaute some JS in the page.
            });
        });
    },
