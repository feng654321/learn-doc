**********************
Common review issues
**********************

.. contents:: :local:

Introduction
###############
There are common issues found during code reviews. It can help committer to avoid repeating the same issues.

Code review common issues
#########################

Useless commit log
******************

The commit log has no WHY
++++++++++++++++++++++++++++
It only describes what the change is but there is no why for this change. 
A git diff will tell you what changed, but only the commit log can properly
tell you why. So, without why the commit log is useless.

An example for this issue:
::

  Use lib http-parser directly

Please refer to `why not how <https://chris.beams.io/posts/git-commit/#why-not-how>`_.

The commit log is too general
++++++++++++++++++++++++++++++++
It looks like the commit log has WHY and WHAT already, but it does not contain
enough information. Sometimes the change itself is over a hundred lines, but the
commit log is very general like  `Fix ... for ...`. When others read this commit,
they may still have questions because the commit log is not helpful. 

A good commit log can help readers to fully understand the change.

Here are some good commit log examples. 

`Example1 <https://gerrite1.ext.net.nokia.com/c/scm_rcp/SS_CertMan/+/1223042>`_
`Example2 <https://gerrite1.ext.net.nokia.com/c/scm_rcp/SS_CertMan/+/1210336>`_

The commit log does not match the change
++++++++++++++++++++++++++++++++++++++++

**1. The commit log describes part of the whole change.**

If there are multiple corrections in one commit either split this to multiple
commits or author must describe all in one commit log.

**2. The commit log does not describe what the change is in essence.**

For example:
A change fix a memory leak bug which is found by coverity.
It is incorrect to use `Fix coverity issue` as a commit log title. Because this
change fix memory leak issue of source code but not coverity tool issue.
A correct commit log title would be `Fix a memory leak bug`.

REVIEW-LANG-ERROR
*****************

Typo
++++

There are still quite lots of commit logs with typos. Typo can be avoid by the
following ways:

In Linux environment, please use vim's spell checking.
Set vim as core editor of git command:

`git config --global core.editor=vim`

A vim console will open automatically by "git commit" command, enable spell
checking by typing `:set spell`. Then vim will highlight any typo with red color.

In Windows environment, spell checking is enable in Microsoft Word by default,
copy/paste a commit log into a Word document, the typo can be visible at once.
Or you can use other editor which support spell checking like `Vscode`.

Confusing expression
++++++++++++++++++++++++++

For example following reviewed commit message:
::

 What happened in this commit:
 1.When a new ministarter socket fd is coming,
 haagent should send the role to this ministarter
 when current role is not unknown.
 2.If haagent subscribe role notification failed,

For this commit, we suggest using "connection is accepted" wording.
"socket fd is coming" sounds like ministarter would send socket to
haagent.

Incorrect terminology
+++++++++++++++++++++
**1. Case**

`case` has multiple meanings, for example `a particular situation or a situation
of a particular type.`. But non of them means `Test case or Robot case`. 
Please use `Test Case`, `Robot case`  instead of `Case`

**2. pod**

So far `pod` in lower case is used widely in commit logs.
According to `concept of Pod <https://kubernetes.io/docs/concepts/workloads/pods/>`_
in kubernetes official website, the `Pod` in upper case is the correct terminology.

**3. code**

It can be different kinds of code, source code, error code, genetic code, security
code...
Please use `source code` instead of `code`.

REVIEW-COMMIT-NEED-SPLIT
****************************

Split formatting changes to a separate commit
+++++++++++++++++++++++++++++++++++++++++++++

Formatting changes should be done in separate commits.
The main reason is that if you mix those then when you need to revert
or cherry-pick the functional change then you would need to revert and
cherry-pick also formatting changes.
If one commit contains lots of formatting changes, it will be difficult
for the reviewers to identify the function changes.

A commit contains multiple logical changes
++++++++++++++++++++++++++++++++++++++++++

A logical change is an independent functional change. It should be done in one
commit separately. Sometimes such commit is also a big change. It is
quit difficult to do code review on such change. We need to have efficient code
review in order to get good source code quality. Code review is efficient only if the
changes are small enough, less than 200 lines.

