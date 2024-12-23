---------------
Getting Started
---------------

^^^^^^^^^^^^^
Prerequisites
^^^^^^^^^^^^^

The RCP is `Linux`_ platform intended for deployment under `virtualization`_.
Both `OpenStack`_ `hypervisor based virtualization`_ deployments as well as
`OCI`_ `OS-level virtualization`_ deployments in `Kubernetes`_ are supported.

Knowledge of Linux operating systems, cloud technologies and software development
for these technologies are prerequisites assumed to be known to the reader.

For working with RCP a Linux based system is required with OCI compliant
run-time such as `Docker`_ or `Podman`_ at minimum. For working with Kubernetes
deployment it is further recommended to have available K8s environment,
either cloud or private, such as `minikube`_. To deploy the K8s services
also `Helm`_ tool will be required. For working with virtual
machine deployments `KVM/QEMU`_ is recommended.

In general it is assumed that reader has own or shared Linux environments
and knowledge how to use them. Some guidelines relevant or useful for working
with RCP are presented in these guidelines. But not all details of various
Linux and cloud technologies are described in details.

.. _virtualization: https://en.wikipedia.org/wiki/Virtualization
.. _hypervisor based virtualization: https://en.wikipedia.org/wiki/Hypervisor
.. _OpenStack: https://www.openstack.org/
.. _OCI: https://opencontainers.org/
.. _OS-level virtualization: https://en.wikipedia.org/wiki/OS-level_virtualization
.. _Kubernetes: https://kubernetes.io/
.. _Linux: https://en.wikipedia.org/wiki/Linux
.. _Docker: https://www.docker.com/
.. _Podman: https://podman.io/
.. _KVM/QEMU: https://www.qemu.org/
.. _minikube: https://minikube.sigs.k8s.io/docs/start/
.. _Helm: https://helm.sh/


^^^^^^^^^^
Hello RCP!
^^^^^^^^^^

To run first `Hello RCP!` program on latest version of RCP, just pull the latest RCP
OCI container base image.

.. code-block:: bash

        export RCPIMAGE=rcp-docker-containers-local.artifactory-espoo1.int.net.nokia.com/rcp/rcp-base-origin
        docker pull $RCPIMAGE
        docker run --rm $RCPIMAGE echo 'Hello RCP!'


^^^^^^^^^^^^^^^^^
First application
^^^^^^^^^^^^^^^^^

The RCP is using `RPM package management`_ similarly as for example to RedHat,
Fedora or OpenSUSE. All `FOSS`_ as well as Nokia RCP proprietary software
can be reached through package management, both the pre-built binaries as
well as the source code. As RCP is using Fedora inspired `WadersOS`_ which
is Nokia own Linux based operating system, all the guidelines and instructions
available on Internet for Fedora will work on RCP out of the box.

**NOTE**: in RCP minimal base image there is `microdnf`_ package management
front-end available.

The RCP provides all the usual software development tools for C, C++, Python,
Perl, Go languages.

In order to build first C++ application try following:

.. code-block:: bash

        mkdir app
        cd app
        cat << EOF > app.cpp
        #include <iostream>
        int main() {
            std::cout << "Hello RCP app!" << std::endl;
            return 0;
        }
        EOF
        docker run --rm -t -v $PWD:/work $RCPIMAGE bash -c 'microdnf install gcc-g++; g++ -o /work/app /work/app.cpp'
        docker run --rm -v $PWD:/work $RCPIMAGE /work/app


.. _RPM package management: https://en.wikipedia.org/wiki/RPM_Package_Manager
.. _FOSS: https://en.wikipedia.org/wiki/Free_and_open-source_software
.. _WadersOS: https://waders-infra.gitlabe2-pages.ext.net.nokia.com/waders-docs/
.. _microdnf: https://github.com/rpm-software-management/microdnf
