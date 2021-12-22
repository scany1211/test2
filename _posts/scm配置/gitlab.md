![image-20211118164611830](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211118164611830.png)

![image-20211118172234880](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211118172234880.png)

# 需要安装的plugin

gitlab plugin

gitlab branch source

# 配置

## 1. Jenkins中添加gitlabserver

### `Name` 

 Plugin automatically generates an unique server name for you. User may want to configure this field to suit their needs but should make sure it is sufficiently unique. We recommend to keep it as it is.

### `Server URL` 

 Contains the URL to your GitLab Server. By default it is set to "[https://gitlab.com](https://gitlab.com/)". User can modify it to enter their GitLab Server URL e.g. https://gitlab.gnome.org/, [http://gitlab.example.com:7990](http://gitlab.example.com:7990/). etc.

###  `Credentials` 

Contains a list of credentials entries that are of type GitLab Personal Access Token. When no credential has been added it shows "-none-". User can add a credential by clicking "Add" button.

### Mange Web Hook

If you want the plugin to setup web hook on your GitLab project(s) to get push/mr/tag/note events then check this box.

### Mange System Hook

 If you want the plugin to setup system hook on your GitLab project(s) to detect if a project is removed then check this box. Remember plugin can only setup system hook on your server if supplied access token has `Admin` access.

### Secret Token

 The secret token is required to authenticate the webhook payloads received from GitLab Server. Use generate secret token from Advanced options or use your own. If you are a old plugin user and did not set a secret token previously and want secret token to applied to the hooks of your existing jobs, you can add the secret token and rescan your jobs. Existing hooks with new secret token will be applied.

### Root URL for hooks

 `By default Root URL for hooks created by this plugin is your Jenkins instance url. You can modify the root URL in by adding your custom root URL. Leave empty if you want Jenkins URL to be your custom hook url. A path is added to your hook ROOT URL `/gitlab-webhook/post` for webhooks and `/gitlab-systemhook/post` for system hooks.



## 2. Jenkins中配置gitlab token

(1) gitlab中通过如下步骤创建 `Personal Access Token` 

```
a. Select profile dropdown menu from top-right corner

b. Select `Settings`

c. Select `Access Token` from left column

d. Enter a name | Set Scope to `api` (If admin also give `sudo` which required for systemhooks and mr comment trigger)

e. Select `Create Personal Access Token`

