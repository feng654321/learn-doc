**********************
Code Review Guidelines
**********************

.. contents:: :local:

1. Review groups
################

In order to help teams with reviews in specific area there are predefined review
groups which must be invited in addition to own team members:

+--------------+---------------------------------------+----------------------------------+
|RPM SPECs     |@rcp-computing, @tuturune              | This review group is mandatory   |
+--------------+---------------------------------------+----------------------------------+
|Commit Log    |@rcp-commit-log-reviewers              | This review group is mandatory   |
+--------------+---------------------------------------+----------------------------------+
|Helm Charts   |@rcp-helm-chart-default-reviewers      | This review group is mandatory   |
+--------------+---------------------------------------+----------------------------------+
|Containers    |@rcp-container-default-reviewers       | This review group is mandatory   |
+--------------+---------------------------------------+----------------------------------+
|CI            |@RCP-CI-default-reviewers              | This review group is mandatory   |
+--------------+---------------------------------------+----------------------------------+
|Code          |not defined                            |                                  |
+--------------+---------------------------------------+----------------------------------+

There are also special interest groups which can be joined and used to
get comments from specific areas or to follow reviews in those areas out
of interest or for competence learning:

+--------------+---------------------------------------+----------------------------------+
|C/C++         |@rcp-cpp-reviewers                     | This review group is optional    |
+--------------+---------------------------------------+----------------------------------+
|Bash scripts  |@rcp-bash-reviewers                    | This review group is optional    |
+--------------+---------------------------------------+----------------------------------+
|Python        |@rcp-python-reviewers                  | This review group is optional    |
+--------------+---------------------------------------+----------------------------------+
|Go            |@rcp-go-reviewers                      | This review group is optional    |
+--------------+---------------------------------------+----------------------------------+
|Robot         |@rcp-robot-reviewers                   | This review group is optional    |
+--------------+---------------------------------------+----------------------------------+

1.2. How to join review groups
******************************

The reviewer groups are created in **gerrite1**, **gitlabe1** and **gitlabe2**.

In order to join the group in Gitlab, navigate to group and click **Request access**.
The **Request access** appears on the main page of the group, not in Members
area. For more info where to find it look at
`Request access to a group <https://docs.gitlab.com/ee/user/group/#request-access-to-a-group>`_.

In Gerrit click the **[Members]** button left, put the email address in
the **[Members]** box and click **[ADD]**.

.. list-table::
   :header-rows: 1

   * - **gitlabe1**
     - **gitlabe2**
     - **gerrite1**
   * - `rcp-cpp-reviewers <https://gitlabe1.ext.net.nokia.com/groups/rcp-cpp-reviewers>`__
     - `rcp-cpp-reviewers <https://gitlabe2.ext.net.nokia.com/groups/rcp-cpp-reviewers>`__
     - `rcp-cpp-reviewers <https://gerrite1.ext.net.nokia.com/admin/groups/4032,members>`__
   * - `rcp-bash-reviewers <https://gitlabe1.ext.net.nokia.com/groups/rcp-bash-reviewers>`__
     - `rcp-bash-reviewers <https://gitlabe2.ext.net.nokia.com/groups/rcp-bash-reviewers>`__
     - `rcp-bash-reviewers <https://gerrite1.ext.net.nokia.com/admin/groups/4033,members>`__
   * - `rcp-python-reviewers <https://gitlabe1.ext.net.nokia.com/groups/rcp-python-reviewers>`__
     - `rcp-python-reviewers <https://gitlabe2.ext.net.nokia.com/groups/rcp-python-reviewers>`__
     - `rcp-python-reviewers <https://gerrite1.ext.net.nokia.com/admin/groups/4034,members>`__
   * - `rcp-go-reviewers <https://gitlabe1.ext.net.nokia.com/groups/rcp-go-reviewers>`__
     - `rcp-go-reviewers <https://gitlabe2.ext.net.nokia.com/groups/rcp-go-reviewers>`__
     - `rcp-go-reviewers <https://gerrite1.ext.net.nokia.com/admin/groups/4035,members>`__
   * - `rcp-robot-reviewers <https://gitlabe1.ext.net.nokia.com/groups/rcp-robot-reviewers>`__
     - `rcp-robot-reviewers <https://gitlabe2.ext.net.nokia.com/groups/rcp-robot-reviewers>`__
     - `rcp-robot-reviewers <https://gerrite1.ext.net.nokia.com/admin/groups/4036,members>`__

You need to join each of the git hosting servers separately.


.. _How to use review groups:

1.3. How to use review groups
*****************************

In Gitlab when you wish to invite specific review group use **@group-name**,
for example **@rcp-robot-reviewers** and mail notification will be automatically
sent to the group members.

