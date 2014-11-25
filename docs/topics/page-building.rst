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


.. _views:

Views
~~~~~

.. _builder-object:

Builder Object
~~~~~~~~~~~~~~

.. _defer-block:

Defer Block
~~~~~~~~~~~
