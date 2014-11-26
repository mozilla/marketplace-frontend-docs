.. _page-building:

Page Building
=============

This section details the entire process of page building, from routing to
views to the builder object to the defer block. It is MVC-esque and is the
meat of the Marketplace Framework.

.. _routing:

Routing
~~~~~~~

All pages should have a route that which is used to resolve the appropriate
view. To make a page accessible, you must create a route in the project's
``src/media/js/routes.js``. A route is represented by an object in the list of
routes. The ``routes.js`` file may look like:

.. code-block:: javascript

    function() {
        // The list of routes.
        var routes = window.routes = [
            {'pattern': '^$', 'view_name': 'homepage'},
            {'pattern': '^/app/([^/<>"\']+)/?$', 'view_name': 'app'},
        ];

        define('routes', [
            'views/app',
            'views/homepage'
        ], function() {
            for (var i = 0; i < routes.length; i++) {
                var route = routes[i];
                var view = require('views/' + route.view_name);
                route.view = view;
            }
            return routes;
        });
    }();

Each route object should match the following prototype:

.. code-block:: javascript

    {
        'pattern': '<regexp>',  // Regular expression to match current path.
        'view_name': '<view name>'  // Name of view module w/o 'views/' prefix.
    }

In the define call, the list of routes is looped over and the respective view
modules are attached to the route objects. Note that the define call **must**
list every view's module name as a dependency as seen above.

When navigating or loading a page, ``marketplace-core-modules/views.js`` will
loop over all the routes in sequence and try to match it with the current path.
If no match is found, it will just redirect the browser to the navigating URL.
When a pattern is matched, the view (which returns a builder object as we will
see later) will be invoked to build the page.

Regular Expressions
-------------------

There are some quirks with regular expressions in routes:

- **non-matching groups may not be used** - cannot use a group simply to
  tell the parser that something is optional. Groups are only for parameters
  during matching the current path and reverse URL resolving.
- **? may not be used to perform greedy matching** - the optional operator
  can only be performed on static, non-repeating components.
- **multiple optional groups are not allowed** - optional groups are not evaluated
  greedily. Use multiple routes instead.


.. _building-views:

Building Views
~~~~~~~~~~~~~~

Each view is represented by its own module in the ``src/media/js/views``
directory. Here's an example view:

.. code-block:: javascript

    define('views/myview', ['l10n', 'z'], function(l10n, z) {
        // -- MODULE LEVEL --.
        var gettext = l10n.gettext;

        z.page.on('click', 'button', function() {
            console.log('Put your delegated event handlers on this level.');
        });

        return function(builder) {
            // -- BUILDER LEVEL --.
            builder.z('title', gettext('My View Page Title'));
            builder.z('type', 'leaf');

            builder.start('some_template.html', {
                someContextVar: true
            });
        };
    });

At the the module level, the view usually consists of mostly delegated event
handlers. At the *Builder level*, inside the function returned by the view,
consists of code that handles the actual page rendering using the injected
*Builder object*.

Guidelines
----------

Here are some important guidelines around building views:

- You should **not** perform **any** delegated event bindings on elements that
  are not a part of the page
  (e.g., ``z.page.on('click', '.element-from-another-page'...)``).
- And all delegated event bindings on non-page elements should be bound at the
  module level. This prevents *listener leaks*.
- Expensive lookups, queries, and data structures should be done **outside**
  the Builder level. This ensures that any value is computed only once.
- Delegated events should be used whenever possible, and state should be
  preserved on affected elements (e.g., via data attributes) rather than within
  variables.
- State variables should never exist at the module level unless it represents
  a persistent state.

The Builder Object
------------------

A Builder object is injected into the function returned by the view when.
``marketplace-core-modules/views.js`` matches the respective route and invokes
the view. The Builder object is defined in ``marketplace-core-modules/builder.js``.
It is the pure meat behind page rendering, handling Nunjucks template
rendering, defer block injections, pagination, requests, callbacks, and more.

And to note **the Builder object itself is a promise object** (jQuery
Deferred-style). It has accepts ``.done()`` and ``.fail()`` handlers. These
**represent the completion of the page as a whole** (including asynchronous API
requests via defer blocks). However, the promise methods should not be used
to set event handlers or modify page content. The ``builder.onload`` callback
should be used instead which happens when each defer block returns, which
makes the view more reactive and non-blocking. This will be described more
below.

In the barebones example above, we set some context variables with ``z``.  Then
we calls ``builder.start`` to start the build process, passing in the template
to render and a context to render it with. After some magic, the page will
render.

