.. _developing-components:

Developing Components
=====================

This section is about developing, testing, and updating Marketplace Framework
components such as:

- `Marketplace Core Modules <https://github.com/mozilla/marketplace-core-modules>`_
- `Marketplace Gulp <https://github.com/mozilla/marketplace-gulp>`_.
- `Commonplace <https://github.com/mozilla/commonplace>`_.

We have two types of components: Bower components and Node modules.

Bower Components
~~~~~~~~~~~~~~~~

Bower is the package manager we use for frontend assets. The assets are
downloaded to ``bower_components`` and most are currently copied into the
project with ``make install``.

Development and Testing Workflow
--------------------------------

To develop changes for ``marketplace-core-module`` first checkout a
local copy using ``git clone``. Once you have the code you can ``cd``
into the folder and run ``bower link`` (see the `bower docs`_ for
more information). ``bower`` will now know that this is your local
copy of ``marketplace-core-modules``. To use your local copy in a
frontend project like ``fireplace`` you can run
``bower link marketplace-core-modules`` in the frontend project's
directory to tell ``bower`` to use your local copy. If you are not
making local changes to ``marketplace-core-modules`` then you should
not use a local copy to ensure that you are up to date. To stop
using your local copy simply uninstall it with
``bower uninstall marketplace-core-modules`` and then run
``make install`` to get the latest version.

For most other Bower components, they are copied into JS and CSS ``lib``
directories into the source tree where the project can locate them and then
gitignored. You can employ some strategies for developing these such as setting
up symlinks. We are sorry for this.

.. _bower docs: http://bower.io/docs/api/#link

Updating a Component
--------------------

When pushing an update for a component:

- Bump the version in ``bower.json`` (e.g., *1.5.2* to *1.6.0*), following
  semver
- Commit the patch and push (e.g., ``git commit -m v1.6.0 important bug fix``)
- Git tag the version (e.g., ``git tag v1.6.0``)
- Push to Github (i.e., ``git push origin v1.6.0 && git push origin master``)

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

When developing a node module, it is annoying to have to continually rebuild it
or copy files back and forth between a version-tracked directory.  ``npm link``
eases this:

- Go to the project directory of the node module you are developing
  (e.g., marketplace-gulp)
- Run ``npm link`` to create a global link
- Go to the project directory of the frontend project you want test it with
  (e.g., fireplace)
- Run ``npm link <PACKAGE_NAME>`` to link-install the package

Then changes are made in the node module's project directory will be reflected
within the frontend project it is being tested with.

When doing ``npm link`` for marketplace-gulp, make sure your project's gulpfile
looks similar to this `reference gulpfile
<https://github.com/mozilla/marketplace-template/blob/master/gulpfile.js>`_. We
require tasks exposed by marketplace-gulp, and attach it to the local Gulp
object. This is a workaround for issues with ``npm link`` and Gulp.

Updating a Module
-----------------

When pushing an update for a module:

- Bump the version in ``package.json`` (e.g., *1.5.2* to *1.6.0*)
- Commit and push to Github (i.e., ``git push origin master``)
- Run ``npm publish`` to publish it to npm

To **consume** the updated component from a frontend project:

- Bump the module's version in the frontend project's ``package.json``
- Run ``make install`` to update the project's node modules
- Commit and push the ``package.json`` to deploy the updated module for the
  project


Guidelines
~~~~~~~~~~

When updating a module:

- Try to bump the ``bower.json`` or ``package.json`` in the same commit
- Prepend the commit message with the version (e.g., *v1.5.3 updated logs*)
- Use `semver's Semantic Versioning <http://semver.org/>`_
