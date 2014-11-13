[![Documentation Status](https://readthedocs.org/projects/marketplace/badge/?version=latest)](https://readthedocs.org/projects/marketplace-frontend/?badge=latest)

Developer documentation for Firefox Marketplace frontend projects. Hosted at
[marketplace-frontend.readthedocs.org](https://marketplace-frontend.readthedocs.org).

Other Marketplace documentation repositories include:

- [Marketplace high-level documentation](https://github.com/mozilla/marketplace-docs).
- [Marketplace API documentation](https://github.com/mozilla/zamboni/tree/docs/api).


Marketplace frontend projects include:

- [Marketplace Frontend](http://github.com/mozilla/fireplace)
- [Marketplace Template](http://github.com/mozilla/marketplace-template)
- [Marketplace Curation Tools](http://github.com/mozilla/transonic)
- [Marketplace Communication Dashboard](http://github.com/mozilla/commbadge)
- [Marketplace Operator Dashboard](http://github.com/mozilla/marketplace-operator-dashboard)
- [Marketplace Statistics](http://github.com/mozilla/marketplace-stats)


# Building the Documentation

To prepare to build this documentation:

    mkvirtualenv marketplace-docs
    pip install -r requirements/dev.txt

To build this documentation:

    make docs

To have changes built as they happen:

    make watch
