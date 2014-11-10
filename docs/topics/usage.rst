.. _usage:

Usage
=====

This section is about installation, command-line interface, and configuration
of Marketplace frontend projects.

.. _installation:

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
* Start a local webserver with a filesystem watcher


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
  be recompiled
* When a Stylus library file is modified (`css/lib` or `css/lib.styl`), every
  Stylus files will be recompiled
* When a HTML/Nunjucks template is modified, `src/templates.js` will be
  recompiled via Nunjucks
* When a root HTML file is modified (`src/*.html`), `index.html` will be
  re-generated from the active template

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
`Marketplace Gulp <https://github.com/mozilla/marketplace-gulp>`_. To run the
build system::

    make build

About the CSS builds:

* CSS will be minified and concatenated into `src/media/css/include.css`
* Extra CSS bundles can be configured in `config.js`, such as we do for
  `src/media/css/splash.css`

About the JS builds:

* The JS will be bundled into `src/media/js/include.js`
* JS is run through a `RequireJS optimizer <http://requirejs.org/docs/optimization.html>`_
* The RequireJS configuration used for local development is also passed in to
  the our RequireJS optimizer
* `almond <http://github.com/jrburke/almond>`_, a lightweight AMD loader, will
  be prepended onto the bundle
* A sourcemap will be generated at `src/media/js/include.js.map`

About template builds:

* We use Nunjucks to compile our templates into `src/templates.js`
* We have Node modules that monkeypatch the official Nunjucks compiler to
  perform various (non-upstream compatible) optimizations to reduce the size
  of our templates bundle

Other things that will be generated:

* `src/media/build_id.txt` contains the timestamp during that build, which is
  used as a cachebusting query string when we serve our assets in production
* `src/media/imgurls.txt` contains image URLs found in our CSS stylesheets,
  which is used to generate an appcache manifest on the server

If you want to test the project with the built bundles above, serve with a
template that uses the bundle, such as `src/app.html`. Read about the webserver
above for more details

If you want to disable uglification and minification of JS and CSS::

    NO_MINIFY=1 make build


Additional Command-Line Interface
---------------------------------

Most of our commands are brought to you by our build system and task runner,
Gulp. And most of the useful ones have been aliased with Makefile directives
such that we don't have to expect developers to have Gulp installed globally.
For commands that do not have Makefile aliases, if you don't have Gulp
installed globally, you can run Gulp through `node_modules/.bin/gulp`.

You won't often need these, but here is a list of commands not mentioned above:

* `make clean` - deletes generated and temporary files
* `make lint` - lints the project's JS with JSHint
* `gulp bower_copy` - performs the Bower copying step described in Installation
* `gulp require_config` - performs the `require.js` generation described in
                          Installation
* `gulp css_compile` - compiles Stylus files
* `gulp templates_build` - compiles Nunjucks templates
* `node_modules/.bin/commonplace langpacks` - extracts locales into JS modules


Bower and RequireJS Configuration
---------------------------------

Above we mentioned the installation and update steps will:

* Copy assets from `bower_components` into the project
* Generate a require.js with an injected paths and shim configuration

These two things, setting up Bower and RequireJS, do not happen magically. They
are both specifically configured (though with reusable code and handy loops).

The base configuration lives in
`Commonplace <https://github.com/mozilla/commonplace>`_, our Node modules, in
`lib/config.js`. This configuration ships and is required with every frontend
project. It sets up Bower copying paths, and RequireJS paths and shims for
modules that we know ships with every frontend project (e.g.,
marketplace-core-modules).

There are two exported configuration objects, one for Bower and one for
RequireJS. The Bower configuration, `require('commonplace').bowerConfig`, for
example may look like::

    {
        'jquery/jquery.js': 'src/media/js/lib/',
        'marketplace-frontend/src/templates/feed.html': 'src/templates'
    }

The keys of the object specifies the source path of the file within
`bower_components`. The values of the object specify the destination path. The
RequireJS configuration, `require('commonplace').requireConfig`, for example
may look like::

    {
        paths: {
            'jquery': 'lib/jquery'
        },
        shim: {
            'underscore': {
                'exports': '_'
            }
        }
    }

This will be used to generate a `require.js` file that contains an injected
`require.config`. It is also used during our RequireJS optimization build step.
Our project runs on AMD so understanding `RequireJS configuration
<http://requirejs.org/docs/api.html#config>`_ is very helpful.

The base Commonplace configuration can be extended within frontend projects
in `config.js`. It will become straightforward once you check out the file. We
extend the base configuration usually if we want to add a module or component
that only matters one of the several Marketplace frontend projects.
