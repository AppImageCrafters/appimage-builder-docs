===================
Path Mapping (GIMP)
===================

In this recipe you will learn how to map file paths in order to work-around fixed paths in compiled binaries. In the
process we will create an AppImage for the GNU Image Manipulation Program.

For this example we will used the binaries from the Gimp deb package, but if you can also build your own binaries
from source.

Annotations
-----------

Gimp is a huge and complex project. It needs python, gtk, BABL, GEGL, fontsconfig and many other components therefore
we must properly setup everyone for behaving well in a portable environment. This setup is made using the environment
variables that those pieces of software use in the *runtime* > *env* section.

- PYTHON: requires PYTHONPATH_
- GTK: requires GTK_PATH_ , GTK_EXE_PREFIX, GTK_DATA_PREFIX
- GDK-Pixbuf: requires the loaders path to be present in the LIBRARY_PATH
- GIMP: requires BABL_PATH, GEGL_PATH, GIMP2_LOCALEDIR

.. _GTK_PATH: https://www.geany.org/manual/gtk/gtk/gtk-running.html
.. _PYTHONPATH: https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH

Besides the above mentioned configuration we still need to make Gimp able to find it's configuration and data files.
Those files are usually installed to */etc/gimp/2.0/* and */usr/share/gimp/2.0* but those paths are hardcoded on build.
To make them available we will use the *runtime* > *path_mappings* feature as follows:

.. code-block:: yaml

      runtime:
        path_mappings:
          - /etc/gimp/2.0/:$APPDIR/etc/gimp/2.0/
          - /usr/share/gimp/2.0/:$APPDIR/usr/share/gimp/2.0/

At runtime the Gimp binary will be deceived to access *$APPDIR/etc/gimp/2.0/* instead of */etc/gimp/2.0/* and
*/usr/share/gimp/2.0/* instead of *$APPDIR/usr/share/gimp/2.0/*.

Recipe
------

.. code-block:: yaml

    version: 1

    AppDir:
      path: ./AppDir

      app_info:
        id: gimp
        name: GNU Image Manipulation Program
        icon: gimp
        version: latest
        exec: usr/bin/gimp

      apt:
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
          - gimp
          - libgtk2.0-0
          - librsvg2-common
          - libjson-glib-1.0-0
          - liblcms2-2
          - libgexiv2-2
          - python2.7
          - libgirepository-1.0-1
          - librsvg2-2
          - librsvg2-bin
          - graphviz
          - libamd2
          - libbtf1
          - libcamd2
          - libccolamd2
          - libcholmod3
          - libcolamd2
          - libcxsparse3
          - libgexiv2-2
          - libgirepository-1.0-1
          - libglib2.0-0
          - libglu1-mesa
          - libgs9
          - libjpeg8
          - libjson-glib-1.0-0
          - libklu1
          - liblapack3
          - liblcms2-2
          - libldl2
          - liblensfun1
          - libpango-1.0-0
          - libpangocairo-1.0-0
          - libpng16-16
          - librbio2
          - librsvg2-2
          - libsdl1.2debian
          - libspqr2
          - libsuitesparseconfig5
          - libtiff5
          - libumfpack5
          - libv4l-0
          - libwebp6
          - python-gobject-2
          - libwebpmux3
          - libwebpdemux2
          - libpoppler-glib8
          - libmng2
          - libfreetype6
          - libfontconfig1
          - libblas3
          - libpulse0

        exclude:
          - adwaita-icon-theme
          - humanity-icon-theme

      files:
        exclude:
          - usr/lib/x86_64-linux-gnu/gconv
          - usr/share/man
          - usr/share/doc/*/README.*
          - usr/share/doc/*/changelog.*
          - usr/share/doc/*/NEWS.*
          - usr/share/doc/*/TODO.*
          - usr/include
      runtime:
        path_mappings:
          - /etc/gimp/2.0/:$APPDIR/etc/gimp/2.0/
          - /usr/share/gimp/2.0/:$APPDIR/usr/share/gimp/2.0/
        env:
          APPDIR_LIBRARY_PATH: '$APPDIR/usr/lib/x86_64-linux-gnu:$APPDIR/lib/x86_64-linux-gnu:$APPDIR/usr/lib:$APPDIR/usr/lib/x86_64-linux-gnu/gdk-pixbuf-2.0/2.10.0/loaders'
          GTK_EXE_PREFIX: $APPDIR/usr
          GIMP2_LOCALEDIR: $APPDIR/usr/share/locale
          PYTHONPATH: $APPDIR/usr/lib/python2.7:$APPDIR/usr/lib/python2.7/site-packages:$PYTHONPATH
          GTK_PATH: $APPDIR/lib/gtk-2.0
          GTK_DATA_PREFIX: $APPDIR
          XDG_DATA_DIRS: $APPDIR/share:$XDG_DATA_DIRS
          BABL_PATH: $APPDIR/usr/lib/x86_64-linux-gnu/babl-0.1
          GEGL_PATH: $APPDIR/usr/lib/x86_64-linux-gnu/gegl-0.4
      test:
        debian:
          image: appimagecrafters/tests-env:debian-stable
          command: "./AppRun"
          use_host_x: True
        centos:
          image: appimagecrafters/tests-env:centos-7
          command: "./AppRun"
          use_host_x: True
        fedora:
          image: appimagecrafters/tests-env:fedora-30
          command: "./AppRun"
          use_host_x: True
        ubuntu:
          image: appimagecrafters/tests-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True

    AppImage:
      arch: x86_64
