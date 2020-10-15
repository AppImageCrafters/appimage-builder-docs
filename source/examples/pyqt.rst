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
          PATH: '${APPDIR}/usr/bin:${PATH}'
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
