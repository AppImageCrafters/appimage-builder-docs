.. _advanced-setup-helpers:

"""""""""""""
Setup Helpers
"""""""""""""


appimage-builder includes support for making applications created with several technologies portable. It patches bundle
files and define environment variables so every binary behaves as expected.

Here you will fin a list of those with the features they support.

=========================
The GNU C Library (glibc)
=========================

`glibc` is at the core of almost every GNU/Linux binary and is responsible for dynamic linking and other start related
processes. `appimage-builder` makes use of the `AppRun`_ project to deal with the complexities of runtime setup.

The most significant issue is that AppImage must be always ran with the newest version of the glibc. To workaround
this issue appimage-builder setups two runtimes according to the `AppRun specifications`_. At runtime AppRun will
select the right one.

See the `AppRun usage instructions`_ for more details.

.. _AppRun: https://github.com/appimagecrafters/AppRun
.. _AppRun specifications: https://github.com/AppImageCrafters/AppRun/blob/master/docs/USAGE.md
.. _AppRun usage instructions: https://github.com/AppImageCrafters/AppRun/blob/master/docs/USAGE.md

==========
MIME TYPES
==========

In case custom mime-types are include the `update-mime-database` command will be executed to generate the respective
cache file.


=========
GStreamer
=========

GStreamer is a library for constructing graphs of media-handling components. When included appimage-builder will define
the following environment variables:

- GST_PLUGIN_PATH
- GST_PLUGIN_SYSTEM_PATH
- GST_REGISTRY_REUSE_PLUGIN_SCANNER
- GST_PLUGIN_SCANNER
- GST_PTP_HELPER
- GST_REGISTRY

It will also run gst "diagnostic" to force registry generation.

.. note:: You can find more detail about running gstreamer applications at: https://gstreamer.freedesktop.org/documentation

============
glib/gdk/gtk
============

When bundled appimage-builder the following environment variables:

- GIO_MODULE_DIR
- GI_TYPELIB_PATH
- GSETTINGS_SCHEMA_DIR
- GDK_PIXBUF_MODULEDIR
- GDK_PIXBUF_MODULE_FILE
- GTK_EXE_PREFIX
- GTK_DATA_PREFIX
- GTK_PATH

Additionally it will run:

- `glib-compile-schemas`: to ensure that schema files inside the bundle get compiled
- `gdk-pixbuf-query-loaders`: to build loaders cache file. File paths will be patched
- `gtk-update-icon-cache`: to generate the icon theme cache files


.. note:: See https://docs.gtk.org/glib/running.html for more details about running Glib/Gtk applications.


===
Qt5
===

Qt5 is a cross-platform software development framework. When included in the bundle the tool will create a `qt.conf`
file with the proper configuration next to each executable to ensure they will be able to find their resources at
runtime.

.. note:: You can read more about the qt.conf file at: https://doc.qt.io/qt-5/qt-conf.html

====
Java
====

Java is a programing language and development platform. When included in a bundle the `JAVA_HOME` environment will
be configured to the parent dir where the `java` binary is found.

======
Python
======

Python is a high-level, interpreted, general-purpose programming language. When included in the bundle the `PYTHONHOME`
will be set to the parent dir where the `python` binary is found.

.. note:: In cases when multiple versions of python are bundled you will have to manually define this variable.

=======
OpenSSL
=======

When `libopenssl` is embed the `OPENSSL_ENGINES` environment variable will be set to the proper location in the bundle.

You chan check the generated `AppRun.env` file to verify it's being set properly.


=====
LibGL
=====

`libGL` requires a set of drivers to work properly. Those are usually located at `<libs path>/dri/`. This
path will be set to the environment variable `LIBGL_DRIVERS_PATH`_

.. note:: Embedding libGL and family is discouraged as your app will not be able to use your system drivers.

.. _LIBGL_DRIVERS_PATH: https://dri.freedesktop.org/wiki/libGL/