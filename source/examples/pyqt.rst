=================
PyQt5 application
=================

Packaging a Python3 application into an AppImage is quite similar to packaging a regular
compiled application. The trick consist on embedding the python interpreter along with
the application code.

Requirements
------------
- modern Debian/Ubuntu system
- python3 and pip
- appimage-builder installed
- apt-get

Instructions
------------
0. Use the recipe below as template
1. Copy the application code into AppDir/usr/src
2. Copy the application icon to  AppDir/usr/share/icons/hicolor/256x256/apps/
3. Install the application requirements using pip:
    `python3 -m pip install --system --ignore-installed --prefix=/usr --root=AppDir -r ./requirements.txt`
4. Setup the **PYTHONHOME** and **PYTHONPATH** environment variables
5. Run `appimage-builder`

**The complete example source code can be found** `here <https://github.com/AppImageCrafters/appimage-builder/tree/master/examples/pyqt5>`_.

Recipe
------
.. code-block:: yaml

    version: 1
    script:
      # Remove any previous build
      - rm -rf AppDir  | true
      # Make usr and icons dirs
      - mkdir -p AppDir/usr/src
      # Copy the python application code into the AppDir
      - cp main.py  AppDir/usr/src -r
      # Install application dependencies
      - python3 -m pip install --system --ignore-installed --prefix=/usr --root=AppDir -r ./requirements.txt


    AppDir:
      path: ./AppDir

      app_info:
        id: org.appimage-crafters.python-appimage-example
        name: python appimage hello world
        icon: utilities-terminal
        version: 0.1.0
        # Set the python executable as entry point
        exec: usr/bin/python3
        # Set the application main script path as argument. Use '$@' to forward CLI parameters
        exec_args: "$APPDIR/usr/src/main.py $@"

      apt:
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
          - python3
          - python3-pkg-resources
          - python3-pyqt5
        exclude: []

      runtime:
        env:
          # Set python home
          # See https://docs.python.org/3/using/cmdline.html#envvar-PYTHONHOME
          PYTHONHOME: '${APPDIR}/usr'
          # Path to the site-packages dir or other modules dirs
          # See https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH
          PYTHONPATH: '${APPDIR}/usr/lib/python3.6/site-packages'

      test:
        fedora:
          image: appimagecrafters/tests-env:fedora-30
          command: ./AppRun
          use_host_x: true
        debian:
          image: appimagecrafters/tests-env:debian-stable
          command: ./AppRun
          use_host_x: true
        arch:
          image: appimagecrafters/tests-env:archlinux-latest
          command: ./AppRun
          use_host_x: true
        centos:
          image: appimagecrafters/tests-env:centos-7
          command: ./AppRun
          use_host_x: true
        ubuntu:
          image: appimagecrafters/tests-env:ubuntu-xenial
          command: ./AppRun
          use_host_x: true

    AppImage:
      update-information: 'gh-releases-zsync|AppImageCrafters|python-appimage-example|latest|python-appimage-*x86_64.AppImage.zsync'
      sign-key: None
      arch: x86_64


Tips/Tricks
-----------

Resolving python versions
=========================

In some scenarios a fixed python version may be required. If this version is not included in your default repository you may find
it in others such as:

- the `deadsnakes ppa`_ for Ubuntu

.. _`deadsnakes ppa`: https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa


Installing dependencies using the embed python
==============================================

If you are embedding a python version different from the one in your system the `pip install` command will fail to resolve and
install the right packages (it will install the packages for the python version in your system). To workaround this issue you
will have to use the python in the bundle.

To use the bundled python binary we will move the `pip install command` from the main script section to the 'after_bundle' section.
There we will also need to `configure the python home, paths`_ and provably install pip. In the following snippet you will find an example:

.. _`configure the python home, paths`: https://docs.python.org/es/3/using/cmdline.html?highlight=pythonhome#environment-variables

.. code-block:: yaml

  AppDir:

    after_bundle: |
    # Set python 3.9 env
    export PYTHONHOME=${APPDIR}/usr
    export PYTHONPATH=${APPDIR}/usr/lib/python3.9/site-packages:$APPDIR/usr/lib/python3.9
    export PATH=${APPDIR}/usr/bin:$PATH
    # Set python 3.9 as default
    ln -fs python3.9 $APPDIR/usr/bin/python3
    # Install pip
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    python3.9 get-pip.py
    # Install pipenv
    python3.9 -m pip install pipenv
    # Generate the requirements.txt file
    python3.9 -m pipenv lock -r > requirements.txt
    # Install application dependencies in AppDir
    python3.9 -m pip install --upgrade --isolated --no-input --ignore-installed --prefix=$APPDIR/usr wheel
    python3.9 -m pip install --upgrade --isolated --no-input --ignore-installed --prefix=$APPDIR/usr -r ./requirements.txt


SSL Certificates
================

Sadly in the GNU/Linux world the SSL certificates are not stored in a fixed location, therefore if we include
libssl.so in our bundle it may not be able to find the certificates in some distributions. This is issue is
discussed in detail in the `probono Linux Platform Issues`_ talk. To work around it we could embed our own copy of the certificates.

.. _probono Linux Platform Issues: https://gitlab.com/probono/platformissues/-/blob/master/README.md#certificates

The `certifi` python package give us a curated collection of Root Certificates that we can embed. It can be
installed using pip o the `python3-certifi` package from Debian and Ubuntu repositories.

Additionally you will have to set the SSL_CERT_FILE environment pointing to the `cacert.pem` file.


.. code-block:: yaml

      runtime:
        env:
          # Set python home
          # See https://docs.python.org/3/using/cmdline.html#envvar-PYTHONHOME
          PYTHONHOME: '${APPDIR}/usr'
          # Path to the site-packages dir or other modules dirs
          # See https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH
          PYTHONPATH: '${APPDIR}/usr/lib/python3.8/site-packages'
          # SSL Certificates are placed in a different location for every system therefore we ship our own copy
          SSL_CERT_FILE: '${APPDIR}/usr/lib/python3.8/site-packages/certifi/cacert.pem'