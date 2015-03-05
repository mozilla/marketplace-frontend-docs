Testing
=======

Marketplace frontend projects have both unit tests and end-to-end tests. These
tests help catch regressions, validate user flow, and add confidence the
codebase. When adding a feature, try to look to cover it with tests.

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

- ``<frontend-project>/tests/unit`` for frontend projects
- ``marketplace-core-modules/tests`` for Marketplace Core Modules

To run the unit tests once::

    make unittest

To continuosly run unit tests when files change::

    make unittest-watch

Troubleshooting
---------------

If you encounter an error where the ``karma`` command cannot be found try
running ``rm -rf node_modules && npm install`` to get a fresh copy of the
node dependencies.

How They Work
-------------

The unit tests are powered by RequireJS, in terms of being able to "import"
modules and unit test their interfaces. The tests use the following libraries:

* `Mocha`_ for defining tests.
* `Chai`_ for assertions.
* `Sinon`_ for mocks, stubs and spies.
* `Squire`_ for managing RequireJS.
* `Karma`_ to run the tests in a real browser.

.. _Mocha: http://mochajs.org/
.. _Chai: http://chaijs.com/
.. _Sinon: http://sinonjs.org/
.. _Squire: https://github.com/iammerrick/Squire.js/
.. _Karma: http://karma-runner.github.io/

Writing a Unit Test
-------------------

A basic unit test may look like:

.. code-block:: javascript

    define('tests/unit/some-module', ['tests/unit/helpers'], function(h) {
        describe('someModule.someFunction', function() {
            it('gives the expected value',
                h.injector()
                .mock('someModuleToMock', {mockKey: mockValue})
                .run(['someModule'], function(someModule) {
                    assert.equal(someModule.someFunction(),
                                 'My Expected Value');
                }));
        });
    });

Important things to note:

- ``describe`` and ``it`` come from `Mocha`_. When the tests run the strings
  passed to ``describe`` and ``it`` will be combined to describe the test in
  the output. This test will be called "someModule.someFunction gives the
  expected result".
- ``h.injector()`` is a shorthand for ``new Squire()``. It also takes a
  variable number of functions as arguments to intialize the mocking. These
  functions should accept the injector and call its ``mock`` method. There are some
  helpers included in ``tests/unit/helpers.js``, such as ``mockSettings`` which
  can be used as: ``h.injector(h.mockSettings({mySetting: 'foo'}))``.
- Calling ``run`` on the injector will automatically end the test for synchronous
  code. If you have asynchronous code you will need to use ``require`` instead and
  call Mocha's ``done()`` function when complete.
- See the `Squire`_ page for documentation on how to use Squire.


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

  casper.test.begin('Test Some Selector', {
      setUp: function() {
        // Setup ran before the test.
      },

      tearDown: function() {
        // Teardown ran after the test.
      },

      test: function(test) {
          helpers.startCasper({path: '/some/path'});

          helpers.waitForPageLoaded(function() {
              // Run an assertion.
              test.assertVisible('.some-selector',
                                 'Check that Some Selector is visible');
              casper.click('.go-to-some-page');
          });

          casper.waitForSelector('.some-page', function() {
              test.assertVisible('.some-page',
                                 'Check navigated to Some Page');
          });

          helpers.done(test);  // Required for test to run!
      },
  });

``helpers`` is always available and contains useful boilerplate such as for
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

Mocking Login
-------------

To mock login, run ``helpers.fake_login()``. This will, within the
PhantomJS browser context, set a fake shared-secret token, set user's apps and
settings, add a login state on the body, and then asynchronously reload the
page.

Usually, you will run ``fake_login()`` and then immediately use a
``helpers.waitForPageLoaded()`` to wait for the ``fake_login()``
to reload the page.

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
``body.loaded`` which is how we know the page is done rendering. We can also do
this when we click around with ``casper.click``, and tell CasperJS to wait
until a selector we expect to be visible is loaded.

Here is a list of commonly used `waitFor` methods:

* `waitForSelector <http://docs.casperjs.org/en/latest/modules/casper.html#waitforselector>`_ -
   wait for selector to exist in the DOM
* `waitWhileVisible <http://docs.casperjs.org/en/latest/modules/casper.html#waitwhilevisible>`_ -
   wait for selector to disappear
* `waitUntilVisible <http://docs.casperjs.org/en/latest/modules/casper.html#waituntilvisible>`_ -
   wait for selector to appear
* `waitForUrl <http://docs.casperjs.org/en/latest/modules/casper.html#waitforurl>`_ -
   wait until casper has moved to the desired or matching url
*  helpers.waitForPageLoaded -
   a custom waitFor helper we wrote that waits for page to load (``body.loaded``)

You can make custom `waitFor
<http://docs.casperjs.org/en/latest/modules/casper.html#waitfor>`_ by defining
a function that returns true when a custom condition is met.

Debugging Tests
---------------

Some useful tips when debugging a failing test:

- Set the system environment variable, ``SHOW_TEST_CONSOLE``, to see every ``console.log``
that is sent to the client-side console. This is useful for
debugging tests.
- Whenever a test fails, CasperJS will automatically take a screenshot using
PhantomJS. The screenshot is stored in the ``tests/captures`` directory. Check
it out to see what the page looked like when an assertion fails.

Tips and Guidelines
-------------------

- Keep tests organized. Ideally, each test file tests a page or component,
  and each test (``casper.begin('Test...')``) tests a specific part of that
  page or component.
- If testing a page, place the test file in a location that would match the
  route of the page.
- If you write something reusable, consider adding it to ``helpers.js``
- If you use a constant, consider adding it to ``constants.js``
- Keep selectors short and specific. We don't want tests to break as UI changes
  are made. One-class-name selectors are preferred over element selectors.
- Avoid specific string checking as the test may break if strings are updated.
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
