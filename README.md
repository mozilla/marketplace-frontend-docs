[![Documentation Status](https://readthedocs.org/projects/marketplace/badge/?version=latest)](https://readthedocs.org/projects/marketplace/?badge=latest)

# Marketplace Frontend Docs

Developer documentation for Firefox Marketplace frontend projects. Projects
include:

- [Marketplace Frontend](http://github.com/mozilla/fireplace)
- [Marketplace Template](http://github.com/mozilla/marketplace-template)
- [Marketplace Curation Tools](http://github.com/mozilla/transonic)
- [Marketplace Operator Dashboard](http://github.com/mozilla/marketplace-operator-dashboard)
- [Marketplace Statistics](http://github.com/mozilla/marketplace-stats)

See also the
[high-level developer documentation for the Firefox Marketplace](https://marketplace.readthedocs.org/).


# Building the Documentation

To prepare to build this documentation:

    mkvirtualenv marketplace-docs
    pip install -r requirements/dev.txt

To build this documentation:

    make docs

To have changes built as they happen:

    make watch