For Gerrit you need to ask existing member to add you.

If you already are one of the members of the group, to add new members
click the **[Members]** button, in text box on the left write the email
address or name in the **[Members]** box, it is will pop-up the full
address of the member, select it and click **[ADD]**.
E.g. you can search Zizka, Jan (Nokia - CZ/Prague) by just input `zizka` in the box,
it is will pop-up "Zizka, Jan (Nokia - CZ/Prague) <jan.zizka@nokia.com>",
select it and then **[ADD]** button can be clicked.

You have to select the address that pop-up by gerrit, otherwise the **[ADD]**
button will appear gray and you can not click it.

If you are not a member of the group, click the link and find one member
of the group and send emails to ask them to add you to the group.


2. MUST - Review prerequisites
##############################

Developers must be responsible for their work and they are expected to learn
needed skills to submit code for review which doesn't have basic deficiencies.
Therefore:

It is responsibility of developer who submits change for review to fulfill
following basic criteria, without those the review group may refuse to do code
review. None of this is a rocket science and anyone should be able to ensure
these basic criteria. The review group is heavily overloaded and it is waste to
ask for review when there are basic errors in the commit:

1. :boldred:`There is no dead code`
2. :boldred:`There are no formatting errors`

   a. Indentation is aligned with project source
   b. There is no mix of TABs and spaces
   c. There are no extra spaces at the end of line or on empty lines
   d. There are no new lines missing at the end of files

3. :boldred:`There are no obvious spelling errors`
4. :boldred:`There is no obvious code duplication visible with naked eye`
5. :boldred:`Commit log complies to basic conventions`
   `<https://chris.beams.io/posts/git-commit/>`_

   You can use exception to rule 3. Capitalize the subject line from the link. If the
   subject begins with a submodule label, then the subject format can follow:

   <submodule>: <Capitalized sentence>

   a. <submodule> can start with a lower-case letter
   b. There is a space after the colon
   c. <Capitalized sentence> should be capitalized as rule 3 required

   There is an example from the Linux kernel commits
   https://github.com/torvalds/linux/commit/7968c7c79d3be8987feb8021f0c46e6866831408

6. :boldred:`The change was reviewed by own team member`

   a. Own team member have to leave comment that review was really done, thumbs-up, or +2 is not enough
   b. The own team reviewer must not be same person as the submitter
   c. The team reviewer guarantees that everything on prerequisites was checked

7. :boldred:`The prerequisites are checked from this page as they might change
   from time to time`
8. :boldred:`SPEC file changes comply to guidelines: Checklist for spec file review`
   `<https://confluence.ext.net.nokia.com/display/RCP/Checklist+for+spec+file+review>`_
9. :boldred:`HELM chart changes comply to guideline: RCP Helm Chart Guidelines`

   a. :boldred:`Also the latest helm repository template is checked`
      `<https://gitlabe2.ext.net.nokia.com/rcp/helmcharts/template>`_

10. :boldred:`CONTAINER repository changes are in line with common-container-pipeline
    documentation`
11. :boldred:`CODE change has unit tests`
12. :boldred:`Pipeline must be passed, or explanation given why the pipeline has
    not been successfully executed.`

It is perfectly ok to say that all was checked to best of persons knowledge. If
a gap is found then the competence development will be started to cover the gap.
There are tools to help with many of the tasks, competence development can be
for effective tool use for example. It is good to fail as that gives opportunity
to learn.

3. Review acceptance criteria
#############################

1. :boldgreen:`Mandatory review group, is invited`
2. :boldgreen:`Two thumbs up, there are at least two thumbs-up on GitLab or two
   reviewers in Gerrit with +1 and +2`
3. :boldgreen:`Pipeline passes, pipeline attached to component is passing`
4. :boldgreen:`No unresolved discussions, there are no unresolved discussions`

The team can decide to overrule the criteria in exceptional cases, in which case
it is mandatory to state that clearly in review with reasons why commit is
merged without fulfilling any of the criteria. It is fully then responsible of
the designer to guarantee quality of the change.

In case of simple version bumps, for example when updating container image versions
used for helm chart, without any changes to chart itself, the team can accept the
merge request without comments from default reviewers.

4. Review rules
###############

1. :boldgreen:`Use of WIP`, if the work is still under progress and change should not be
   reviewed yet use `WIP` flag to indicate that ( in review title or in the title
   of commit logs)
2. :boldgreen:`Author resolves comments`, never resolve discussions started by
   other, always wait for author of the comment to resolve discussion ( in case of
   delays, others can resolve, but then clearly state in review reasoning)
