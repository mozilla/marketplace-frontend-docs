Testing
=======

Marketplace frontend projects have both unit tests and end-to-end tests. These
tests help catch regressions, validate user flow, and add confidence the
codebase. When adding feature, try to look to cover it with test.

Our tests expect that you use the
`Marketplace API Mock <https://github.com/mozilla/marketplace-api-mock>`_. You
can do so by setting in your local settings::

    api_url: 'https://flue.paas.allizom.org',
    media_url: 'https://flue.paas.allizom.org'

Expect to have to modify the Marketplace API Mock when writing tests for new
features. When doing so, try to keep the responses predictable rather than
random as to not introduce flakiness into our tests.

When modifying a module that has defined input and output, write a unit test.
When modifying the UI or something that affects multiple pages and URLs, then
write an end-to-end test.

Unit Tests
~~~~~~~~~~

Unit tests live in:

- ``<frontend-project>/src/tests`` for frontend projects
- ``marketplace-core-modules/tests`` for Marketplace Core Modules

To run unit tests, simply navigate to::

    http://localhost:xxxx/tests

where xxxx is the port the project is being served from with ``make serve``.
You *could* also run it on the command line with ``node_modules/.bin/casperjs
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

A basic unit test may look like:

.. code-block:: javascript

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

Important things to note:

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
- The function that comes after the mock is injected with the module being
  tested. You can make assertions and raise failures in this function. At the
  end, make sure to call the ``done`` callback.
- The last argument is the ``fail`` callback

Check out the ``assert.js`` module to see the testing API.


End-to-End Tests
~~~~~~~~~~~~~~~~

We use `CasperJS <http://casperjs.readthedocs.org/en/latest/>`_
(*v1.1.0-beta3*) to write our end-to-end, or integration, tests. These
Node-based tests live in the ``tests`` directory. This directory comprises of:

- ``captures`` - contains screenshots taken whenever a test fails
- ``lib/constants.js`` - holds reusable constants
- ``lib/helpers.js`` - helps power our tests on top of CasperJS. contains
  various assertion facilities, utilities, and sets up necessary callbacks.
- ``settings_travis.js`` - contains test settings and is copied into the
  project directory during Travis tests (in .travis.yml)
- ``ui`` - holds the actual test suites

CasperJS spins up PhantomJS, a headless browser, and runs the tests. The tests
*usually* consist of telling CasperJS what to click and then asserting that a
selector is visible. An example test:

.. code-block:: javascript

  var helpers = require('../helpers');

  casper.test.begin('Test Some Selector', {
      setUp: function() {
        // Setup ran before the test.
      },

      tearDown: function() {
        // Teardown ran after the test.
      },

      test: function(test) {
          helpers.startCasper({path: '/some/path'});

          casper.waitForSelector('#splash-overlay.hide', function() {
              // Run an assertion.
              test.assertVisible('.some-selector',
                                 'Check that Some Selector is visible');
              casper.click('.go-to-some-page');
          });

          casper.waitForSelector('.some-page', function() {
              test.assertVisible('.some-page',
                                 'Check navigated to Some Page');
          });

          casper.run(function() {
              test.done();  // Required for test to run!
          });
      },
  });

We require ``helpers``, which contains useful boilerplate such as for
initializing CasperJS. We pass a path to ``startCasper`` which the page
CasperJS will tell PhantomJS to initially load. Try to use ``startCasper``
within the ``test function`` as to keep the Casper environment isolated.

We begin a test, named *Test Some Selector*, which takes an object. The
``test`` function is injected with the `CasperJS test module
<http://docs.casperjs.org/en/latest/modules/tester.html>`_ which contains
assertion facilities and callbacks. Then we run the test, but make sure that
the ``test.done()`` callback is invoked at the end.

Check out the CasperJS docs and `our existing Fireplace tests
<https://github.com/mozilla/fireplace/tree/master/tests/ui>`_ for clues on how
to write end-to-end tests for our frontend projects.

Debugging Tests
---------------

Some useful tips when debugging a failing test:

