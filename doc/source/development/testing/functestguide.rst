***********************************
System Component Testing Guidelines
***********************************

.. contents:: :local:
.. highlight:: text

This document aims to describe the test framework, keyword libraries and
architecture of test solution used in RCP container based system component
testing (SCT), when testing RCP containers and PODs.

1. Generic decisions
####################

- only Python 3 is supported, so all keyword libraries and tools should
  work with python3. For installing packages pip3 should be used.
- use Robot Framework version 6.1.1
- domain's test data is published as :code:`*-robottests.rpm` packages and
  installed to test container with microdnf

2. Development environment
##########################

The recommended environment for developing tests is cloud-development_
instance on EE Cloud, using single-node Kubernetes configuration.

.. _cloud-development: https://gitlabe2.ext.net.nokia.com/cloud-tools/cloud-development/-/blob/master/README.md

Container pipelines use similar environment, so one should be able to run all
tests 'locally', i.e. in own cloud-development instance. In order to save
pipeline resources it is recommended to test your changes in your own local
development environment and push changes only when local tests pass.

Additional playbooks for installing Mate desktop, VNC server and PyCharm
community are provided, see MATE_DESKTOP.md_ for details

.. _MATE_DESKTOP.md: https://gitlabe2.ext.net.nokia.com/cloud-tools/cloud-development/blob/mate-desktop/MATE_DESKTOP.md

3. Development workflow
#######################

