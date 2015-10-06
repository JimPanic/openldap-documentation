.. $OpenLDAP$
.. Copyright 1999-2015 The OpenLDAP Foundation, All Rights Reserved.
.. COPYING RESTRICTIONS APPLY, see COPYRIGHT.

===========
Quick Start
===========

The following is a quick start guide to OpenLDAP, including the Standalone
:term:`LDAP` Daemon, *slapd* (8).

It is meant to walk you through the basic steps needed to install and configure
`OpenLDAP Software`_.  It should be used in conjunction with the other
chapters of this document, manual pages, and other materials provided with the
distribution (e.g. the `INSTALL`_ document) or on the `OpenLDAP`_ web
site, in particular the OpenLDAP Software
:term:`FAQ` (http://www.openldap.org/faq/?file=2).

.. _OpenLDAP Software: http://www.openldap.org/software/
.. _INSTALL: https://github.com/openldap/openldap/blob/master/INSTALL
.. _OpenLDAP: http://www.openldap.org/

If you intend to run OpenLDAP Software seriously, you should review all of this
document before attempting to install the software.

.. note::

  This quick start guide does not use strong authentication nor any integrity
  or confidential protection services.  These services are described in other
  chapters of the OpenLDAP Administrator's Guide.

Install from Source
===================

.. rubric:: Get the software

You can obtain a copy of the software by following the instructions on the
OpenLDAP Software download page
(http://www.openldap.org/software/download/).  It is recommended that new
users start with the latest *release*.

.. rubric:: Unpack the distribution

Pick a directory for the source to live under, change directory to there, and
unpack the distribution using the following commands::

  gunzip -c openldap-VERSION.tgz | tar xvfB -

then relocate yourself into the distribution directory::

  cd openldap-VERSION

You'll have to replace ``VERSION`` with the version name of the release.

.. rubric:: Review documentation

You should now review the ``COPYRIGHT``, ``LICENSE``, ``README`` and
``INSTALL`` documents provided with the distribution.  The ``COPYRIGHT`` and
``LICENSE`` provide information on acceptable use, copying, and limitation of
warranty of OpenLDAP Software. 

You should also review other chapters of this document.  In particular the
:doc:`Building and Installing OpenLDAP Software <build-install>` chapter
provides detailed information on prerequisite software and installation
procedures.

.. rubric:: Run ``configure``

You will need to run the provided ``configure`` script to *configure* the
distribution for building on your system. ``configure`` script accepts many
command line options that enable or disable optional software features.
Usually the defaults are okay, but you may want to change them.  To get a
complete list of options that ``configure`` accepts, use the ``--help``
option::

  ./configure --help

However, given that you are using this guide, we'll assume you are brave enough
to just let *configure* determine what's best::

  ./configure

Assuming ``configure`` doesn't dislike your system, you can proceed with
building the software.  If ``configure`` did complain, well, you'll likely need
to go to the `Software FAQ Installation section
<http://www.openldap.org/faq/?file=8>`_) and/or read the :doc:`Building and
Installing OpenLDAP Software <build-install>` chapter.

.. rubric:: Build the software

The next step is to build the software.  This step has two parts, first we
construct dependencies and then we compile the software::

  make depend
  make

Both invocations of ``make`` should complete without error.

.. rubric:: Test the build

To ensure a correct build, you should run the test suite (it only takes a few
minutes)::

  make test

Tests which apply to your configuration will run and they should pass.  Some
tests, such as the replication test, may be skipped.

.. rubric:: Install the software

You are now ready to install the software; this usually requires *super-user*
privileges::

  su root -c 'make install'

Everything should now be installed under ``/usr/local`` (or whatever
installation prefix was used by ``configure``).

.. rubric:: Edit the configuration file

Use your favorite editor to edit the provided ``slapd.ldif`` example (usually
installed as ``/usr/local/etc/openldap/slapd.ldif``) to contain a MDB database
definition of the form::

  dn: olcDatabase=mdb,cn=config
  objectClass: olcDatabaseConfig
  objectClass: olcMdbConfig
  olcDatabase: mdb
  OlcDbMaxSize: 1073741824
  olcSuffix: dc=<MY-DOMAIN>,dc=<COM>
  olcRootDN: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
  olcRootPW: secret
  olcDbDirectory: /usr/local/var/openldap-data
  olcDbIndex: objectClass eq

Be sure to replace ``<MY-DOMAIN>`` and ``<COM>`` with the appropriate domain
components of your domain name.  For example, for ``example.com``, use::

  dn: olcDatabase=mdb,cn=config
  objectClass: olcDatabaseConfig
  objectClass: olcMdbConfig
  olcDatabase: mdb
  OlcDbMaxSize: 1073741824
  olcSuffix: dc=example,dc=com
  olcRootDN: cn=Manager,dc=example,dc=com
  olcRootPW: secret
  olcDbDirectory: /usr/local/var/openldap-data
  olcDbIndex: objectClass eq

If your domain contains additional components, such as
``eng.uni.edu.eu``, use::

  dn: olcDatabase=mdb,cn=config
  objectClass: olcDatabaseConfig
  objectClass: olcMdbConfig
  olcDatabase: mdb
  OlcDbMaxSize: 1073741824
  olcSuffix: dc=eng,dc=uni,dc=edu,dc=eu
  olcRootDN: cn=Manager,dc=eng,dc=uni,dc=edu,dc=eu
  olcRootPW: secret
  olcDbDirectory: /usr/local/var/openldap-data
  olcDbIndex: objectClass eq

Details regarding configuring *slapd* (8) can be found in the *slapd-config*
(5) manual page and the :doc:`Configuring slapd <configuration>` chapter.  Note
that the specified ``olcDbDirectory`` must exist prior to starting *slapd* (8).

.. rubric:: Import the configuration database

You are now ready to import your configration database for use by *slapd* (8),
by running the command::

  su root -c /usr/local/sbin/slapadd -F /usr/local/etc/cn=config -l /usr/local/etc/openldap/slapd.ldif

.. rubric:: Start SLAPD

You are now ready to start the Standalone LDAP Daemon, *slapd* (8), by running
the command::

  su root -c /usr/local/libexec/slapd -F /usr/local/etc/cn=config

To check to see if the server is running and configured correctly, you can run
a search against it with *ldapsearch* (1).  By default, *ldapsearch* is
installed as ``/usr/local/bin/ldapsearch``::

  ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts

Note the use of single quotes around command parameters to prevent special
characters from being interpreted by the shell.  This should return::

  dn:
  namingContexts: dc=example,dc=com

Details regarding running *slapd* (8) can be found in the *slapd* (8) manual
page and the :doc:`Running slapd <running-slapd>` chapter.

.. rubric:: Add initial entries to your directory

You can use *ldapadd* (1) to add entries to your LDAP directory.  *ldapadd*
expects input in :term:`LDIF` form.  We'll do it in two steps:

#. create an LDIF file
#. run *ldapadd*

Use your favorite editor and create an LDIF file that contains::

  dn: dc=<MY-DOMAIN>,dc=<COM>
  objectclass: dcObject
  objectclass: organization
  o: <MY ORGANIZATION>
  dc: <MY-DOMAIN>

  dn: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
  objectclass: organizationalRole
  cn: Manager

Be sure to replace ``<MY-DOMAIN>`` and ``<COM>`` with the appropriate domain
components of your domain name.  ``<MY ORGANIZATION>`` should be replaced
with the name of your organization.  When you cut and paste, be sure to trim
any leading and trailing whitespace from the example.

::

  dn: dc=example,dc=com
  objectclass: dcObject
  objectclass: organization
  o: Example Company
  dc: example

  dn: cn=Manager,dc=example,dc=com
  objectclass: organizationalRole
  cn: Manager

Now, you may run *ldapadd* (1) to insert these entries into
your directory.

::

  ldapadd -x -D "cn=Manager,dc=<MY-DOMAIN>,dc=<COM>" -W -f example.ldif

Be sure to replace ``<MY-DOMAIN>`` and ``<COM>`` with the appropriate domain
components of your domain name.  You will be prompted for the "``secret``"
specified in ``slapd.conf``.  For example, for ``example.com``, use::

  ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f example.ldif

where ``example.ldif`` is the file you created above.

Additional information regarding directory creation can be found in the
:doc:`Database Creation and Maintenance Tools <db-tools>` chapter.

.. rubric:: See if it works

Now we are ready to verify the added entries are in your directory.  You can
use any LDAP client to do this, but our example uses the *ldapsearch* (1) tool.
Remember to replace ``dc=example,dc=com`` with the correct values for your
site::

  ldapsearch -x -b 'dc=example,dc=com' '(objectclass=*)'

This command will search for and retrieve every entry in the database.

You are now ready to add more entries using *ldapadd* (1) or another LDAP
client, experiment with various configuration options, backend arrangements,
etc

Note that by default, the *slapd* (8) database grants *read access to
everybody* excepting the *super-user* (as specified by the ``rootdn``
configuration directive).  It is highly recommended that you establish controls
to restrict access to authorized users.  Access controls are discussed in the
:doc:`Access Control <access-control>` chapter.  You are also encouraged to
read the :doc:`Security Considerations <security>`, :doc:`Using SASL
<using-sasl>` and :doc:`Using TLS <using-tls>` sections.

The following chapters provide more detailed information on making, installing,
and running *slapd* (8).
