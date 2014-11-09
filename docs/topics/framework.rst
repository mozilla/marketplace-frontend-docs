.. _framework:

Marketplace Framework
=====================

This section describes on a high-level the framework that we use when
developing our frontend projects. Let's call it the Marketplace Framework. The
Marketplace Framework is an in-house MVC framework comprised of
`AMD modules <https://github.com/amdjs/amdjs-api/blob/master/AMD.md>` and
`Nunjucks templates <https://http://mozilla.github.io/nunjucks/>`
that allows us to performant single-page apps. We'll go over in-depth what the
Marketplace Framework looks like.

If you are curious why we have our own framework, and not use something like
Backbone or Angular? Simply put for Backbone, it had a lot stuff we didn't need
like syncing and data manipulation as the main Marketplace is largely
read-only, and it didn't suit our needs. Angular was tempting, but we wanted
the flexibility to render templates server-side, and the l10n support wasn't
quite there.

If you want a quick run-down, you can watch a 20-minute video of the "masta"
of the framework, Matt Basta,
`talking about the framework and the decisions behind it
<https://air.mozilla.org/building-the-firefox-marketplace/>`_. We highly
recommend watching it.

Goals of the Marketplace Framework
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Performance (and perceived performance) on low-end devices is the headlining
goal:

* Have everything client-side (even the templates) for less requests
* Use RESTful APIs for everything for asynchronous and deferred loading
* Always show something (throbbers) to the user while data is being fetched
* Cache as much as possible
* Pave the way for the Marketplace to become a
  `packaged app <https://developer.mozilla.org/Marketplace/Options/Packaged_apps>`_.

Everything Client-Side
~~~~~~~~~~~~~~~~~~~~~~

Here is a simple diagram of the client-side Marketplace (codename Fireplace).
It's pretty straightforward description of a single-page app:

.. image:: img/clientside.png

* Data is requested from the API
* The client receives the data
* The client renders the page with the data
* The client stores the data in its request cache (key-value store keyed by URL)

When we navigate to another page, we simply have to fetch data from another
endpoint since the page rendering happens client-side. When we navigate to an
already-visited page, the request goes to the cache. Less requests, better
performance.

Fetching and Rendering Data from RESTful APIs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While an asynchronous request to an API endpoint is being made, we generally
want to show something to the user in the process, like a spinner. And when
the data comes in, we want to replace that *placeholder* with rendered
markup. To do so, the Marketplace Frontend implements in the page rendering
engine something called a *defer block*, which is used from the templates. Here
is a visual representation using for example an
`Marketplace app detail page <https://marketplace.firefox.com/app/twitter>`_:

.. image:: img/deferblocks.png

From the templates, it might look like::

    {% defer (url='https://marketplace.firefox.com/api/v2/app/twitter') %}
      <h1 class="app-name">{{ this.name }}</h1>
    {% placeholder %}
      <p>Loading app data...</p>
      <div class="spinner"></div>
    {% end %}

Each defer block is a request to an API endpoint. In the defer block
*signature*, you pass in a URL. In the defer block body, you add the templating
markup you want displayed once the data comes in. In the *placeholder block*,
you add templating markup you want displayed while the data is loading. The
defer block is one of the magical facilities that the Marketplace Framework
provides.
