.. _usage:

Usage
=====

This section is about setting up the frontend projects (which is very easy),
and the command-line interface powered by our build system, `Marketplace Gulp
<https://github.com/mozilla/marketplace-gulp>`_, that ships with each project.

Installation
------------

You will need Node and NPM installed.

Setting up an instance of the Marketplace frontend for development is very
simple. For this example, we'll check out
`Fireplace <https://github.com/mozilla/fireplace>`_::

    git clone git@github.com:mozilla/fireplace
    make init
    make serve

This will:

* Clone the project
* Install Node and Bower dependencies
* Copy assets from `bower_components` into the project source directory
* Generate a local settings file at `src/media/js/settings_local.js`
* Generate a require.js with an injected paths and shim configuration
* Generate an index.html file with an injected LiveReload script
* Start a local webserver with a filesystem watcher.


Pulling in Updates
------------------

When fetching updates (e.g., `git pull`), due to often-updating Bower and Node
dependencies, you will often have to run::

    make update

This will also do every part of the installation step above, except for
generating a local settings file (as to not overwrite your current one).


About the Webserver
-------------------

The webserver launched by `make serve` will watch the filesystem for changes
and recompile anything if necessary. Here is what the webserver watches for:

* When a Stylus file is modified, then only that specific Stylus file will
  be recompiled.
* When a Stylus library file is modified (`css/lib` or `css/lib.styl`), every
  Stylus files will be recompiled.
* When a HTML/Nunjucks template is modified, `src/templates.js` will be
  recompiled via Nunjucks.
* When a root HTML file is modified (`src/*.html`), `index.html` will be
  re-generated from the active template.

To run the webserver on a different port::

    PORT=8000 make serve

To serve a different template other than `dev.html` (in case you want to test
the project with bundled CSS/JS), pass in a template name that lives within
the root of the `src` directory::

    TEMPLATE=app make serve

The webserver will also launch a LiveReload server for CSS modifications. When
CSS is recompiled, the browser sessions will automatically refresh their
CSS stylesheets live without a page refresh. To disable LiveReload::

    NO_LIVERELOAD=1 make serve


Building the Project for Production
-----------------------------------

Outside of the dev environment, we bundle all of our CSS, JS, and templates.
We use Gulp to build our projects; our Gulp code lives in
`Marketplace Gulp <https://github.com/mozilla/marketplace-gulp`_. To run the
build system::

    make build

About the CSS builds:

* CSS will be minified and concatenated into `src/media/css/include.css`.
* Extra CSS bundles can be configured in `config.js`, such as we do for
  `src/media/css/splash.css`.

About the JS builds:

* JS will be run through a `RequireJS optimizer <http://requirejs.org/docs/optimization.html>`_
* The JS will be bundled into `src/media/js/include.js`.
* `almond <http://github.com/jrburke/almond>`_, a lightweight AMD loader, will
  be prepended onto the bundle.
* A sourcemap will be generated at `src/media/js/include.js.map`.

About template builds:

* We use Nunjucks to compile our templates into `src/templates.js`.
* We have Node modules that monkeypatch the official Nunjucks compiler to
  perform various (non-upstream compatible) optimizations to reduce the size
  of our templates bundle.

If you want to test the project with the built bundles above, serve with a
template that uses the bundle, such as `src/app.html`. Read about the webserver
above for more details.

If you want to disable uglification and minification of JS and CSS::

    NO_MINIFY=1 make build
