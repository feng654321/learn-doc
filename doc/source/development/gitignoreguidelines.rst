**********************
Git ignore Guidelines
**********************

Introduction
###############

A **.gitignore** file specifies intentionally untracked files that Git
should ignore. Use Git ignore is one of `the best practices for managing Git repos <https://opensource.com/article/20/7/git-repos-best-practices>`_
List some benefits of Git ignore in the following:

1. Make `git status` list clean from unwanted files

The following bad example shows a noisy `git status` list
after compiling without a **.gitignore** file.
It is quite difficult to do `git add` because you need to find your new
changes from the long list.
Please add the unwanted files and folders into a **.gitignore** file to
make the `git status` list clean.
::

  $git status
  ...
  Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .symlink.lock
        SCLI/src/CertManSCLIController.d
        SCLI/src/CertManSCLIController.o
        SCLI/src/CertManSCLIDeleteCACert.d
        SCLI/src/CertManSCLIDeleteCACert.o
        SCLI/src/CertManSCLIDeleteCMP.d
        SCLI/src/CertManSCLIDeleteCMP.o
        SCLI/src/CertManSCLIDeleteCRLDP.d
        SCLI/src/CertManSCLIDeleteCRLDP.o
        ..
        tst/ut/Certificate_test.gcda
        tst/ut/Certificate_test.gcno
        tst/ut/Certificate_test.o
        tst/ut/build.tag
        tst/ut/lcov_report/
        tst/ut/testRunner
        tst/ut/testreport.xml
        upgradeapp/CertManUpgradeApp.d
        upgradeapp/CertManUpgradeApp.o
        upgradeapp/build.tag
        upgradeapp/certmanupgradeapp
        validator/CertManCertificateValidator.d
        validator/CertManCertificateValidator.gcda
        validator/CertManCertificateValidator.gcno
        validator/CertManCertificateValidator.o

2. Make `Git add` changes simple

Before `Git add`, please review the untracked file list by `git status`.
If there is unwanted files, you can specify them in the **.gitignore** file.
Then you can use `git add .` to add all new files and modifications
in once.

3. Ignore the files without git version control

There are some files in git repository without git version control,
for example, libtool.m4. To avoid such files appear in untracked
file list, please specify they in the **.gitignore** file.

Thus, basically most repositories shall contain at least one **.gitignore** file.

Here is `an example of the gitignore file <https://gerrite1.ext.net.nokia.com/plugins/gitiles/scm_rcp/SS_CertMan/+/refs/heads/rcp2.0/.gitignore>`_.

Please refer to `git webpage <https://git-scm.com/docs/gitignore>`_ for more details.

Templates for .gitignore
########################

The `github project <https://github.com/github/gitignore>`_ has a collection of
**.gitignore** templates. You can take the related template as a reference when you
start to write a **.gitignore** file.
