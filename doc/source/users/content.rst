-----------------
Provided Software
-----------------

The RCP is providing following:

* FOSS software
* RCP software
* Development tools
* VM QCOW2 base image
* Container base images
* Containerized services
* LXC containerized services
* Kubernetes services

^^^^^^^^^^^^^
FOSS software
^^^^^^^^^^^^^

The RCP is providing FOSS software in pre-build binary format though its
Linux distribution WadersOS. Similarly as to other Linux distributions
such as Fedora, RedHat, OpenSUSE etc.

All the provided FOSS software complies to :xref:`Nokia DFSEC` requirements.

Complete list of provided FOSS packages can be found on :xref:`WadersOS build server`.

The FOSS packages are built using `rpmbuild` based on :xref:`WadersOS SPECS`,
the actual source code archives are stored on :xref:`WadersOS source lookaside cache`.

The source code and patches can be also obtained from SRPMs.

^^^^^^^^^^^^
RCP Software
^^^^^^^^^^^^

List of all RCP own software can be obtained from :xref:`RCP build server`.
The packages are built on top of WadersOS using :xref:`RCP SPECS`.

The actual source code is stored either in Gitlab or Gerrit. The easiest
way to get the source code is to use :xref:`conde` tool which provides
interface to RCP SCM.

^^^^^^^^^^^^^^^^^
Development tools
^^^^^^^^^^^^^^^^^

The RCP provides all necessary development tools including compilers,
Linux utilities, interpreters etc. All the tools are provided for the
native use inside RCP images.

The WadersOS also provides `gcc` and `protobuf` cross-compilers. These
are used by legacy application build systems based on cross-compilation.
For more information about the cross-compilation see :xref:`WadersOS crosscompilers`.


^^^^^^^^^^^^^^^^^^^
VM QCOW2 base image
^^^^^^^^^^^^^^^^^^^

For OpenStack based products such as ASRNC, RCP provides QCOW2 image
with full OS and RCP services. The image can be used both as base
for the build system as well as as base for the run-time image.


^^^^^^^^^^^^^^^^^^^^^
Container base images
^^^^^^^^^^^^^^^^^^^^^

The RCP provides set of base images which can be used whether for creating
a build and development environments or as base for a run-time product
images.


^^^^^^^^^^^^^^^^^^^^^^
Containerized services
^^^^^^^^^^^^^^^^^^^^^^

The RCP provides set of containerized services ready to be integrated
in Kubernetes based applications.


^^^^^^^^^^^^^^^^^^^^^^^^^^
LXC containerized services
^^^^^^^^^^^^^^^^^^^^^^^^^^

For ClassicalBTS RCP provides containerized services delivered in LXC containers.


^^^^^^^^^^^^^^^^^^^
Kubernetes services
^^^^^^^^^^^^^^^^^^^

The RCP provides fully deploy-able Kubernetes services consisting of Helm
or operator based deployments with one or several Pods containing one or
several containers.