In Gerrit one commit equals one code review. In GitLab one merge request equals
one code review. The same FCI branch can have multiple commits.
No feature can be done in less than 200 lines, so one feature will require multiple
commits or merge requests. So, we need to be able to merge changes to trunk in small pieces.

`audilog <https://gitlabe2.ext.net.nokia.com/rcp-security/auditlog>`_ subsystem
is a very good example to show how a big feature was built by multiple small commits.
Please also refer to the related `yammar link <https://web.yammer.com/main/threads/eyJfdHlwZSI6IlRocmVhZCIsImlkIjoiNjM0Mzk4MjI0Njc4OTEyIn0>`_.

Resolve comment thread without reviewer's permission
****************************************************

Normally comment threads should be resolved by reviewers. If author wants to
resolve them by his/her own, then the author must leave a message to explain the
reason.

In gerrit, author can notify reviewer to indicate the comment has been handled by
`Done` button. And author must always remember to uncheck `Resolved` so that the
thread will not be resolved by author. Then reviewer should verify changes
and accept them by marking the comment thread as Resolved by `Ack` button.

REVIEW-FORMAT-ERROR
***********************

Go indentation error
++++++++++++++++++++++++++

Go standard is to use TABs as indentation. So you could consider
using gofmt to generate TABs instead of adding spaces manually.

UT issues
*************

Missing checkpoints
+++++++++++++++++++++++++

Some unit tests are not checking result, just for code coverage. These unit tests 
are meaningless.

For example:
::

 type LoginRequestBody struct {
        ReadOnlyMode bool
 }

 func Test_GetLoginRequestBody(t *testing.T) {
        var requestBody LoginRequestBody
        requestBody.ReadOnlyMode = false
        reqBody, _ := json.Marshal(requestBody)
        req := httptest.NewRequest(
                http.MethodGet,
                "/",
                bytes.NewReader(reqBody),
        )
        loginBody, err := GetLoginRequestBody(req)
        fmt.Printf("loginBody = %v \n", loginBody)
        fmt.Printf("err = %v \n", err)
 }

We can see the UT code is not check loginBody and err.

Hard to read and understand UT case title
+++++++++++++++++++++++++++++++++++++++++++++++

The title of UT case is useless, like case1,case2,case3,etc. 
We don't know what the UT case did by the title.
The title of UT case should point out what the case have done.

Instead of list just mentioning a test condition, the test case titles should use languages 
that clearly defines the expected behavior. That way the test titles create an up-to-date software 
implementation specification and you don't need to look at the test to figure out 
what the expected behavior is (also an error in intended / actual test would be easier to spot).

For example:

::

 func Test_generateURL(t *testing.T) {
        tests := []struct {
                name        string
                addr        string
                port        string
                expectation string
        }{
                {
                        name:        "case1",
                        addr:        "192.168.0.1",
                        port:        "8080",
                        expectation: `{"url":"http://192.168.0.1:8080/api/sec/v1/symptom"}`,
                },
                {
                        name:        "case2",
                        addr:        "2001:db8:4:0:f2d0:7ffc:308d:9fd4",
                        port:        "8080",
                        expectation: `{"url":"http://[2001:db8:4:0:f2d0:7ffc:308d:9fd4]:8080/api/sec/v1/symptom"}`,
                },
        }

        for _, tt := range tests {
                t.Run(tt.name, func(t *testing.T) {
                        if got := generateURL(tt.addr, tt.port, defaultApi); got != tt.expectation {
                                t.Errorf("generateURL() = %v ,Expectation = %v", got, tt.expectation)
                        }
                })
        }
 }

Use "generateUrlSuccessInIPV4Format" as the test case title instead of "case1".

Use "generateUrlSuccessInIPV6Format" as the test case title instead of "case2".

Want a deeper understanding, please refer to this link for "unit test naming":

https://qualitycoding.org/unit-test-naming/

Define global variables for UT
++++++++++++++++++++++++++++++++++++