3. :boldgreen:`Everything works locally`, before inviting for review make sure the code
   compiles, there are no linting errors and RPM package can be built, eventually
   that the component, container or helm CI is passing
4. :boldgreen:`Invite for re-review after fix`, once a review comment is corrected and new
   change is pushed indicate so in review to inform author of the comment to
   re-review
5. :boldgreen:`Split changes`, make sure that changes which logically belong together are in
   single commit, each commit must contain only one change, if needed split changes
   to multiple commits, re-factoring and format changes must be in separate commits
6. :boldgreen:`Review has no more than 200 lines`, reviewing more than 200 lines of changed code
   is not effective and impossible to make sure no problems escape, split your work
   to smaller changes which are reviewed separately
7. :boldgreen:`Comment only on changed lines`, if commenting on legacy code which is already
   in code base always be careful, you can create issue or JIRA ticket for further
   followup of issue found in legacy code unless it is a critical bug.
8. :boldgreen:`Commit message describes changes`, the commit message must clearly describe what is
   changed and why in order to let reviewers know the intent of the change.

5. Resolving discussions
########################

1. :boldgreen:`Wait for author to resolve`
2. :boldgreen:`Create issue or JIRA ticket to handle reported problem with additional
   work task`
3. :boldgreen:`Write proper explanation if resolving`
4. :boldgreen:`Use consensus, if it is impossible to agree on specific reported problem
   invite additional expert to create consensus`
5. :boldgreen:`You may close discussion if author will not respond in 2 days, write
   note to discussion that you are closing it due to delays`

6. Guidelines for reviewers
###########################

6.1. Common review finding comments
***********************************

This is collection of comments reviewers can re-use for common problems found in
RCP code. Credits to Turunen, Tuomo (Nokia - FI/Espoo)

6.1.1. General
**************

.. code-block:: none

    Looks like you are using this branch for development, please specify `WIP` [1]
    flag to avoid unnecessary work for reviewers.

    Remove the `WIP` are re-invite reviewers once you are done and MR is ready.

    [1] https://docs.gitlab.com/ee/user/project/merge_requests/work_in_progress_merge_requests.html

.. code-block:: none

    Please make sure the review prerequisites are all followed
    before review can be done by a designated review group [1].

    [1] https://confluence.ext.net.nokia.com/display/RCP/5.18+Code+Review+Guidelines#id-5.18CodeReviewGuidelines-MUST-Reviewprerequisites

    Regarding "No newline at end of file"


    [1] C99 Ch5.1.1.2
    2. Each instance of a backslash character (/) immediately followed by a
    new-line character is deleted, splicing physical source lines to form logical
    source lines. Only the last backslash on any physical source line shall be
    eligible for being part of such a splice. A source file that is not empty shall
    end in a new-line character, which shall not be immediately preceded by a
    backslash character before any such splicing takes place.


    [2] C99 Rationale Ch5.1.1.2
    A backslash immediately before a newline has long been used to continue
    string literals, as well as preprocessing command lines.  In the interest of
    easing machine generation of C, and of transporting code to machines with
    restrictive physical line lengths, the C89 Committee 20 generalized this
    mechanism to permit any token to be continued by interposing a backslash/newline
    sequence.


    [1] http://www.open-std.org/jtc1/sc22/WG14/www/docs/n1256.pdf
    [2] http://www.open-std.org/jtc1/sc22/wg14/www/C99RationaleV5.10.pdf

.. code-block:: none

    One space after periods, question marks, exclamation points, and colons.


6.1.2. Git Commit Messages
**************************

.. code-block:: none

    Commit message doesn't comply with conventions from [1].
    Please add at least the title line, the metadata can follow after a blank line.
    Look also at slide 23 from [2] to see as example, note that same applies also
    to RCP software not only to FOSS.

    [1] https://chris.beams.io/posts/git-commit/
    [2] https://confluence.ext.net.nokia.com/display/RCP/4.3.1+RCP+code+commit+log+guide?preview=/489285279/708816041/Traing_RCP_CommitLogTool_FCI_2.pptx

6.1.3. SPEC files
*****************

.. code-block:: none

    Otherwise looks good, but please squash the commits to a single commit with proper commit message:

    https://confluence.ext.net.nokia.com/pages/viewpage.action?pageId=748045868#Checklistforspecfilereview-SquashMRcommitsintosinglecommitbeforemerging

.. code-block:: none

    Remove **all** `Group` directives. See
    https://confluence.ext.net.nokia.com/display/RCP/Checklist+for+spec+file+review#Checklistforspecfilereview-Groupdirective

