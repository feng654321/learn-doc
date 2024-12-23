*********************
Plugins vs. libraries
*********************

.. contents:: :local:

Introduction
############

Developers often confuse plugins with shared libraries. They look the same, but
they are very different and should be treated differently.

Key differences
###############

Usage
*****

Plugins are loaded dynamically with ``dlopen(3)``. Plugins are not used for
linking.

Shared libraries are used for linking. Shared libraries are not opened
dynamically with ``dlopen(3)``.

Version number
**************

Plugins never have version number.
::

 /usr/libexec/genapi/libeeegenapiremotesyslog.so
 /usr/libexec/genapi/libooogenapiministarter.so
 /usr/libexec/genapi/libqqqgenapistdoutlogger.so

Shared libraries always have version number.
::

 /usr/lib64/libgenapi.so -> libgenapi.so.4.7.32
 /usr/lib64/libgenapi.so.4 -> libgenapi.so.4.7.32
 /usr/lib64/libgenapi.so.4.7.32
 /usr/lib64/libministarter.so -> libministarter.so.0.1.15
 /usr/lib64/libministarter.so.0 -> libministarter.so.0.1.15
 /usr/lib64/libministarter.so.0.1.15

Install directory
*****************

Plugins should **not** be installed to ``/usr/lib64`` or ``/opt/nokia/lib64``,
because they are not used for linking.

Shared libraries are always installed to ``/usr/lib64`` (or ``/opt/nokia/lib64``
in legacy code), because they are used for linking.

Building plugins with libtool
*****************************

Plugins can be built with ``libtool`` by appending
``-avoid-version -module -shared`` to ``la_LDFLAGS``.

Rpm spec file
*************

Plugins are **not** placed into ``*-libs`` sub-package.

Shared libraries are placed to ``*-devel`` and ``*-libs`` sub-packages as
follows:
::

 %files libs
 %{_libdir}/libministarter.so.*

 %files devel
 %{_libdir}/libministarter.so
