.. _hosted-services-gitlab-ci:

""""""""""""""""""""""""""""""""
Producing AppImages on Gitlab CI
""""""""""""""""""""""""""""""""

``appimage-builder`` can be easily integrated into a gitlab-ci recipe. Here you will learn how to do it.

=============
Docker Images
=============

There is an ``appimage-builder`` docker image ready to be used on gitlab-ci you can find it `here`_. It's based on
Ubuntu 18.04 and brings all the dependencies required by the tool. The docker image name is:
``appimagecrafters/appimage-builder``.


.. _here: https://hub.docker.com/repository/docker/appimagecrafters/appimage-builder


======
Recipe
======

The usual approach while writing a gitlab-ci recipe for building AppImages is to use the `before_script`_ section
to install the application dependencies. In the `script`_ section we will do the project configuration, binary
building and the AppImage generation.

.. _before_script: https://docs.gitlab.com/ee/ci/yaml/#before_script
.. _script: https://docs.gitlab.com/ee/ci/yaml/#script

In the code snippet below you can find a complete ``gitlab-ci.yml`` recipe for building an AppImage for a
`Hello World Qt project`_ using the latest Qt from the `KDE Neon repositories`_

.. _Hello World Qt project: https://www.opencode.net/azubieta/qt-hello-world/
.. _KDE Neon repositories: http://archive.neon.kde.org/

.. code-block:: yaml

    appimage-amd64:
        image: appimagecrafters/appimage-builder
        before_script:
          # update appimage-builder (optional)
          - apt-get update
          - apt-get install -y git wget
          - pip3 install --upgrade git+https://www.opencode.net/azubieta/appimagecraft.git

          # app build requirements
          - echo 'deb http://archive.neon.kde.org/user/ bionic main' > /etc/apt/sources.list.d/neon.list
          - wget -qO - https://archive.neon.kde.org/public.key | apt-key add -
          - apt-get update
          - apt-get install -y qt5-default qtdeclarative5-dev cmake
        script:
          - cmake . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
          - make install DESTDIR=AppDir
          - appimage-builder --skip-test
        artifacts:
          paths:
            - '*.AppImage*'
          expire_in: 1 week

========
OpenCode
========

Any Gitlab instance can be used to host your AppImage builds but it's recommended to do it on `opencode.net`_. This
instance part of the OpenDesktop ecosystem were the `AppImageHub`_ project lives. This will allow to mark published
binaries as "OFFICIAL".

.. _opencode.net: https://www.opencode.net/
.. _AppImageHub: https://www.appimagehub.com/