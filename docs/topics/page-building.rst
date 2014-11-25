.. _page-building:

Page Building
=============

This section details the entire process of page building, from routing to
views to the builder object to the defer block. It is MVC-esque and is the
meat of the Marketplace Framework.

.. _routing:

Routing
~~~~~~~

All pages have a route, or URL, that is used by the framework to resolve the
appropriate view:

- On page load or click of an anchor tag, ``navigation.js`` calls ``views.js``
- ``views.js`` looks up the appropriate view for the current window path
- ``views.js`` loops over the list of routes built by the project's ``routes.js``
- When ``views.js`` finds a match, it returns the view module that matches the path

Path-to-view mappings are defined within ``src/media/js/routes.js``. For
example, ``routes.js`` may look like:

.. code-block:: javascript

    function() {
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

The routes mapping is created outside of the define call. It is a list of
objects containing a *pattern* and a *view_name*:

- *pattern* -a regular expression used to match against the window pathname
- *view_name* - name of the view module to resolve to. The view module should
  live in ``src/media/js/views`` and the original name should be prefixed with
  ``views/``. But the prefix should be left out of the *view_name* as it is
  automatically prepended in the define call.

In the define call, the list of routes is looped over and actual view modules
to are attached to the route objects. Note that the define call **must** list
every view's module name as a dependency as seen above.

Adding a Route
--------------

To add a route:

- Add a route object to the list of ``routes`` at the top.
- The route object should contain a *pattern* regex to match its respective window pathname
- The route object should contain a *view_name* to resolve to
- Add the full view name to the dependency list of ``routes``'s define call

.. _views:

Views
~~~~~

.. _builder-object:

Builder Object
~~~~~~~~~~~~~~

.. _defer-block:

Defer Block
~~~~~~~~~~~
