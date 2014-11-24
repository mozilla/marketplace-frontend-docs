.. _developing-components:

Developing Components
=====================

This section is about developing, testing, and updating Marketplace Framework
components such as:

- `Marketplace Core Modules <https://github.com/mozilla/marketplace-core-modules>`_
- `Marketplace Gulp <https://github.com/mozilla/marketplace-gulp`_.
- `Commonplace <https://github.com/mozilla/commonplace`_.

We have two types of components: Bower components and Node modules.

Bower Components
~~~~~~~~~~~~~~~~

Bower is the package manager we use for frontend assets. The assets are
downloaded to ``bower_components`` and copied into the project with ``make
install``.

Development and Testing Workflow
--------------------------------

For convenience, you can ``git clone`` components straight into the source of
your frontend project to where it would normally be copied to. That way, you
can test your modules and handle revisions in the same place (without needing
to copy modules to a different folder to commit).

For example, you can git clone ``marketplace-core-modules`` straight to
``src/media/js/lib``, and develop and test the modules there.

Be careful as running ``make install`` may overwrite changes you make in that
directory, which will copy the source files from bower_components over to the
project directory.

Updating a Component
--------------------

When pushing an update for a component:

- Bump the version in ``bower.json`` (e.g., *1.5.2* to *1.5.3*)
- Git tag the version (e.g., ``git tag v1.5.3``)
- Push to Github (i.e., ``git push origin v1.5.3 && git push origin master``)

To **consume** the updated component from a frontend project:

- Bump the component's version in the frontend project's ``bower.json``
- Run ``make install`` to get these modules copied into your project and into
  your RequireJS development configuration
- Commit and push the ``bower.json`` to deploy the updated component for the
  project


Node Modules
~~~~~~~~~~~~

We have node modules such as Commonplace and Marketplace Gulp.

Development and Testing Workflow
--------------------------------

When developing a node module, it is annoying to have to continually rebuild
it or copy files back and forth between a version-tracked directory.
``npm link`` eases this:

- Go to the project directory of the node module you are developing (e.g., marketplace-gulp)
- Run ``npm link`` to create a global link
- Go to the project directory of the frontend project you want test it with (e.g., fireplace)
- Run ``npm link <PACKAGE_NAME>`` to link-install the package

Then changes are made in the node module's project directory will be reflected
within the frontend project it is being tested with.

Updating a Module
-----------------

When pushing an update for a module:

- Bump the version in ``package.json`` (e.g., *1.5.2* to *1.5.3*)
- Commit and push to Github (i.e., ``git push origin master``)
- Run ``npm publish`` to publish it to npm

To **consume** the updated component from a frontend project:

- Bump the module's version in the frontend project's ``package.json``
- Run ``make install`` to update the project's node modules
- Commit and push the ``package.json`` to deploy the updated module for the
  project