There are many UT functions to use the same global variable in one test fixture.  
It is hard to ensure maintainability.
Each UT who use the global variable should remember to reset it in its teardown 
otherwise it will influence other cases.
And it is very easy to make mistakes when someone write new UT code.
It is best to define local variables in UT functions.

For example:
::

 var Token1 tokenInfo = tokenInfo{
        Type:                   "Type",
        Token:                  "Token1",
        Expires:                "Expires",
        LastLogin:              "LastLogin",
        LastFailedLogin:        "LastFailedLogin",
        FailedLoginAttempts:    "FailedLoginAttempts",
        PasswordExpirationDate: "PasswordExpirationDate",
        AdditionalText:         "AdditionalText",
        ReadOnlyMode:           false}

 func callback(token *tokenInfo) error {
        token.Type = "expireToken"
        return nil
 }

 func Test_startExpire(t *testing.T) {
        tests := []struct {
                name        string
                token       *tokenInfo
                expectValue string
        }{
                {
                        name:        "case1",
                        token:       &Token1,
                        expectValue: "expireToken",
                },
        }
        for _, tt := range tests {
                extendTokenTimeOut, _ := time.ParseDuration("3s")
                extendTokenTimeOut1, _ := time.ParseDuration("5s")
                tt.token.startExpire(extendTokenTimeOut, callback)
                time.Sleep(extendTokenTimeOut1)
                if tt.expectValue != tt.token.Type {
                        t.Error("test start expire failed")
                }
                tt.token.Type = "Type"
        }

 }

 func Test_stopExpire(t *testing.T) {
        tests := []struct {
                name        string
                token       *tokenInfo
                expectValue string
        }{
                {
                        name:        "case1",
                        token:       &Token1,
                        expectValue: "expireToken",
                },
        }
        for _, tt := range tests {
                extendTokenTimeOut, _ := time.ParseDuration("3s")
                tt.token.startExpire(extendTokenTimeOut, callback)
                tt.token.stopExpire()
                time.Sleep(extendTokenTimeOut)
                if tt.expectValue == tt.token.Type {
                        t.Error("test stop expire failed")
                }
                tt.token.Type = "Type"
        }
 }

Too many checkpoints in a test
++++++++++++++++++++++++++++++++++++

There are too many test points in one UT function. But We cannot distinguish them from each other.
So the test result is not clear enough. For example, when we write GO UT, We can use **t.run** to do sub test, 
and the test result is more clear. We also can split the UT function into several UT functions.

For example:

UT code before re-factoring:

::

 func TestValidateReqWithUserName(t *testing.T) {
        req := &http.Request{}
        roles := []string{"tst"}

        patchs := ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                return nil, errors.Errorf("Get user from request failed.")
        })
        usr, msg, err, _ := ValidateReqWithUserName(req, roles)
        assert.NotNil(t, err)
        assert.Equal(t, "", usr)
        assert.Equal(t, "Failed to get the user from the request.", msg)
        patchs.Reset()

        patchs.ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                return &userInfo{Username: "tst"}, nil
        })
        patchs.ApplyFunc(isUserAllowed, func(_ string, _ []string) (bool, error, int) {
                return false, errors.Errorf("User not allowed."), 500
        })
        usr, msg, err, _ = ValidateReqWithUserName(req, roles)
        assert.NotNil(t, err)
        assert.Equal(t, "tst", usr)
        assert.Equal(t, "Authorization Failed. Failed to check roles from UM.", msg)
        patchs.Reset()

        patchs.ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                return &userInfo{Username: "tst"}, nil
        })
        patchs.ApplyFunc(isUserAllowed, func(_ string, _ []string) (bool, error, int) {
                return false, errors.New("Insufficient Access Rights."), 405
        })
        usr, msg, err, _ = ValidateReqWithUserName(req, roles)
        assert.NotNil(t, err)
        assert.Equal(t, "tst", usr)
        assert.Equal(t, "Authorization Failed, insufficient Access Rights.", msg)
        patchs.Reset()

        patchs.ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                return &userInfo{Username: "tst"}, nil
        })
        patchs.ApplyFunc(isUserAllowed, func(_ string, _ []string) (bool, error, int) {
                return true, nil, 200
        })
        usr, msg, err, _ = ValidateReqWithUserName(req, roles)
        assert.Nil(t, err)
        assert.Equal(t, "tst", usr)
        assert.Equal(t, "Authorization Success.", msg)
        patchs.Reset()
 }

