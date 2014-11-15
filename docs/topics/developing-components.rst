.. _developing-on-components:

Developing on Components
========================

This section is about developing on Marketplace Framework components, such as
`Marketplace Core Modules <https://github.com/mozilla/marketplace-core-modules>`_.

Developing Components
~~~~~~~~~~~~~~~~~~~~~

For convenience, you can `git clone` components straight into the source of
your frontend project to where it would normally be copied to. That way, you
can test your modules and handle revisions in the same place (without needing
to copy modules to a different folder to commit).

For example, you can git clone `marketplace-core-modules` straight to
`src/media/js/lib`, and develop and test the modules there.

Be careful as running `make install` may overwrite changes you make
in that directory. Such is the nature of working on shared components.

Updating a Component
~~~~~~~~~~~~~~~~~~~~

When pushing an update for a component:

- Bump the version in `bower.json` (e.g., "1.5.2" to "1.5.3")
- Git tag that version and push to Github (e.g., `git tag v1.5.3 && git push`)

To consume the updated component from a frontend project:

- Bump the component's version in the frontend project's `bower.json`
- Run ```make install``` to get these modules copied into your project and into
  your RequireJS development configuration
- Commit and push the `bower.json` to deploy the updated component for the
  project