For more details and functionality, below describes the Builder object's API:

.. function:: builder.start(template[, context])

    Starts the build process by rendering a base template to the page.

    :param template: path to the template
    :param context: object which provides extra context variables to the template
    :rtype: the Builder object

.. function:: builder.z(key, value)

    Sets app context values. Any values are accepted and are stored in
    require('z').context. However, certain keys have special behaviors. 'title'
    sets the window's page title. 'type' sets the body's data-page-type data
    attribute.

    :param key: name of the context value (e.g., 'title', 'type')
    :param value: the context value
    :rtype: the Builder object

.. function:: builder.onload(defer_block_id, handler_function)

    Registers a callback for a defer block matching the defer_block_id.
    Depending on whether the defer block's URL or data was request-cached or
    model-cached, it may fire synchronously.

    Given an example defer block ``{% defer (url=api('foo'), id='some-id') %}``,
    we can register a handler for when data is finished fetching from the 'foo'
    endpoint like ``builder.onload('some-id', function(endpoint_data) {...``

    :param defer_block_id: ID of the defer block to register the handler to
    :param handler_function: the handler, which is given the resulting data
                             from the defer block's API endpoint
    :rtype: the Builder object

.. function:: builder.results

    An object containing results from API responses triggered in defer blocks.
    The results will be keyed by the ID of the defer blocks.

.. function:: builder.terminate()

    While the Builder object is often not accessible, if a reference to it
    is available and the current viewport is changing pages, calling
    builder.abort will terminate all outstanding HTTP requests and begin
    cleanup. This should never be called from a builder.onload handler.

    :rtype: the Builder object


.. _defer-block:

Defer Blocks
~~~~~~~~~~~~

In the :ref:`framework` section about :ref:`fetching-restful-apis`, we
introduced **defer blocks** as a page rendering component, invoked from the
templates, that asynchronously fetches data from APIs and renders a template
fragment once finished. We *heavily* recommend reading that section if you have
not already. As we described the Builder as the meat of the framework, defer
blocks are the magic.

.. function:: {% defer (api_url[, id[, pluck[, as[, key[, paginate]]]]]) %}

    Fetches data from api_url, renders the template in the body, and injects
    it into the page.

    :param api_url: the API endpoint URL. Use the ``api`` helper and an API
                    route name to reverse-resolve a URL. The response will
                    be made available as variables ``this`` and ``response``
    :param id: ID of the defer block, used to store API results and fire
               callbacks on the Builder object (Builder.onload)
    :param pluck: extracts a value from the returned response and reassigns the
                  value to ``this``. Often used to facilitate model caching.
                  The original value of ``this`` is preserved in the variable
                  ``response``.
    :param as: determines which model to cache ``this`` into
    :param key: determines which field of ``this`` to use as key to store the
                object in the model. Used in conjunction with ``as``
    :param paginate: selector for the wrapper of paginatable content to enable
                     continuous pagination

A basic defer block looks like::

    {% defer (url=api('foo')) %}
      <h1 class="app-name">{{ this.name }}</h1>
    {% placeholder %}
      <div class="spinner"></div>
    {% end %}

In this example, the defer block asynchronously loads the content at
api('foo').  While waiting for the response, we can show something in the
meantime with the ``placeholder``. Once it has loaded, the content within the
defer block is injected into the page's template by replacing the
placeholder. ``this`` will then contain the API response returned form the
server.

Note that if two defer blocks use the same API endpoint, the request will only
be made once in the background and then cached.

Pagination
----------

Pagination is done by passing in a selector to the defer block's ``paginate``
and having an element ``.loadmore button`` inside the defer block. When the
button is clicked, the defer block will re-run using the button's ``data-url``
as the API URL. The template is re-rendered with the new (next page) content
while keeping what was previously within the pagination container. Thus, it
essentially seem as if the new content was just appended to the pagination
container. For example:

.. code-block:: javascript

    {% defer (url=api('foo'), paginate='ul.list') %}
      <ul class="list">
        {% for result in this %}
          <li>{{ this.bar }}</li>
        {% endfor %}
      </ul>
      <div class="load_more">
        <button data-url="{{ api('this.meta.next_page') }}">Load More</button>
      </div>
    {% end %}

If pagination fails, a fallback template will be loaded instead as specified
by ``settings.pagination_error_template``. Note that this is separate behavior
from the ``{% except %}`` block which renders when an API request errors.
The error template's context is passed the variable ``more_url``, the URL
originally requested and failed. This is useful for creating a retry button.