- Whenever a test fails, CasperJS will automatically take a screenshot using
PhantomJS. The screenshot is stored in the ``tests/captures`` directory. Check
it out to see what the page looked like when an assertion fails.
- If the failing test is related to a ``waitFor``, try increasing the timeout
of the ``waitFor`` by passing the timeout in milliseconds as the third argument
to the ``waitFor`` (where the first argument is the callback, and the second is
the timeout callback). The default timeout is 5000ms.

Mocking Login
-------------

To mock login, run ``require('helpers').fake_login()``. This will, within the
PhantomJS browser context, set a fake shared-secret token, set user's apps, add
a login state on the body, and then reload the page.

Reloading the page takes a bit of time. Most likely, you will have to increase
the timeout of the subsequent ``waitFor`` to at least 8 seconds.

Executing Code Within the Browser Environment
----------------------------------------------

The code within the tests themselves executes in Node runtime, not PhantomJS
browser runtime. CasperJS handles the communication to the PhantomJS browser.
If you wish to run something within browser environment, you can use
``casper.evaluate``:

.. code-block:: javascript

    var returnValue = casper.evaluate(function() {
        window.querySelector('.some-selector').setAttribute('data-value', value);
        return window.querySelector('.some-selector').getAttribute(value);
    });

``casper.evaluate`` runs synchronously and is allowed to return primitive
values up to the Node runtime.

Using waitFor's
---------------

`waitFor <http://docs.casperjs.org/en/latest/modules/casper.html#waitfor>`_
methods are useful for making CasperJS wait until a condition is met before
running assertions. Generally, timeouts should be avoided with `casper.wait`.

For example, on many tests, we tell CasperJS to ``waitForSelector`` on
``#splash-overlay.hide`` because the splash screen being hidden is one of the
ways we can tell that the page has sufficiently loaded. We can also do this
when we click around with ``casper.click``, and tell CasperJS to wait until a
selector we expect to be visible is loaded.

Here is a list of commonly used `waitFor` methods:

* `waitForSelector <http://docs.casperjs.org/en/latest/modules/casper.html#waitforselector>`_ -
   wait for selector to exist in the DOM
* `waitWhileVisible <http://docs.casperjs.org/en/latest/modules/casper.html#waitwhilevisible>`_ -
   wait for selector to disappear
* `waitUntilVisible <http://docs.casperjs.org/en/latest/modules/casper.html#waituntilvisible>`_ -
   wait for selector to appear
* `waitForUrl <http://docs.casperjs.org/en/latest/modules/casper.html#waitforurl>`_ -
   wait until casper has moved to the desired or matching url

Or you can make a custom
`waitFor <http://docs.casperjs.org/en/latest/modules/casper.html#waitfor>`_ by
defining a function that returns true when a custom condition is met.

Viewing Console Logs (Verbose)
------------------------------

Set the system environment variable, ``SHOW_TEST_CONSOLE``, to see every
``console.log`` that is sent to the client-side console. This is useful for
debugging tests.

Tips and Guidelines
-------------------

- If you write something reusable, consider adding it to ``helpers.js``
- If you use a constant, consider adding it to ``constants.js``
- If you are testing common functionality across multiple pages, give it its
  own test suite and loop a test suite over multiple pages...rather than
  rewriting a test across multiple test suites.
- Try to keep selectors short, but concise. We don't want tests to break
  often as we move around and change elements. One-class-name selectors are
  usually preferred.
- Try to avoid specific string checking as the test may break if strings
  are updated
- If ``setUp`` is firing too early, then try running the code within
  ``casper.once('page.initialized', function() {...)``.


Continuous Integration (Travis)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On every commit (on projects that have a ``.travis.yml``), a `Travis
<https://travis-ci.org/>`_ build is triggered that runs the project's test
suite (both unit and end-to-end tests). ``.travis.yml`` sets up the continuous
integration testing process.

For the Marketplace frontend, tests are run using the
`Marketplace Mock API <http://github.com/mozilla/marketplace-mock-api>`_. A
specific settings file for is used for Travis, found in
``tests/settings_travis.js``.

Results of each build are posted to the IRC channel,
``irc.mozilla.org#amo-bots``.
