***************************
Small git commit guidelines
***************************

.. contents:: :local:

Motivation
##########

One git commit should form one logical entity. One git commit should not add,
remove or change more than 200 lines of code. By following these guidelines we
can achieve the following:

1. Writing a commit message is easy, because the commit message describes only
   that *one thing*
2. Reviewing the commit is easy and efficient, because reviewers can
   concentrate only on that *one thing*
3. Using ``git revert`` will be easy, if needed
4. Using ``git cherry-pick`` will be easy, if needed

Guidelines
##########

The first commit in a new repository
************************************

When creating a new code repository, the first commit should add a compilation
framework. The first commit should not contain any real code, only dummy code
to prove that the compilation framework works (i.e. dummy code can be compiled
and dummy unit test cases can be executed).

Setting up the initial compilation framework may require more than 200 lines of
code. That is not a problem, because most of those lines are trivial and can be
reviewed quickly.

Example commit:
`Add initial compilation framework <https://gitlabe2.ext.net.nokia.com/rcp-security/certfetcher/commit/7973335e82f71739fffd877cb9dc717b3ed408d0>`_

Add one class per commit
************************

When implementing new code, try to add one class (or equivalent) per commit.
Avoid adding multiple classes in the same commit even if the classes are related
to each others. Ideally the commit should add one header file, one
implementation file, one unit test file and modify ``Makefile.am`` or
``CMakeLists.txt``.

**Always include unit test code into the same commit with the production code**.

Example commit:
`Add FileDescriptor class <https://gitlabe2.ext.net.nokia.com/rcp-security/certfetcher/commit/f935b5c105e24ef48d76ea88f826f9bc69f910df>`_

If one class (together with unit tests) is too big to implement in one commit
(considerable more than 200 lines), then you can implement the class in several
commits. Mark unimplemented functions or branches with TODO-comments. For
example:

.. code-block:: c++

 void MyClass:func(int param)
 {
     /** @todo IMPLEMENT ME! */
     static_cast<void>(param);
 }

Example commit:
`Start implementing simple storage plugin <https://gitlabe1.ext.net.nokia.com/genapi/genapiaws/commit/4ced39b65d2c5d8f65f47bd2f3a3832902305156>`_

Add one test double per commit
******************************

When implementing test doubles (e.g. mocks), try to add one test double per
commit. Avoid implementing any functional changes or unit test cases in the same
commit. The motivation is to make code reviews quick and to reduce noise in
actual *real code* commits.

Example commit:
`Add AsyncStorageMock <https://gitlabe2.ext.net.nokia.com/sdlevent/sdlevent/commit/12f606aa663459b8547e62d79d38c0cffec70c41>`_

Fix one bug per commit
**********************

When fixing a bug in existing code, fix one bug in one commit. Do not make any
other changes in the same commit. This has the following benefits:

1. Explaining the bug and the fix in the commit message is easy
2. Reviewing the commit is easy
3. Cherry-picking the commit to maintenance branches is easy
4. If the fix turns out to be incorrect, reverting the commit is easy

Example commit:
`Update Aatestportclient ministarter configuration <https://gitlabe2.ext.net.nokia.com/rcp-security/certfetcher/commit/7a0950d4f6d8978d3a81a6da4982a33e5a05a52b>`_

Do one refactoring per commit
*****************************

When refactoring existing code, do one refactoring in one commit. Do not make
any other changes in the same commit. This has the same benefits as
`Fix one bug per commit`_.

Example commit:
`Replace boost::optional with std::optional <https://gitlabe1.ext.net.nokia.com/genapi/genapi/-/commit/9efc0bf409079ef9a2e4b916a1f8cbe05a89f879>`_

Do not mix format changes with functional changes
*************************************************

When fixing or changing source code format, always do that in a separate commit.
Never do any functional changes in the same commit. If the whole repository is
reformatted (for example with ``clang-format``), then the commit size may exceed
200 lines. That is not a problem if there are no functional changes. It is
enough that reviewers agree about the new format.

Example commit:
`Run "make format" <https://gitlabe1.ext.net.nokia.com/genapi/genapi/-/commit/c115516b84887169fa1f5662b8982167b3c03763>`_

Update version numbers in a separate commit
*******************************************

If the repository contains library version or component version numbers in
``configure.ac``, ``Makefile.am`` or ``CMakeLists.txt``, then update those in a
separate commit. Do not do any other changes in the same commit. This is because
if code changes needs to be reverted or cherry-picked, version number changes
must not be reverted or cherry-picked along with the code changes.

If the repository contains ``CHANGELOG.md`` (or equivalent), then that can be
changed in the same commit with the version numbers.

Example commit:
`genapi v1.7.0 <https://gitlabe1.ext.net.nokia.com/genapi/genapi/commit/6beb960c4d345d3effabf9dbc333089287f18e67>`_

Using git revert
****************

When the above guidelines are followed, using ``git revert`` is easy. By
reverting one commit you revert one logical change. Avoid doing any manual
changes in the same commit (except for fixing possible conflicts).

Add explanation **WHY** the commit was reverted into the commit message
(but do not delete or change the automatically generated lines).

Example commit:
`Revert "Add backward compatibility for notification key subscribing" <https://gitlabe2.ext.net.nokia.com/rcp-security/certfetcher/commit/221c960731a4bf42bf178b66c6a0102b0e5a6d93>`_

Using git cherry-pick
*********************

When the above guidelines are followed, using ``git cherry-pick -x`` is easy. By
cherry-picking one commit from master to maintenance branch you can be sure you
cherry-pick only the needed bug fix. Avoid doing any manual changes in the same
commit (except for fixing possible conflicts).

Remember to use the ``-x`` option to get the automatic
``(cherry picked from commit xxx)`` text appended to the commit message. Avoid
changing the commit message manually.

Do not push the cherry-pick commit for review before the original commit has
been reviewed and merged. This is because:

1. The SHA-1 hash of the original commit is not certain until the commit has
   been merged
2. There is no point to review the same change twice

Example commit:
`Do not throw exception if no CRL DPs are written <https://gerrite1.ext.net.nokia.com/c/scm_rcp/SS_CertMan/+/1391835>`_

Additional material
###################

See
`RCP info session : Code review sharing <https://confluence.ext.net.nokia.com/display/RCP/2022-09-06++RCP+info+session+%3A+Code+review+sharing>`_
and the related material
`Making big change in small pieces <https://nokia.sharepoint.com/:b:/r/sites/rcpswdevelopmentguild/Shared%20Documents/making_big_change_in_small_pieces.pdf?csf=1&web=1&e=xpD46u>`_.