UT code after re-factoring:

::

 func TestValidateReqWithUserName(t *testing.T) {
        req := &http.Request{}
        roles := []string{"tst"}

        t.Run("ValidateReqWithUserNameFailIfRequestHasNoUser", func(t *testing.T) {
            patchs := ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                    return nil, errors.Errorf("Get user from request failed.")
            })
            usr, msg, err, _ := ValidateReqWithUserName(req, roles)
            assert.NotNil(t, err)
            assert.Equal(t, "", usr)
            assert.Equal(t, "Failed to get the user from the request.", msg)
            patchs.Reset()
        })

        t.Run("ValidateReqWithUserNameFailIfUserNotAllowed", func(t *testing.T) {
            patchs.ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                    return &userInfo{Username: "tst"}, nil
            })
            patchs.ApplyFunc(isUserAllowed, func(_ string, _ []string) (bool, error, int) {
                    return false, errors.Errorf("User not allowed."), 500
            })
            usr, msg, err, _ = ValidateReqWithUserName(req, roles)
            assert.NotNil(t, err)
            assert.Equal(t, "tst", usr)
            assert.Equal(t, "Authorization Failed. Failed to check roles from UM.", msg)
            patchs.Reset()
        })

        t.Run("ValidateReqWithUserNameFailIfUserHaveInsufficientAccessRights", func(t *testing.T) {
            patchs.ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                    return &userInfo{Username: "tst"}, nil
            })
            patchs.ApplyFunc(isUserAllowed, func(_ string, _ []string) (bool, error, int) {
                    return false, errors.New("Insufficient Access Rights."), 405
            })
            usr, msg, err, _ = ValidateReqWithUserName(req, roles)
            assert.NotNil(t, err)
            assert.Equal(t, "tst", usr)
            assert.Equal(t, "Authorization Failed, insufficient Access Rights.", msg)
            patchs.Reset()
        })

        t.Run("ValidateReqWithUserNameSuccess", func(t *testing.T) {
            patchs.ApplyFunc(getUserFromRequest, func(_ *http.Request) (*userInfo, error) {
                    return &userInfo{Username: "tst"}, nil
            })
            patchs.ApplyFunc(isUserAllowed, func(_ string, _ []string) (bool, error, int) {
                    return true, nil, 200
            })
            usr, msg, err, _ = ValidateReqWithUserName(req, roles)
            assert.Nil(t, err)
            assert.Equal(t, "tst", usr)
            assert.Equal(t, "Authorization Success.", msg)
            patchs.Reset()
        })
 }

UT result before re-factoring:

::

 === RUN TestValidateReqWithUserName
 --- PASS: TestValidateReqWithUserName (0.00s)

UT result after re-factoring:

::

 === RUN TestValidateReqWithUserName
 === RUN TestValidateReqWithUserName/ValidateReqWithUserNameFailIfRequestHasNoUser
 === RUN TestValidateReqWithUserName/ValidateReqWithUserNameFailIfUserNotAllowed
 === RUN TestValidateReqWithUserName/ValidateReqWithUserNameFailIfUserHaveInsufficientAccessRights
 === RUN TestValidateReqWithUserName/ValidateReqWithUserNameSuccess
 --- PASS: TestValidateReqWithUserName (0.00s)
     --- PASS: TestValidateReqWithUserName/ValidateReqWithUserNameFailIfRequestHasNoUser (0.00s)
     --- PASS: TestValidateReqWithUserName/ValidateReqWithUserNameFailIfUserNotAllowed (0.00s)
     --- PASS: TestValidateReqWithUserName/ValidateReqWithUserNameFailIfUserHaveInsufficientAccessRights (0.00s)
     --- PASS: TestValidateReqWithUserName/ValidateReqWithUserNameSuccess (0.00s)

We can see the UT result is more clear than before.
