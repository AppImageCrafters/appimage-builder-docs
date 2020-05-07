.. _hosted-services-github-actions:

"""""""""""""""""""""""""""""
Producing AppImages on GitHub
"""""""""""""""""""""""""""""


Build AppImage Action
=====================

To produce an AppImage on GitHub use the `build AppImage Action`_. This will run ``appimage-builder`` in
an Ubuntu container and will output the AppImage file in the current working directory.

.. _build AppImage Action: https://github.com/marketplace/actions/build-appimage

It takes for input the path to the ``appimage-builder`` recipe file and outputs the paths to the AppImage and
the zsync file.

**NOTE**: Use the same sources lists for the recipe and the build system, otherwise resulting bundle may be faulty.

Update Information
==================

AppImage Update support fetching updates from GitHub releases. To enable it se the following update information
in your recipe file:

``gh-releases-zsync|<user>|<project>|latest|*.AppImage.zsync```.

Replace ``<user>`` by the GitHub user or organization name and ``project`` buy the project name.


Workflow example
================

A complete example project can be found at: https://github.com/AppImageCrafters/qt-hello-world

.. code-block:: yaml

    name: C/C++ AppImage

    on:
      push:
        branches: [ master ]
      pull_request:
        branches: [ master ]

    jobs:
      build-appimage:

        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v2
          - name: install dependencies
            run: |
              sudo apt-get update
              sudo apt-get install -y qt5-default qtdeclarative5-dev cmake
          - name: configure
            run: cmake . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
          - name: build
            run: make -j`nproc`  install DESTDIR=AppDir
          - name: Build AppImage
            uses: AppImageCrafters/build-appimage-action@master
            env:
              UPDATE_INFO: gh-releases-zsync|AppImageCrafters|qt-hello-world|latest|*x86_64.AppImage.zsync
            with:
              recipe: AppImageBuilder.yml
          - uses: actions/upload-artifact@v2
            with:
              name: AppImage