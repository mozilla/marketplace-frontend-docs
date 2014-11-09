.. _about-framework:

Marketplace Framework
=====================

This section describes on a high-level the framework that we use when
developing our frontend projects. Let's call it the Marketplace Framework. The
Marketplace Framework is an in-house MVC framework comprised with AMD modules
and Nunjucks templates that allows us to build single-page apps. We'll go over
in-depth what the Marketplace Framework looks like.

If you are curious why we have our own framework, and not use something like
Backbone or Angular? Simply put for Backbone, it had a lot stuff we didn't need
like syncing and data manipulation as the main Marketplace is largely
read-only, and it didn't suit our needs. Angular was tempting, but we wanted
the flexibility to render templates server-side, and the l10n support wasn't
quite there.

If you want the whole history and hoedown, you can watch a video of the "masta"
of the framework, Matt Basta,
`talk about building the framework and the decisions behind it
<https://air.mozilla.org/building-the-firefox-marketplace/>`_. We highly
recommend watching it.

Goals of the Marketplace Framework
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Performance (and perceived performance) on low-end devices is the headlining
goal:

* Separated frontend to be almost completely client-side (even the templates)
* RESTful APIs for everything for asynchronous loading
* Always show something (throbbers) to the user while data is being fetched
* Lots of client-side caching layers and facilities
* Allows the Marketplace to become a
  `packaged app <https://developer.mozilla.org/Marketplace/Options/Packaged_apps>`_.
