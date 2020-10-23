.. _intro-when-using-aib:

"""""""""""""""""""""""""""""""""""""
When appimage-builder should be used?
"""""""""""""""""""""""""""""""""""""

appimage-builder uses a new approach for creating AppImages which is based on:

- bundling almost every dependency inside the AppImage (including glibc and family)
- selecting at runtime the newer glibc version to be used while running the bundled
  app using a custom AppRun
- excluding by default drivers and using the system ones at runtime
- restoring the environment variables while calling external binaries using function hooks
- patching paths on function calls at runtime using function hooks

This allows creating backward and forward compatible bundles whit little effort
if compared to other existent solutions where the developer has to setup or
tweak a build environment and finding/making backports of their app dependencies.

But this approach also has drawbacks, bundling everything means:

- AppImage will be at least 30Mb bigger
- critical software such as libssl will be frozen into the bundle

-------------------------------------------
When should I **NOT** use appimage-builder?
-------------------------------------------

- The target application can be built on the oldest and still maintained LTS
  distribution.

You will get an smaller bundle that will require less updates using other
AppImage creation tools such as `linuxdeploy`_

.. _linuxdeploy: https://appimage-builder.readthedocs.io/en/latest/

-----------------------------------
When should I use appimage-builder?
-----------------------------------

- You require using cutting edge technologies that cannot be found on the oldest
  and still maintained LTS distribution
- You depend on binaries with fixed paths in their code
- You want to do a cross-build

In general it's quite safe to use appimage-builder as long as you know the implications.
Below you will find some recommendations to mitigate those issues.

------------------------------------------
Issues mitigation and other considerations
------------------------------------------

1. Critical software frozen inside the bundle
---------------------------------------------

This is the most important issue on the approach used by appimage-builder. The application
author **must** take the responsibility of updating this software when required. Therefore,
it's recommended to:

- use a trusted binary source such as the Debian/Ubuntu repositories
- setup a continuous integration mechanism to recreate bundles when required
- set the update information on the AppImage bundles
- encourage/notify final users to update their bundles with regularity

2. Extra size
-------------

As libc and other libs that are traditionally excluded from AppImages are now bundled the
final size is about 30Mb more. Also, as the system package manager is used to resolve
dependencies some non-required software may be also deployed. It's recommend to inspect
the bundles manually after they are created. There will be a `.bundle` file in the
AppDir root with the information of the packages that were included.

To exclude some non-required pacakge use the exclude section of the :ref:`recipe_version_1_apt` instruction.