f. Copy the token generated
```



（2）Return to Jenkins | Select `Add` in Credentials field | Select `Jenkins`.

（3）Set `Kind` to GitLab Personal Access Token.

（4） Enter `Token`.

（5）Enter a unique id in `ID`.

（6）Enter a human readable description.

![gitlab-credentials](https://cdn.jsdelivr.net/gh/jenkinsci/gitlab-branch-source-plugin@master/docs/img/gitlab-credentials.png)



​	在jenkins中创建server时添加token：

1. Select `Advanced` at the bottom of `GitLab` Section.

2. Select `Manage Additional GitLab Actions`.

3. Select `Convert login and password to token`.

4. Set the `GitLab Server URL`.

5. There are 2 options to generate token:

   i. `From credentials` - To select an already persisting Username Password Credentials or add an Username Password credential to persist it.

   ii. `From login and password` - If this is a one time thing then you can directly enter you credentials to the text boxes and the username/password credential is not persisted.

6. After setting your username/password credential, select `Create token credentials`.

7. The token creator will create a Personal Access Token in your GitLab Server for the given user with the required scope and also create a credentials for the same inside Jenkins server. You can go back to the GitLab Server Configuration to select the new credentials generated (select "-none-" first then new credentials will appear). For security reasons this token is not revealed as plain text rather returns an `id`. It is a 128-bit long UUID-4 string (36 characters).

   [![gitlab-token-creator](https://cdn.jsdelivr.net/gh/jenkinsci/gitlab-branch-source-plugin@master/docs/img/gitlab-token-creator.png)](https://github.com/jenkinsci/gitlab-branch-source-plugin/blob/master/docs/img/gitlab-token-creator.png)



## 3. 在gitlab上创建webhook



GitLab Server.中的如下配置中， `Jenkins Url` 需要是domain name (FQDN) 不能是 `localhost`.

#### WebHook

```
<jenkins_url>/gitlab-webhook/post
```

with `push`, `tag`, `merge request` and `note` events.

#### SystemHook

```
<jenkins_url>/gitlab-systemhook/post
```

with `repository update` event.



# Multibranch Pipeline Jobs

The Multibranch Pipeline job type enables you to implement different Jenkinsfiles for different branches of the same project. In a Multibranch Pipeline job, Jenkins automatically discovers, manages and executes Pipelines for Branches/Merge Requests/Tags which contain a `Jenkinsfile` in source control. This eliminates the need for manual Pipeline creation and management.

To create a `Multibranch Pipeline Job`:

1. Select `New Item` on Jenkins home page.

2. Enter a name for your job, select `Multibranch Pipeline` | select `Ok`.

3. In `Branch Sources` sections, select `Add source` | select `GitLab Project`.

4. Now you need to configure your jobs.

   [![branch-source](https://cdn.jsdelivr.net/gh/jenkinsci/gitlab-branch-source-plugin@master/docs/img/branch-source.png)](https://github.com/jenkinsci/gitlab-branch-source-plugin/blob/master/docs/img/branch-source.png)

   i. Select `Server` configured in the initial server setup.

   ii. [Optional] Add `Checkout Credentials` (SSHPrivateKey or Username/Password) if there is any private projects that will be built by the plugin.

   iii. Add path to the owner where the project you want to build exists. If user, enter `username`. If group, enter `group name`. If subgroup, enter `subgroup path with namespace`.您可以添加在您的 Owner（用户/组/子组）中所有项目。 表单验证将与 GitLab 服务器检查 owner 是否有效。 您可以添加 `Discover subgroup project` 的特性，该特性允许您发现组或子组中所有子组的子项目，但此特性不适用于用户

   iv. Based on the owner provided. All the projects are discovered in the path and added to the `Projects` listbox. You can now choose the project you want to build.

   v. `Behaviours` (a.k.a. SCM Traits) allow different configurations option to your build. More about it in the SCM Trait APIs section.

5. Now you can go ahead and save the job.

For more info see [this](https://jenkins.io/doc/book/pipeline/multibranch/).

After saving, a new web hook is created in your GitLab Server if a `GitLab Access Token` is specified in the server configuration. Then the branch indexing starts based on what options behaviours you selected. As the indexing proceeds new jobs are started and queued for each branches with a `Jenkinsfile` in their root directory.

The Job results are notified to the GitLab Server as Pipeline Status for the HEAD commit of each branches built. The build for forked MR cannot be notified to GitLab Server as GitLab doesn't provide Pipeline status for Merge Requests from forks for security concerns. See [this](https://docs.gitlab.com/ee/ci/merge_request_pipelines/#important-notes-about-merge-requests-from-forked-projects).

We have a workaround for this. Jenkins will build the MRs from forked projects if the MR author is a trusted owner i.e. has `Developer`/`Maintainer`/`Owner` access level. More about it in the SCM Trait APIs section.

As the web hook is now setup on your Jenkins CI by the GitLab server. Any push-events or merge-request events or tag events trigger the concerned build in Jenkins.

# Folder Organization

Folders Organization enable Jenkins to monitor an entire GitLab `User`/`Group`/`Subgroup` and automatically create new Multibranch Pipelines for projects which contain branches/merge requests/tags containing a `Jenkinsfile`. In our plugin this type of job is called `GitLab Group`.

To create a `GitLab Group Job`:

1. Select `New Item` on Jenkins home page.

2. Enter a name for your job, select `GitLab Group` | select `Ok`.

3. Now you need to configure your jobs.

   i. Select `Server` configured in the initial server setup.

   ii. [Optional] Add `Checkout Credentials` (SSHPrivateKey or Username/Password) only if there are any private projects required to be built.

   iii. Add path to the owner whose projects you want to build. If user, enter `username`. If group, enter `group name`. If subgroup, enter `subgroup path with namespace`.

   v. `Behaviours` (a.k.a. SCM Traits) are allow different configuration option to your build. More about it in the SCM Trait APIs section.

The indexing in this group job type only needs to discover one branch with`Jenkinsfile` and thus it only shows the partial indexing log. You need to visit individual projects to see their full indexing.

# SCM Trait APIs

The following behaviours apply to both `Multibranch Pipeline Jobs` and `Folder Organization` (unless otherwise stated).

### 1. Default Traits:

实现对项目合并请求的支持具有挑战性。 第一，MR 有两种类型，即原始分支和 Fork 的项目分支，因此每个 head 必须有不同的实现。 第二，来自 fork 的 MR 可能来自不可信的源，所以实现了一种新的策略 `Trust Members`，它允许 CI 仅从具有 `Developer`/`Maintainer`/`Owner` 访问级别的可信用户构建 MR。

第三，来自 fork 的 MR 由于 GitLab 的问题不支持流水线状态通知，请参考[这里](https://docs.gitlab.com/ee/ci/merge_request_pipelines/#important-notes-about-merge-requests-from-forked-projects)。 您可以添加一个特性 `Log Build Status as Comment on GitLab` ，它允许那您添加一个 sudo 用户（如果你希望 owner 用户为空）以在 commit/tag/mr 上对构建结果进行评论。 要添加 sudo 用户，令牌必须具有管理访问权限。 默认情况下，只有失败/出错以评论的形式被记录，但是您也可以通过勾选复选框来启用成功构建的日志记录。

- `Discover branches` - To discover branches.

  - `Only Branches that are not also filed as MRs` - If you are discovering origin merge requests, it may not make sense to discover the same changes both as a merge request and as a branch.
  - `Only Branches that are filed as MRs` - This option exists to preserve legacy behaviour when upgrading from older versions of the plugin. NOTE: If you have an actual use case for this option please file a merge request against this text.
  - `All Branches` - Ignores whether the branch is also filed as a merge request and instead discovers all branches on the origin project.

- `Discover merge requests from origin` - To discover merge requests made from origin branches.

  - `Merging the merge request merged with current target revision` - Discover each merge request once with the discovered revision corresponding to the result of merging with the current revision of the target branch.
  - `The current merge request revision` - Discover each merge request once with the discovered revision corresponding to the merge request head revision without merging.
  - `Both current mr revision and the mr merged with current target revision` - Discover each merge request twice. The first discovered revision corresponds to the result of merging with the current revision of the target branch in each scan. The second parallel discovered revision corresponds to the merge request head revision without merging.

- `Discover merge requests from forks` - To discover merge requests made from forked project branches.

  - Strategy:
    - `Merging the merge request merged with current target revision` - Discover each merge request once with the discovered revision corresponding to the result of merging with the current revision of the target branch.
    - `The current merge request revision` - Discover each merge request once with the discovered revision corresponding to the merge request head revision without merging.
    - `Both current mr revision and the mr merged with current target revision` - Discover each merge request twice. The first discovered revision corresponds to the result of merging with the current revision of the target branch in each scan. The second parallel discovered revision corresponds to the merge request head revision without merging.
  - Trust
    - `Members` - Discover MRs from Forked Projects whose author is a member of the origin project.
    - `Trusted Members` - [Recommended] Discover MRs from Forked Projects whose author is has Developer/Maintainer/Owner accesslevel in the origin project.
    - `Everyone` - Discover MRs from Forked Projects filed by anybody. For security reasons you should never use this option. It may be used to reveal your Pipeline secrets environment variables.
    - `Nobody` - Discover no MRs from Forked Projects at all. Equivalent to removing the trait altogether.

  If `Members` or `Trusted Members` is selected, then plugin will build the target branch of MRs from non/untrusted members.

### 2. Additional Traits:

These traits can be selected by selecting `Add` in the `Behaviours` section.

- `Tag discovery` - Discover tags in the project. To automatically build tags install `basic-branch-build-plugin`.
- `Discover group/subgroup projects` - Discover subgroup projects inside a group/subgroup. Only applicable to `GitLab Group` Job type whose owner is a `Group`/`Subgroup` but not `User`.
- `Log build status as comment on GitLab` - Enable logging build status as comment on GitLab. A comment is logged on the commit or merge request once the build is completed. You can decide if you want to log success builds or not. You can also use sudo user to comment the build status as commment e.g. `jenkinsadmin` or something similar.
- `Trigger build on merge request comment` - Enable trigger a rebuild of a merge request by comment with your desired comment body (default: `jenkins rebuild`). The job can only be triggered by trusted members of the project i.e. users with Developer/Maintainer/Owner accesslevel (also includes inherited from ancestor groups). By default only trusted members of project can trigger MR. You may want to disable this option because trusted members do not include members inherited from shared group (there is no way to get it from GitLabApi as of GitLab 13.0.0). If disabled, MR comment trigger can be done by any user having access to your project.
- `Disable GitLab project avatar` - Disable avatars of GitLab project(s). It is not possible to fetch avatars when API has no token authentication or project is private. So you may use this option as a workaround. We will fix this issue in a later release.
- `Project Naming Strategy` - Choose whether you want `project name` or the `project path (with namespace)` as job names of each project. Users generally prefer the first option but due to legacy reasons we have `project path (with namespace)` as default naming scheme. Note if a job is already created and the naming strategy is changed it will cause projects and build logs to be destroyed.
- `Filter by name (with regex)` - Filter the type of items you want to discover in your project based on the regular expression specified. For example, to discover only `master` branch, `develop` branch and all Merge Requests add `(master|develop|MR-.*)`.
- `Filter by name (with wildcards)` - Filter the type of items you want to discover in your project based on the wildcards specified. For example, to discover only `master` branch, `develop` branch and all Merge Requests add `development master MR-*`.
- `Skip pipeline status notifications` - Disable notifying GitLab server about the pipeline status.
- `Override hook management modes` - Override default hook management mode of web hook and system hook. `ITEM` credentials for webhook is currently not supported.
- `Checkout over SSH` - [Not Recommended] Use this mode to checkout over SSH. Use `Checkout Credentials` instead.

# Environment Variables

By default Multibranch Jobs have the following environment variables (provided by Branch API Plugin):

Branch - `BRANCH_NAME`

Merge Request - `BRANCH_NAME`, `CHANGE_ID`, `CHANGE_TARGET`, `CHANGE_BRANCH`, `CHANGE_FORK`, `CHANGE_URL`, `CHANGE_AUTHOR`, `CHANGE_TITLE`. `CHANGE_AUTHOR_DISPLAY_NAME`

Tag - `BRANCH_NAME`, `TAG_NAME`, `TAG_TIMESTAMP`, `TAG_DATE`, `TAG_UNIXTIME`

This plugin adds a few more environment variables to Builds (`WorkflowRun` type only) which is the payload received as WebHook) See https://docs.gitlab.com/ee/user/project/integrations/webhooks.html#events.

A few points to note:

> If no response is recorded for any field in the Web Hook Payload, it returns an empty String. To add more variables see `package io.jenkins.plugins.gitlabbranchsource.Cause`.
>
> `GITLAB_OBJECT_KIND` - This environment variable should be used to check the event type before accessing the environment variables. Possible values are `none`, `push`, `tag_push` and `merge_request`.
>
> Any variables ending with `#` indicates the index of the list of the payload starting from 1.

Environment Variables are available from Push Event, Tag Push Event and Merge Request Event.