Container and helm pipeline acts as final verification of changes before
merging to master branch (which should be always clean, so direct commits do
master shouldn't be allowed).

Even though every commit pushed to remote will trigger the pipeline,
we should not use the pipeline as a means of verification of every small
commit during development work. For that purpose we should use personal or
team's own cloud-development environment, avoiding frequent pushes to the
repository.

3.1. Developing on 'local' cloud-development env
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Shell scripts in your container pipeline repository's ci/ sub-directory are
used by the Jenkins pipeline and should be written so that it is easy
to run them manually in a 'local' shell.

The following workflow illustrates the steps to run tests in robottest
container (commands should be executed in a terminal on your
cloud-development instance):

- clone container repository
- create a feature branch for your changes
- run a clean-build-test cycle - tests should pass on master. This is done
  usually by executing the following commands:

.. code-block:: console

    $ ./ci/docker-clean.sh
    $ ./ci/docker-build.sh x86
    $ ./ci/docker-test.sh x86

- do your changes and repeat the clean-build-test cycle until tests pass
- commit and push changes to your remote feature branch - pipeline should be
  triggered automatically

3.2. Testing manually on OpenEdge NCIR CaaS environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The container/helm pipelines support running tests on OE19 NCIR or other CaaS
environments. This section describes how tests can be run manually in case
debugging is needed.

Running tests on a CaaS environment differs from the NESC/cloud-development
workflow due to the following:

- NCIR CaaS is a multi-node Kubernetes environment where only some of the nodes
  have access to external network, so images must be on-board to be usable for
  pods from every node
- amount of available CaaS environments is limited, so usage should be kept
  at minimum, i.e. building container images on this env should be avoided

The following workflow illustrates the manual process:

- push your changes to the container pipeline repository in order to build and
  publish container images to be used for tests on OE env.
- find out docker image repository and tag from pipeline logs - to be used in
  on-board commands
- on-board the images built by the pipeline. This involves pulling from
  Artifactory, tagging and pushing to the internal image registry
  ``registry.kube-system.svc.nokia.net:5555``.

  The following commands should be repeated for all the needed images,
  including the image used by robot test container:

.. code-block:: console

    sudo docker pull rcp-docker-containers-local.esisoj70.emea.nsn-net.net/<repo_name>/<img_name>:<tag>
    sudo docker tag rcp-docker-containers-local.esisoj70.emea.nsn-net.net/<repo_name>/<img_name>:<tag> <internal_registry>/<repo_name>/<img_name>:<tag>
    sudo docker push registry.kube-system.svc.nokia.net:5555/<repo_name>/<img_name>:<tag>

- clone container repository and check out the correct branch
- run tests by calling ./ci/docker-test.sh

Note:

  Use internal registry ``registry.kube-system.svc.nokia.net:5555`` for NCIR19
  and ``bcmt-registry:5000`` for NCS clouds. The pipelines provide this URL as
  environment variable 'LOCAL_REGISTRY', so that scripts don't need to hardcode
  it.

3.3. Publishing changes
~~~~~~~~~~~~~~~~~~~~~~~

When change is ready:

- push local commits to the remote feature branch - pipeline should be triggered automatically
- create merge request to master branch
- invite reviewers as described in :doc:`../reviews/reviewguidelines` section :ref:`How to use review groups`
- when changes are reviewed and all discussions solved merge the merge request
- create annotated tag, which then triggers delivery pipeline

4. Robot test containers
########################

Test containers should use robottest-base_ image as base.

.. _robottest-base: https://gitlabe2.ext.net.nokia.com/rcp/containers/robottest-base

The base image:

 - provides basic tools and common libraries needed to run tests
 - provides global pip configuration in `/etc/pip.conf`
 - maintains list of latest verified versions for commonly used robot libraries
   and their dependencies in `/robot/libraries/frozen-robot-deps.txt`
 - as small as possible to keep cloud resource usage at minimum
 - based on rcp-minimal
 - `Publish rcp-base-image`_ pipeline is releasing a new robottest-base image for
   every rcp-minimal image automatically with the same tag. This means
   that same base image tag can/should be used for building application and
   robottest containers.
 - uses dnf repository from its base so robottest rpms can be installed with
   microdnf install

.. _publish rcp-base-image: http://rcp-dev-pl-jenkins.eecloud.dynamic.nsn-net.net:8080/job/tools/job/publish-rcp-base-image/

The example below has argument ROBOT_BASE_IMAGE_VER set to 'NOT_DEFINED' - it
must be provided by the shell script calling docker build. This is done to make sure
the version is not left accidentally as 'latest' in build process.

Dockerfile example for robottest containers based on robottest-base > 2.0.0

.. code-block:: dockerfile
    :caption: Dockerfile

    ARG ROBOT_BASE_IMAGE=rcp-docker-containers-local.esisoj70.emea.nsn-net.net/robottest-base/robottest-base
    ARG ROBOT_BASE_IMAGE_VER=NOT_DEFINED

    FROM ${ROBOT_BASE_IMAGE}:${ROBOT_BASE_IMAGE_VER}

    RUN microdnf install my-robottests.rpm

    RUN pip install -c /robot/libraries/frozen-robot-deps.txt crl.k8sdeployer

4.1. Directory structure
~~~~~~~~~~~~~~~~~~~~~~~~

Test containers should use same directory structure as robottest-base container::

    /
    +-- opt
    |   +-- nokia
    |       +-- bin
    |           +-- start_robot.sh    <<< test statrup script
    +-- usr
    |   +-- local
    |       +-- lib
    |           +-- python3.7
    |               +-- site-packages
    |                   +-- crl       <<< pip3 installs CRL libraries here
    +-- robot
        +-- libraries      <<< common libraries and package constraint files
        +-- resources      <<< common resources
        +-- tests          <<< for test data and custom libraries
        |   +-- CCS        <<< for CCS domain test data (libraries, resources, test cases...)
        |   +-- <domainX>  <<< for domain X test data
        +-- logs           <<< for logs and results from test run

Currently robottest RPMs don't follow this structure. One solution is to make
them relocatable - i.e. installable to any directory, but microdnf install
doesn't support `--prefix` argument like rpm install does.

``/robot`` directory and its child directories ``libraries``, ``resources``, ``tests`` and ``logs``
are created by ``robot-dirs.rpm``, see `%install` section of robot-dirs.spec_ for details.
This package is installed into robottest-base_ container image, so all containers based on
it will inherit the directory structure.

.. _robot-dirs.spec: https://gitlabe1.ext.net.nokia.com/rcp_rpm_specs/robot-dirs/-/blob/master/robot-dirs.spec

5. Dependency management
########################

In order to ensure repeatable and deterministic builds in container pipelines we need to
use predefined versions of Python packages installed into test containers.
This is referred as 'pinning' and shall be done for packages we use (direct dependencies)
and also packages used by them (indirect dependencies) - the whole dependency tree.

Centralized version updates of commonly used packages is a slow process as all container
pipelines must pass in FCI. This process is too heavy for updating packages used only by
a single test container. As a compromise, two levels of dependency management are defined:

- common: dependency management handled by maintainers of robottest-base_ via robot-dirs_
- test container specific: managed by domain, but indirect dependencies constrained to
  common

Both levels shall use requirement files generated with `pip-compile` (provided by pip-tools_)
to ensure repeatable and deterministic pipeline execution, as explained in following sections.

.. _pip-tools: https://github.com/jazzband/pip-tools

5.1. Common packages
~~~~~~~~~~~~~~~~~~~~

Python packages used by multiple test containers are considered common and their versions
are controlled via `/robot/libraries/frozen-robot-deps.txt` provided by ``robot-dirs.rpm``.

Direct dependencies are listed in `robot-dirs/robot-deps.in`_. This file is used as input
when generating `robot-dirs/frozen-robot-deps.txt`_ with `robot-dirs/update-requirements.sh`_.

When common packages are upgraded all dependent container pipelines need to be triggered in FCI
to verify the change. Step-by-step upgrade instructions are available in `robot-dirs/README.md`_.

.. _robot-dirs: https://gitlabe1.ext.net.nokia.com/rcp_rpm_specs/robot-dirs
.. _robot-dirs/robot-deps.in: https://gitlabe1.ext.net.nokia.com/rcp_rpm_specs/robot-dirs/-/blob/master/robot-deps.in
.. _robot-dirs/frozen-robot-deps.txt: https://gitlabe1.ext.net.nokia.com/rcp_rpm_specs/robot-dirs/-/blob/master/frozen-robot-deps.txt
.. _robot-dirs/update-requirements.sh: https://gitlabe1.ext.net.nokia.com/rcp_rpm_specs/robot-dirs/-/blob/master/update-requirements.sh
.. _robot-dirs/README.md: https://gitlabe1.ext.net.nokia.com/rcp_rpm_specs/robot-dirs/-/blob/master/README.md

Test containers using only commonly used Python packages should use `frozen-robot-deps.txt`
as constraint when installing packages with pip:

.. code-block:: dockerfile
    :caption: Dockerfile-test

    RUN pip install -c /robot/libraries/frozen-robot-deps.txt \
                    crl.k8sdeployer \
                    crl.koredump

Note:

  Version of packages is not specified in pip install, instead it is left to be resolved according the constraint.

5.2. Test container specific packages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Version of Python packages used by a single test container can be managed by the domain.

Upgrading such packages is quicker as it doesn't require triggering all container pipelines in FCI like in case of
common packages. On the other hand, care must be taken to avoid dependency conflicts with common packages or their
indirect dependencies:

- custom requirements shall be listed with exact versions in requirements.in file
- `frozen-robot-deps.txt` must be set as constraint

.. code-block::
    :caption: custom/requirements.in

    -c /robot/libraries/frozen-robot-deps.txt
    ldap3==2.9
    hexdump==3.3

- `robot-dirs/update-requirements.sh`_ should be used to generate `custom/requirements.txt` based on the input file

.. code-block:: bash

    update-requirements.sh custom/requirements.in custom/requirements.txt

- the resulting `requirements.txt` shall be used when installing dependencies to test container:

.. code-block:: dockerfile
    :caption: Dockerfile-test

    RUN pip install -c /robot/libraries/frozen-robot-deps.txt \
                    /robot/libraries/<domain>/requirements.txt

Note:

  `frozen-robot-deps.txt` is used as constraint in pip install even if the same file was used when generating
  `/robot/libraries/<domain>/requirements.txt`. This is needed in order to make sure there is no conflict in
  indirect dependencies caused by possible update of `frozen-robot-deps.txt` and domains' own `requirements.txt`
  generated previously. Instructions to solve such conflicts can be found in `robot-dirs/README.md`_.



6. Test data in containers/pods
###############################

6.1. Packaging of test data
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot test data - test cases, resource files and libraries - shall be packaged into
RPM packages. As a convention, the name of these packages shall have '-robottests'
postfix.

All robottest RPMs should declare ownership of their files by listing them in `%files`
section of the rpm.spec file. Note that ``/robot`` directory structure is owned by robot-dirs.rpm
- see robot-dirs.spec_ - and this shall be declared by adding the following lines to
every ``*-robottests.spec`` file:

.. code-block:: spec

  Name: example-robottest
  BuildRequires: robot-dirs
  Requires: robot-dirs

6.1. Deployment of test data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot test data can be "installed" into
test containers using few different methods:

- When building the container image

  - leads to slower development cycle as requires re-building the test
    container for every change in test data you want to test
  - COPY command in dockerfiles - to copy whole directory structure when
    creating robottest container

    - it is sensitive to location from where it is executed, so it is not the
      preferred method

  - using Docker RUN command

    - to install robottests RPM packages

      - 'microdnf install' if default install location is OK
      - 'dnf download' & 'rpm install --prefix' for relocatable RPM packages -
        install destination can be provided by user

    - to install Python packages, for example CRL or other Robot Framework
      libraries ('pip install')

- When creating/deploying robot test pod

  - development cycle is faster compared to previous case, as same container
    image can be reused
  - using hostPath volumes for mounting local volumes into containers on K8s pods

    - see https://kubernetes.io/docs/concepts/storage/volumes/#hostpath

      Limitations:

      - some hosts don't allow hostPath volumes for security reasons

      - container/pod must be constrained to the 'current' Kubernetes node so
        that local path is available. This can be a problem on multi-node
        environments like OE19+NCIR CaaS.

        - solution is to use nodeSelector in pod spec - hostname/IP address of
          current node must be specified in kubernetes.io/hostname

          - on cloud-development environments (NESC) output of 'hostname'
            command can be used (or omitted as there is only one node)
          - on NCIR CaaS environments the value unfortunately not same as
            'hostname' output, instead it is vlan111 IP address

        - ownership of files if volume is used to store test results

          - solution is to provide securityContext for fsGroup with current
            user/group id as value

      Benefits:

      - easy to collect/archive the logs from Jenkins slave
      - logs available even if test run crashed

- After robot test pod is already deployed

  - use ``kubectl cp`` in combination with ``emptyDir`` volumes to copy test data
    into the running test pod

    - fastest development cycle as doesn't require re-building container neither
      re-deploying pod
    - can be used in CaaS environments which enforce strict pod security
      policies including read-only hostPath volumes
    - test pod can be allocated to any Kubernetes node (not only 'current')

  - use ``kubectl exec`` to run ``pip install`` or ``microdnf install`` inside
    robot container

    - may not be used on clouds which enforce strict ``runAsUser`` policy

While ``hostPath`` volumes seem to be the simplest solution, their use is not
recommended due to the limitations listed above.

7. Test results and logs
########################

7.1. Testing with K8s
~~~~~~~~~~~~~~~~~~~~~

Robot test results and generated logs should be collected and archived with
Jenkins build so that analysis of test run is made easy.

The preferred solution is to use ``emptyDir`` volume for ``/robot/logs`` in pod
and transfer files with ``kubectl cp`` after the test run finished.
For benefits and inconveniences see the previous section.

.. note::

  Sometimes ``kubectl cp`` fails to copy large files with the following error::

    $ kubectl cp robot-pod:/robot/logs/output.xml output.xml
    tar: Removing leading `/' from member names
    Dropping out copy after 0 retries
    error: unexpected EOF

  See https://github.com/kubernetes/kubernetes/issues/60140 for
  more details on this topic. The discussion points to ``--retries`` option
  introduced in version 1.23 which could help overcome such errors::

    $ kubectl cp --retries 3 robot-pod:/robot/logs/output.xml output.xml

.. note::

  Copying test results with ``kubectl cp`` which are already mounted as
  ``hostPath`` volume to the same destination will lead to error::

    $ kubectl cp robot-pod:/robot/logs /var/home/cranuser1/my-pipeline/logs/
    tar: Removing leading `/' from member names
    tar: /robot/logs/output.xml: File shrank by 811648 bytes; padding with zeros

  After this content of output.xml will be corrupted.
  For this reason it is important to remove ``hostPath`` mounts at the same time
  when ``kubectl cp`` is taken in use.

7.2. Testing with docker and podman
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the context of running a container with the same command for ``docker`` and ``podman``
in NCIR-like and OCP-like environment, the ``docker`` command is symbolic link to
``podman`` in OCP-like environment and ``podman`` has different behavior with NCIR-like
environment which is running with docker-engine.

.. code-block:: bash

  mkdir logs
  HOST_UID=`id -u`
  docker run --name placeholder-for-container-name-robottest \
             -u $HOST_UID:$HOST_UID \
             --volume `pwd`/logs:/robot/logs $TEST_IMAGE

The main difference is the ``root`` inside of a docker-engine container is the same ``root``
as it is in the host side, the mounted volume directory is owned by the same login non-root
user inside and outside of a container.

Inside of a container in OCP-like environment with ``podman``, the root is mapped
into the same non-root login user UID and the mounted volume directory is owned by this ``root``.
Even it is called as ``root``, from the host side it is still the same non-root login user.

The login user inside of a container looks still like having the same login UID as
it is in the host side, but in the host side, it is actually mapped into a different UID.
They are different. Because the mounted volume directory is owned by the ``root``, the mapped
login user cannot access the mounted volume directory in OCP-like environment and report
error.

The easy solution would be use podman ``unshare`` option from the host side to chown
the directory ownership to match the mapped login user UID inside of a container, so that
it will have access right to mounted volume directory. The ``unshare`` option shall
be run to restore the ownership in host side to avoid failures later after the test run.

.. code-block:: bash

  podman unshare chown $HOST_UID:$HOST_UID -R `pwd`/logs || echo No podman available, ignoring error
  docker run --name placeholder-for-container-name-robottest \
             -u $HOST_UID:$HOST_UID \
             --volume `pwd`/logs:/robot/logs $TEST_IMAGE

  podman unshare chown root:root -R `pwd`/logs || echo No podman available, ignoring error
  docker ps --all -f "name=placeholder-for-container-name-robottest"

Related MR:
https://gitlabe2.ext.net.nokia.com/rcp/containers/template/-/merge_requests/28

Redhat info:
https://access.redhat.com/articles/5946151,
https://www.redhat.com/sysadmin/rootless-podman-makes-sense

8. SSH connection from robottest pod to Kubernetes node
#######################################################

Opening SSH connections from robottest Pod to Kubernetes nodes should be
avoided if possible, as an increased amount of connections might have
unwanted side effects.

Some test scenarios need to open SSH connection from robottest pod to the
kubernetes node from where the pod was deployed - for example to re-deploy SUT
with different parameters.

This can be achieved by injecting the tenant's private SSH key into robottest
pod as a secret:

  - modify yaml config file used to create robot pod as follows:

    - create a secret from base64 encoded contents of the private key
    - mount the created secret as a read-only file `/var/host_key/id_rsa`
    - inject host IP address and user name as environment variables

    .. code-block:: yaml
        :caption: robot_test_pod.yaml
        :emphasize-lines: 2-8,21-35

        ---
        kind: Secret
        apiVersion: v1
        metadata:
          name: host-key
        type: Opaque
        data:
          id_rsa: '$HOST_ID_RSA'

        ---
        apiVersion: v1
        kind: Pod
        metadata:
          name: robot-pod
        spec:
          restartPolicy: Never
          containers:
          - image: robottest-base/robottest-base:2.17.0
            imagePullPolicy: IfNotPresent
            name: robot-pod-container
            env:
              - name: HOST_IP
                value: "$HOST_IP"
              - name: HOST_USER
                value: "$HOST_USER"
            volumeMounts:
              - name: host-key
                mountPath: "/var/host_key/id_rsa"
                subPath: id_rsa
                readOnly: true
          volumes:
          - name: host-key
            secret:
              secretName: host-key
              defaultMode: 0400

  - read host IP address, user name and contents of the private key into
    environment variables
  - use `envsubst` to expand environment variables in the yaml config file

    .. code-block:: bash
        :caption: deploy_robot_pod.sh

        export HOST_IP=`kubectl describe node ${hostname} | awk '/InternalIP/ {print $2}'`
        export HOST_USER=`whoami`
        export HOST_ID_RSA=`base64 -w0 ~/.ssh/id_rsa`

        envsubst < robot_test_pod.yaml |
        kubectl create --namespace=$NAMESPACE -f -


  - deploy pod and run tests. The private key can be used to open SSH connection:

    .. code-block:: robotframework
        :caption: connection_test.robot

        *** Test Cases ***
        Connection to host is successful
            SSHLibrary.Open Connection    %{HOST_IP}
            SSHLibrary.Login With Public Key    %{HOST_USER}    /var/host_key/id_rsa
            ${response}=    SSHLibrary.Execute Command    kubectl get pod robot-pod
            Should Contain    ${response}    Running
            SSHLibrary.Close Connection

  - after tests are executed delete the robot test pod and secret

    .. code-block:: bash
        :caption: cleanup.sh

        envsubst < robot_test_pod.yaml |
        kubectl delete --namespace=$NAMESPACE -f - || echo robot-test-pod not found

9. Common Robot Framework Test Libraries
########################################

Some of the recommended libraries are packaged as Common Robot Library. These
libraries are published to http://pypi.dynamic.nsn-net.net/rcp/prod and
mirrored to https://artifactory-espoo1.ext.net.nokia.com/artifactory/api/pypi/rcp-prod-virtual.

The Artifactory URL above is configured as global index URL for `pip` in `/etc/pip.conf` in
robottest-base image, so `pip install` will use it by default.

When installing robot libraries (or any other Python package) you should use
`/robot/libraries/frozen-robot-deps.txt` file provided by robottest-base as constraints file
to avoid dependency conflicts between your package requirements and the ones
required by robottest-base:

.. code-block:: console

  $ pip install -c /robot/libraries/frozen-robot-deps.txt <library specifier(s) with version>

To avoid manually updating library versions every time a new version is released you
shall use `/robot/libraries/frozen-robot-deps.txt` provided by robottest-base
as constraint file. This file is updated by maintainers of robottest-base.

.. code-block:: console

  $ pip install -c /robot/libraries/frozen-robot-deps.txt \
                <library specifier(s) without version>


Docker container images are published to Artifactory, so in order to avoid
multiple dependencies the pip index from Artifactory should be preferred when
installing python packages.

Library documentation is available only at http://pypi.dynamic.nsn-net.net,
under indexes rcp/prod or crl/prod.

crl.commonobjects
~~~~~~~~~~~~~~~~~

Provides CommonItem and CommonDict classes as known from rcplib, to be used by
other libraries.

Source code is in Gerrit and CRL project in Jira3 is used for issue tracking.

Documentation is found at http://pypi.dynamic.nsn-net.net/crl/prod/crl-commonobjects/latest/+d/index.html

crl.remotelib
~~~~~~~~~~~~~

This library is packaging connection handling part of rcplib from RCP 1.0 and 2.0 cTAF.

Source code is in Gerrit and CRL project in Jira3 is used for issue tracking.

Documentation can be found at http://pypi.dynamic.nsn-net.net/rcp/prod/crl-remotelib/latest/+d/index.html

crl.k8sdeployer
~~~~~~~~~~~~~~~

Package for interacting with Kubernetes - provides the following libraries:

 - K8sDeployer - based on remote ssh connection + kubectl commandfor running
 - K8sClient - based on Python Kubernetes client library, for running tests
   inside a robot-test pod - see acceptance tests for usage examples

Source code is in Gerrit and CRL project in Jira3 is used for issue tracking.

Documentation is found at http://pypi.dynamic.nsn-net.net/crl/prod/crl-k8sdeployer/latest/+d/index.html

crl.logcheck
~~~~~~~~~~~~

Provides keywords for defining syslog ignore and expect conditions and checking
events during tests and suites.

Source code is in Gerrit and CRL project in Jira3 is used for issue tracking.

Documentation: http://pypi.dynamic.nsn-net.net/crl/prod/crl-logcheck/latest/+d/index.html

10. Common robot resources
##########################

Common test resources which are not published/cannot be published/ as python
packages are coming from repository base-robot-test by means of rpm packaging.

<This part is under construction>

11. Sharing available CaaS environments
#######################################

All CaaS environments available for RCP can be reserved manually from TERSY,
under `Kubernetes Reservation`_ menu. At the moment NCIR19, NCS and StarlingX
clouds are listed.

.. _Kubernetes Reservation: https://tersy.eecloud.dynamic.nsn-net.net/create_caas_reservation

Same operation can be done via ``tersycli`` command line tool:

- reserve test environment::

    tersycli create -e CAAS ...

- get target file::

    tersycli get <reservation_id>

- release test environment::

    tersycli release <reservation_id>

One doesn't need to add tersycli commands to their scripts as environment
reservation is integrated to container and helm chart pipelines. For more
details see sections tersyParams_ and runOnHw_ in the documentation.

.. _tersyParams: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/blob/pkg-mgmt-pipes/vars/helmChartPipeline.md#tersyparams
.. _runOnHw: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/blob/pkg-mgmt-pipes/vars/helmChartPipeline.md#runonhw

See also description of ``runOnHw`` parameter in `'tests' section of common
container pipeline documentation`_.

.. _'tests' section of common container pipeline documentation: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/-/blob/pkg-mgmt-pipes/vars/containerPipeline.md#tests

If needed, the target file can be injected to a POD (via hostPath volume,
Secret or ConfigMap).

11.1. Target file format
~~~~~~~~~~~~~~~~~~~~~~~~

When a reservation succeeds ``tersycli get <reservation-id>`` returns the
target file for the reserved environment in yaml format:

.. code-block:: console

    $ cat target_example.yaml
    target:
      host: 10.9.8.7
      password: salasana
      namespace: cran9
      name: estcoe123_cran9
      user: cranuser9

To see all available parameters of a reserved tenant click 'Download ini' button
in TERSY 2.0 UI / `All reservations`_ page.

.. _`All reservations`: https://tersy.eecloud.dynamic.nsn-net.net/all_reservations

Yaml-files can be passed to robot framework with ``-V target.yaml`` command
line option as variable file. In this case fields can be accessed from robot
test data with dotted variable notation, for example ``${target.namespace}``.

12. Robot test containers in Kubernetes
#######################################

Test container should be deployed separately from system under test (SUT).

In Kubernetes terms this means that robot test container should be in a
separate pod, not to be included in same Helm chart or Kubernetes deployment
with SUT.

12.1. Service account permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to enable using Python Kubernetes client based test library one has to
bind to service account in the robottest pod with proper permissions. Add the
following sections to your robottest pod description file:

.. code-block:: yaml

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: robot-runner
      namespace: <your namespace>
    rules:
    - apiGroups: ["apps"]
      resources: ["deployments"]
      verbs: ["create","delete","deletecollection","get","list","patch","update","watch"]
    - apiGroups: [""]
      resources: ["pods", "serviceaccounts","pods/log", "pods/exec", "persistentvolumeclaims"]
      verbs: ["create","delete","deletecollection","get","list","patch","update","watch"]
    - apiGroups: [""]
      resources: ["services", "services/proxy"]
      verbs: ["create","delete","get","list","patch","update","watch"]
    - apiGroups: [""]
      resources: ["secrets"]
      resourceNames: ["singleuser-image-credentials"]
      verbs: ["list","watch","create","get"]
    ---
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: robot-runner
    subjects:
    - kind: ServiceAccount
      name: default
      namespace: <your namespace>
    roleRef:
      kind: Role
      name: robot-runner
      apiGroup: rbac.authorization.k8s.io

Only the predefined "tenant" namespace should be used when testing on OE+NCIR
CaaS environments, so namespace property may be left out from the yaml above.

In case some auxiliary services are needed for running the tests one should
create a sidecar container inside the same robot executor pod.

Test execution status should be propagated to CI pipeline. This can be done
simply by running tests with ``kubectl exec -it``::

    $ kubectl exec -it <pod name> <container_name> -it -- /opt/nokia/bin/start_robot.sh <robot parameters>
    $ echo "Test status is: $?"

Due to `Kubernetes issue 73056`_ long running keywords without feedback on CLI
might cause `kubectl exec` to exit with 0 before the test finished. To avoid
such problems keywords with long execution time (more than 5 minutes) should
log to console, for example:

.. code-block:: robotframework

    BuiltIn.Log To Console     .     no_newline=true

.. _`Kubernetes issue 73056`: https://github.com/kubernetes/kubernetes/issues/73056

12.2. Writable /tmp directory with readOnlyRootFilesystem set to True
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Security rules on some hosts require all pods to use read only root file system.
This can be achieved any host (even if having looser security rules) by the
following section in the pod description file:

.. code-block:: yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: robot-pod
    spec:
      containers:
      - image: test-container
        securityContext:
          readOnlyRootFilesystem: true

Unfortunately this leads to read-only ``/tmp`` directory and inability to run robot
tests, as Robot Framework requires a writable temporary directory. You can avoid
this issue by using ``emptyDir`` volume mount. The following sections are needed in
the pod description file:

.. code-block:: yaml

    volumeMounts:
    - mountPath: /tmp
      name: tmp
    ...
    volumes:
    - emptyDir: {}
      name: tmp


13. Robot tag usage
###################

**Note:** Mandatory tags are marked **bold**, and will be checked automatically
in all cases except the cases with tag "not-ready".

**Note:** Entity testing tagging needs to be defined.

+----------------+--------------------+-------------------------------------------------------------+
|    Tag         |    Example         |    Description                                              |
+================+====================+=============================================================+
| not-*          | not-ready          | The "not-ready" tag is the tag to be used when test is work |
|                | not-run            | in progress or when you do not want the test case to be run |
|                |                    | yet as part of regression testing. This tag is to be        |
|                |                    | removed when the test is production ready.                  |
+----------------+--------------------+-------------------------------------------------------------+
| **owner-**     | owner-ted.tester@  | Email address, preferably distribution group for            |
|                | nokia.com          | responsible team                                            |
|                |                    |                                                             |
+----------------+--------------------+-------------------------------------------------------------+
| product-*      | product-vDU        | As AiC doesn't have currently any other official deployment |
|                |                    | models as real vDU this tag is not yet mandatory.           |
+----------------+--------------------+-------------------------------------------------------------+
|**component-\***| component-RCP_dhcp | Name of component(s) tested by the test case; it is used to |
|                |                    | collect statistics of planned/executed test cases.          |
|                |                    | Please use names defined in `Pipeline configuration`_.      |
+----------------+--------------------+-------------------------------------------------------------+
| release-*      | release-5G20B      |                                                             |
+----------------+--------------------+-------------------------------------------------------------+
| **domain-\***  | domain-CCS         | Domain tag, can be easily set in top level __init__.robot   |
|                | domain-SEC         | file. Please use abbreviation defined in `Domain names`_.   |
+----------------+--------------------+-------------------------------------------------------------+
| requires-*     | requires-oe        | For test cases that require real OE hardware for execution. |
|                +--------------------+-------------------------------------------------------------+
|                | requires-hp        | cases that require HPE Edgeline hardware for execution      |
|                +--------------------+-------------------------------------------------------------+
|                | requires-oam       | test cases that require O&M interface                       |
+----------------+--------------------+-------------------------------------------------------------+
| requires-ifc-* | requires-ifc-sriov | Test case requires specific external interface type, for    |
|                |                    | example SRIOV                                               |
+----------------+--------------------+-------------------------------------------------------------+
| requirement-*  | requirement-       | DOORS requirement id verified by this test case. One test   |
|                | 5G_RAN_123_456     | case can have multiple such tags, one for each requirement. |
|                |                    | 'requirement-N/A' shall be used in exceptional cases when   |
|                |                    | the test instance does not cover any specific requirement.  |
|                |                    | Mandatory for new test cases, not needed for legacy ones.   |
+----------------+--------------------+-------------------------------------------------------------+
| **priority-\***|                    | Generally speaking this is about how critical your test is  |
|                |                    | from basic functionality point of view:                     |
|                |                    |                                                             |
|                |                    | - must: test cases relevant for application release         |
|                |                    | - should: almost full set of automated test cases developed |
|                |                    |   for specific application                                  |
|                |                    |                                                             |
|                |                    | Used to select test cases to be executed; for this reason   |
|                |                    | avoid dynamic manipulation of these tags with ``Set Tags``  |
|                |                    | and ``Remove Tags`` keywords                                |
|                +--------------------+-------------------------------------------------------------+
|                | priority-CIT       | QL2/QL4 test cases that are executed on daily basis in      |
|                |                    | container development and delivery pipeline. These need to  |
|                |                    | pass 100% before code is committed to the trunk             |
|                +--------------------+-------------------------------------------------------------+
|                | priority-CRT       | Can be considered same as QL5 in RCP2.0, these automated    |
|                |                    | cases are executed bi-weekly.                               |
|                +--------------------+-------------------------------------------------------------+
|                | priority-ET-CIT    | Entity testing cases executed daily for 5G features         |
|                |                    |                                                             |
|                +--------------------+-------------------------------------------------------------+
|                | priority-ET-CRT    | Entity testing cases executed bi-weekly for 5G features     |
|                |                    |                                                             |
+----------------+--------------------+-------------------------------------------------------------+
| subarea-*      | subarea-BCT        | Cross-container/component/pod test case used for backward   |
|                |                    | compatibility testing in CIT; with these test cases don't   |
|                |                    | need to schedule other domains' all CIT cases for backward  |
|                |                    | compatibility testing. If several cases cover the same      |
|                |                    | dependency, only need to tag one of them with this tag.     |
|                +--------------------+-------------------------------------------------------------+
|                | subarea-light      | Test case used for light acceptance test in CIT.            |
|                |                    | Total execution time of the light acceptance case set for   |
|                |                    | any deliverable should be less than 5 minutes. These test   |
|                |                    | cases must be stable, 10 successful runs are needed before  |
|                |                    | adding to light acceptance case set. Any light acceptance   |
|                |                    | case set change must be approved by TAO.                    |
|                +--------------------+-------------------------------------------------------------+
|                | subarea-robustness | Tag used to identify robustness test case.                  |
|                |                    | Results of these test cases are uploaded to COOP.           |
+----------------+--------------------+-------------------------------------------------------------+
| jira-*         | jira-FCA_RCP-12345 | Jira key id, can be CA level or Epic level. It indicates    |
|                |                    | this case is used to verify related item                    |
|                |                    |                                                             |
+----------------+--------------------+-------------------------------------------------------------+
| pronto-*       | pronto-PR123456    | Pronto id, it indicates this case is used to verify related |
|                |                    | pronto                                                      |
+----------------+--------------------+-------------------------------------------------------------+
| case_id-*      | case_id-RCP_CM_007 | Value of `Test case ID` field for the test case in Quality  |
|                |                    | Center. First word of test case name is used if not set.    |
+----------------+--------------------+-------------------------------------------------------------+
| dashboard-CP   | dashboard-CP       | Capacity & Performance test cases for which test results    |
|                |                    | should be uploaded to C&P dashboard                         |
+----------------+--------------------+-------------------------------------------------------------+

.. _Domain names: https://confluence.ext.net.nokia.com/display/RCP/Domain+names
.. _Pipeline configuration: https://nokia.sharepoint.com/sites/RCPTAO/Shared%20Documents/General/Pipeline%20configuration.xlsx?web=1

NOTE:

  It needs to be checked if we can define some concrete requirements for
  those test cases that can't be executed in NESC.

NOTE: Avoid dynamic tag manipulation

  As ``priority-*`` tags are used to select test cases to be executed, we should
  have only one priority tag in one case. Robot Framework supports
  both suite level tagging (via ``Force Tags`` in settings table) and individual
  case tagging (``[Tags]`` in test case tables), but it is not possible to unset
  a tag forced on suite level before the test is already running. For this reason
  **priority tags should be set on test case level** (instead of ``Force Tags`` in
  settings table and ``Remove Tags`` keyword calls during test execution).

.. _Tagging test cases: http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#tagging-test-cases

Mapping of ``Automation level`` field in QC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Automation level`` field of tests in QC Test Plan is mandatory, and should
match ``priority-`` tags of robot test cases as follows:

- ``RCP-CIT``: ``priority-CIT`` triggered by code change in development/delivery pipeline
- ``RCP-CRT``: ``priority-CRT`` executed bi-weekly
- ``RCP-Manual-RT``: manual, executed before application releases
- ``RCP-Manual-NOT-TO-BE-RT``: one time testing during feature development (e.g. exploratory testing)

Mapping of ``Test case ID`` field in QC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Test case ID`` field of tests in QC is used to identify Robot test cases when
uploading test results. Value of the field is formed as follows::

    vRAN2.0: QC Test case ID = given by case_id-* tag if exists
                       OR
                       first word of robot case name
             QC case name = full robot case name

If the case ID tag is added to a test case, the name shall either not have any
case ID prefix at all in its name, or if it has, then it must be exactly the
same as the test case ID in the tag. Note that by convention the case ID must
start either with the domain name of the owning domain of the test cases, i.e.
``CM_``, or with the prefix ``RCP_<domain-name>``, i.e. ``RCP_CM_``.
This prefix shall always be the same as in the domain- tag, but without the
``RCP_`` prefix. All test cases and their test case ID prefixes must belong to
one and only domain as owning domain, and it must be an established domain ID
as per column B "abbreviation" of the RCPWhoIsWho. This prefixing is a simple
way of guaranteeing uniqueness of the IDs. Also it helps quickly identifying to
which domain a test case belongs without having to check the domain tag.
Yes, there is some room for improving this, e.g. to add prefixes automatically,
but sometimes it's just simpler to stick to conventions.

Note that as of 2017-12-15 the large majority of test cases in RCP do not use
the case ID, but instead use the first word of the robot case name as
identifier. As per `a short guide to qcutils`_
the ID must be shorter than 40 characters.

Starting from June-06-2018 (for the sake of consistency with legacy naming
rules it's ok to continue following old rules in a particular domain or folder)
new case IDs should follow the syntax as above, and additionally fulfill the
following syntax::

    <domain-prefix-as-above>_<middle-string>_<3-digit-number-with-leading-zero>

The <middle string> is anything that the domain team agrees on to identify
their test cases. Most appropriate is to use it for further subdividing the
domain, e.g. in case of CM into "Activator", "Conversion", etc.
After this, the middle string can have additional information, like what aspect
is tested, e.g. "Restart", "Failure". But often it's better to stop after the
second-level subdivision and just use numeric IDs (the last part of the ID) to
separate test cases in the same sub-domain. Note that the numeric ID includes
leading zeros (for easy alphabetical sorting).

Good examples are: ``CM_Activator_001``, ``CM_Activator_002``,
``CM_Conversion_001``, ``CM_Mass_Operations_DB_Restart_001``,
``CM_Mass_Operations_DB_Restart_002``, ``CM_Mass_Operations_DB_Failure_001``,
``CM_Mass_Operations_DB_Failure_002``. Note how the numeric IDs are unique
within the scope of the complete ID that precedes it.
I.e. having ``CM_Mass_Operations_DB_Restart_001``, and
``CM_Mass_Operations_DB_Failure_001`` (both with the ID 001) is perfectly fine
as they differ in their prefix.

Related confluence pages:

- `A short guide to qcutils`_

.. _a short guide to qcutils: https://confluence.ext.net.nokia.com/display/RCP/A+short+guide+to+qcutils

Mapping of ``Backlog ID`` and ``Requirements`` fields in QC test instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When uploading robot test results to Quality Center the following test instance
fields are updated according to robot tags (if such tags are present):

- Backlog ID: value(s) taken from 'jira-\*' tag(s), concatenated with ';' if
  more than one found
- Requirements: value(s) taken from 'requirement-\*' tag(s), concatenated
  with ';' if more than one found

To update these fields you need to specify the relevant `qcutils robot2qc`_
option(s) - ``--update-backlog-id`` and/or ``--update-requirements`` for
your tests' `robot2qcParams`_ in your Jenkinsfile:

.. code-block:: groovy

   Map robot2qcParams = [
      'set': 'Root/qcsupport/ci-helm',
      'test-case-search-root': 'Subject/qcsupport/ci-helm',
      'options': '--update-backlog-id --update-requirements'
  ]

.. _qcutils robot2qc: https://pypi.dynamic.nsn-net.net/utils/prod/qcutils/latest/+doc/CLI.html#robot2qc
.. _robot2qcParams: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/-/blob/pkg-mgmt-pipes/vars/containerPipeline.md#tests

14. FCI integration
###################

FCI has support for passing parameters to pipelines, see section `Common Jenkins job parameters`_ for all the available
parameters. In this chapter we explain some of those from the perspective of selecting test cases to be executed.

.. _Common Jenkins job parameters: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/-/blob/pkg-mgmt-pipes/vars/containerPipeline.md#common-jenkins-job-parameters

14.1 TESTCASE_SET
~~~~~~~~~~~~~~~~~

FCI is using this parameter to select test cases to be executed in the pipeline, possible value
being for example ``--include subarea-BCT --include subarea-light``. As many containers divide
tests to be executed into parallel sets with similar ``--include`` rules, combining them with
``TESTCASE_SET`` may lead to complicated boolean logic. Reason is that Robot Framework will
select any test matching one of the include conditions, so one needs to use boolean operators to
combine them.

To avoid this it is recommended to use ``--exclude`` conditions for grouping test cases into
parallel sets, leaving ``--include`` solely for FCI, like in the following example:

- group tests to be run in 2 parallel sub tests (groups) via exclude rules: ``--exclude NOTset-ethip``
  for one group and ``--exclude NOTset-qos`` for the other
- include rules given by FCI can be simply appended to the robot command after exclude rules, like
  ``--exclude NOTset-ethip --include subarea-BCT``

This way there is no need to combine the two include tagging rules with boolean logic, Robot Framework
is doing the trick automatically.

For more details check `this discussion on same topic at nwmgmt container`_.

.. _this discussion on same topic at nwmgmt container: https://gitlabe1.ext.net.nokia.com/RCP/Containers/nwmgmt/-/merge_requests/483#note_7070057

15. Container and Helm pipelines
################################

Pipeline for building, testing and publishing containers is described at containerPipeline_.

.. _containerPipeline: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/-/blob/pkg-mgmt-pipes/vars/containerPipeline.md

Pipeline for helm chart testing and publishing is described at helmChartPipeline_.

.. _helmChartPipeline: https://gitlabe1.ext.net.nokia.com/RCP/pipeline-utils/blob/pkg-mgmt-pipes/vars/helmChartPipeline.md
