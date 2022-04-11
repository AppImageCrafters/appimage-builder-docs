.. _faq:

""""""""""""""""""""""""
Frequent Asked Questions
""""""""""""""""""""""""

What kind of application I can pack as AppImage?
================================================

In theory every kind of application no matter the technology used to build it. But some of then are a bit complex.
Join our :ref:`community <community>` to get some help.

What systems are supported?
===========================

- Debian
- Ubuntu
- Arch
 
Other will be added in the future.

What makes appimage-builder different?
=====================================

appimage-builder uses a new approach for creating AppImages which is based on:

- bundling almost every dependency inside the AppImage (including glibc and family)
- selecting at runtime the newer glibc version to be used while running the bundled
  app using a custom AppRun
- excluding by default drivers and using the system ones at runtime
- restoring the environment variables while calling external binaries using function hooks
- patching paths on function calls at runtime using function hooks

This allows creating backward and forward compatible bundles with little effort
if compared to other existent solutions where the developer has to setup or
tweak a build environment and finding/making backports of their app dependencies.

But this approach also has drawbacks, bundling everything means:

- AppImage will be at least 30Mb bigger
- critical software such as libssl will be frozen into the bundle

When should I use appimage-builder?
===================================

- You require using cutting edge technologies that cannot be found on the oldest
  and still maintained LTS distribution
- You depend on binaries with fixed paths in their code
- You want to do a cross-build

In general it's quite safe to use appimage-builder as long as you know the implications.
Below you will find some recommendations to mitigate those issues.


When should I **NOT** use appimage-builder?
===========================================

- The target application can be built on the oldest and still maintained LTS
  distribution.

You will get an smaller bundle that will require less updates using other
AppImage creation tools such as `linuxdeploy`_

.. _linuxdeploy: https://github.com/linuxdeploy/



Where can I ask more about appimage-builder?
============================================

In the `appimage-builder github project`_ or in the :ref:`community` spaces.


.. _appimage-builder github project: https://github.com/AppImageCrafters/appimage-builder