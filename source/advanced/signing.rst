.. _advanced-signing:

""""""""""""""""
AppImage Signing
""""""""""""""""

AppImage can be signed using the Open PGP standard. This ensures that the AppImage comes from the person who pretends
to be the author, and ensures that the file has not been tampered with.

==============
Key Generation
==============

To sign an AppImage file you will need a key. Use ``gpg2 --full-gen-key`` to generate a new one. You can also check
the `GnuPG documentation`_ to learn more about it.

.. _GnuPG documentation: https://www.gnupg.org/gph/en/manual/c14.html

=======
Signing
=======

To sign your the resulting AppImage using appimage-builder is enough with specifying the key id in the
``AppImage >> signature`` section as follows:


.. code-block:: yaml

    AppImage:
      sign-key: 69O7M4CCE10E0273853CDA121896X515CC81F0AD

=====================
Reading the signature
=====================

The check if the AppImage was properly signed execute it with the following option ``--appimage-signature`` it will
print the signature, if any, to the standard output.

.. code-block:: shell

    $ ./gnome-calculator-3.28.0-x86_64.AppImage --appimage-signature

    -----BEGIN PGP SIGNATURE-----

    iQIzBAABCgAdFiEEaafEzOEOAnOFPNoSGJblFcyB8KwFAl6g2B0ACgkQGJblFcyB
    8KzP6g//WnCjb2HLLJ7U3muPb53py8Y5uwes7wE5w8Xbhy+ed42W6jp48cBl4O2G
    cMaSJR8xH7yPvaLVWOIfDW6i7l3QUwtBfknLBdXrdlrhdMNzgXyQiKbwSgSfQcqi
    kdaX2xFiXIYUV8e5BBZcfmKoFLNy4Lqfm2q8TICxiNiEdJ4eX5UTjfHWijmGg/pQ
    yVNNNGGfhOboT71DNUiJTeffwgwbDIwHHPiuXvfRyB/h8qfIMTYv0GSM3lwNGUkO
    3lN4LRkdZM9t19ZLpvR3uXt5DWlV7i5Q2uIp46pEUGJPnnneO3wM+wzo8r0e9Pur
    Nm/KEA9UG7Lfk6ktlq+e1r2pPWtaPwZCk3A//afPBymmGJACzvbN/XitB++hE5nT
    RUyRDiFW7BWMx9mWbdXaLEfR1ZOAY5rR/QJA6bKC4IvPSvHWUwYMkJdQV+MTZ0JL
    vCao5EtP6FgM0+Hm4dYoSReMK+9IpzIeg8uf8fgcaHa9lMZVryeJzmiR1vx5zu7Z
    1DrDuMrqSyQ10wBBIKA7K8oJKOhrc+yNcCK8ldpYcDi4WVnvsb1ffKAKz8SdqI7/
    RXwkbISmSkloDXkTRlZKW7Kwkj4spJzUEsKsDwif9C4A3lGJw2xj4pAqHlLH1uq9
    u07mp5HT1wPtfoBFSXqX3MVLSzb6x4Qz1gzVXgWnhx5C5/K0L+8=
    =Eruk
    -----END PGP SIGNATURE-----