.. code-block:: none

    The changelog is expected to explain changes done in this
    spec file, not changes done in the source code repository. Simple
    "Update to x.y.z" should be good enough. Readers can assume that the
    changes in file list are caused by the new version.
    What kind of versioning scheme this subsystem follows? Is it documented somewhere?
    You need to update `Version` and/or `Release` and add changelog. See "Scenario 1" from [1].

    [1] https://confluence.ext.net.nokia.com/pages/viewpage.action?pageId=813420401#ChangestocurrentFCIdashboardtosupportKojirelevantverificationandpromotion-HowFCIsupportdiffrentUserscenariobasedoncombinationsofsrc/specchange


6.1.4. Code
***********

.. code-block:: none

    Remove extra spaces after end of the line.

.. code-block:: none

    Remove dead code

.. code-block:: none

    Remove code duplication

.. code-block:: none

    Correct formatting / indentation to be consistent / same with rest of the file.

.. code-block:: none

    Format changes should be done in separate commits. Otherwise it is
    impossible to see the difference. Also "git revert" and "git
    cherry-pick" are useless, if you mix format changes with logic changes.

.. code-block:: none

    Add newline at end of file.
    Refer to: https://stackoverflow.com/questions/5813311/no-newline-at-end-of-file
    Use 80 characters per line.
    Refer to: https://nickjanetakis.com/blog/80-characters-per-line-is-a-standard-worth-sticking-to-even-today

6.2. Best practices for merge request
#####################################

Refer to
`<https://gitlabe2.ext.net.nokia.com/cloud-infra/document/blob/master/best-practices-for-merge-request.md>`_

6.3 Keywords
############

The following keywords should be added into comments by reviewers. They are used
for collecting statistics about problems found by reviews.

* General:

    * :boldgreen:`REVIEW-LOOKS-GOOD`: LGTM, perfect (If there are other
      comments for this MR, then it will be ignored)
    * :boldgreen:`REVIEW-CHANGE-TOO-BIG`: Too many changes in one MR
    * :boldgreen:`REVIEW-LANG-ERROR`: Spelling and grammatical errors (language issue)

* Commit log:

    * :boldgreen:`REVIEW-COMMIT-GOOD`: The commit log was good, nothing
      needs to be updated. (Commit log which was already commented does not count)
    * :boldgreen:`REVIEW-COMMIT-MISS-WHY`: The commit log had no why at all,
      or had why issues (fake why, incorrect why...)
    * :boldgreen:`REVIEW-COMMIT-NEED-SPLIT`: The commit should be split
    * :boldgreen:`REVIEW-COMMIT-FORMAT-ERROR`: The commit log did not follow
      No.1~6 rules in https://chris.beams.io/posts/git-commit

* Source code:

    * :boldgreen:`REVIEW-CODE-BUG`: Bugs, code errors, wrong algorithms
    * :boldgreen:`REVIEW-FORMAT-ERROR`: Formatting and indentation errors
    * :boldgreen:`REVIEW-DUPLICATED-CODE`: Duplicated code
    * :boldgreen:`REVIEW-UT-MISSING`: Missing Unit test cases
    * :boldgreen:`REVIEW-UT-DESIGN`: A problem in UT design
    * :boldgreen:`REVIEW-UT-NO-CHECKS`: There are no or inadequate checkpoints
    * :boldgreen:`REVIEW-DESIGN-ISSUE`: The design is not right, for example
      prevents changes in future
    * :boldgreen:`REVIEW-DEAD-CODE`: Change contains dead code
    * :boldgreen:`REVIEW-VERSION-UPDATE-MISSING`: Version needs to be
      updated, but it has not been done. For example do not update helm
      chart version or shared library version.

* Spec file:

    * :boldgreen:`REVIEW-UPDATE-TO-ISSUE`: The commit log did not use
      'Update to version' or should not use 'Update to version' for release updating
    * :boldgreen:`REVIEW-CHANGELOG-ERROR`: The `rpmdev-bumpspec` was
      not used for changelog updating
    * :boldgreen:`REVIEW-SPEC-ERROR`: Spec errors

* Documentation:

    * :boldgreen:`REVIEW-DOC-MISSING`: Missing documentation, for example
      do not update ReadMe when add new parameter into helm chart.

7. Suggested learning material
##############################

Following trainings and instructions are useful:

1. Training on good commit messages https://confluence.ext.net.nokia.com/display/RCP/2019-02-28+The+git+commit+log+training+by+Jan+Zizka
2. Chris Beam's blog post about good commit messages https://chris.beams.io/posts/git-commit/
3. Checklist for RPM spec file review https://confluence.ext.net.nokia.com/display/RCP/Checklist+for+spec+file+review
4. Commit review guidelines info session https://confluence.int.net.nokia.com/display/RCP/2020-05-27+RCP+info+session%3A+5.18+Code+Review+Guideline
