How to make a psycopg2 release
==============================

- Edit ``setup.py`` and set a stable version release. Use PEP 440 to choose
  version numbers, e.g.

  - ``2.7``: a new major release, new features
  - ``2.7.1``: a bugfix release
  - ``2.7.1.1``: a release to fix packaging problems
  - ``2.7.2.dev0``: version held during development, non-public test packages...
  - ``2.8b1``: a beta for public tests

  In the rest of this document we assume you have exported the version number
  into an environment variable, e.g.::

    $ export VERSION=2.7

- In the `Travis settings`__ you may want to be sure that the variables
  ``TEST_PAST`` and ``TEST_FUTURE`` are set to a nonzero string to check all
  the supported postgres version.

.. __: https://travis-ci.org/psycopg/psycopg2/settings

- Push psycopg2 to master or to the maint branch. Make sure tests on Travis__
  and AppVeyor__ pass.

.. __: https://travis-ci.org/psycopg/psycopg2
.. __: https://ci.appveyor.com/project/psycopg/psycopg2

- For an extra test merge or rebase the `test_i686`__ branch on the commit to
  release and push it too: this will test with Python 32 bits and debug
  versions.

.. __: https://github.com/psycopg/psycopg2/tree/test_i686

- Create a signed tag with the content of the relevant NEWS bit and push it.
  E.g.::

    $ git tag -a -s $VERSION

    Psycopg 2.7 released

    What's new in psycopg 2.7
    -------------------------

    New features:

    - Added `~psycopg2.sql` module to generate SQL dynamically (:ticket:`#308`).
    ...

- Update the `psycopg2-wheels`_ submodule to the tag version::

    $ cd ./psycopg2
    $ git fetch
    $ git reset --hard $VERSION
    $ cd ..
    $ git add psycopg2
    $ git commit -m "releasing psycopg $VERSION"

- Create a tag called like the version and and push it::

    $ git tag $VERSION
    $ git push --tags
  
  This will create artifacts on `Travis CI`__ and `AppVeyor`__ which will be
  uploaded on GitHub at
  https://github.com/psycopg/psycopg2-wheels/releases/tag/$VERSION.

.. _psycopg2-wheels: https://github.com/psycopg/psycopg2-wheels
.. __: https://travis-ci.org/psycopg/psycopg2-wheels
.. __: https://ci.appveyor.com/project/psycopg/psycopg2-wheels

- Download the packages generated (this assumes ssh configured properly)::

    $ wget --quiet -O - "https://api.github.com/repos/psycopg/psycopg2-wheels/releases/tags/$VERSION" \
        | jq -r '.assets | .[] | .browser_download_url'
        | xargs wget -P psycopg2-$VERSION

- Sign the packages::

    $ for f in psycopg2-${VERSION}/*.{exe,tar.gz,whl}; do \
        gpg --armor --detach-sign $f;
      done

- Remove the ``.exe`` from the dir, because we don't want to upload them on
  PyPI::

    $ rm -v psycopg2-${VERSION}/*.exe{,.asc}

- Only for stable packages: upload the packages and signatures on PyPI::

    $ twine upload psycopg2-${VERSION}/*

- Create a release and release notes in the psycopg website, announce to
  psycopg and pgsql-announce mailing lists.

- Edit ``setup.py`` changing the version again (e.g. go to ``2.7.1.dev0``).


Releasing test packages
-----------------------

Test packages may be uploaded on the `PyPI testing site`__ using::

    $ twine upload -r testpypi psycopg2-${VERSION}/*

assuming `proper configuration`__ of ``~/.pypirc``.

.. __: https://test.pypi.org/project/psycopg2/
.. __: https://wiki.python.org/moin/TestPyPI
