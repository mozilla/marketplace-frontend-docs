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
~~~~~~~~~~~~~~~~~~

When fetching updates (e.g., `git pull`), due to often-updating Bower and Node
dependencies, you will often have to run::

    make update

This will also do every part of the installation step above, except for
generating a local settings file (as to not overwrite your current one).
