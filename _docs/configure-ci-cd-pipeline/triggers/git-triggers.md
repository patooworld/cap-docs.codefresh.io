---
title: "Git Trigger"
description: ""
group: configure-ci-cd-pipeline
sub_group: triggers
toc: true
---

GIT triggers are the most basic types of trigger for performing [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) with Codefresh.

At the trigger level you have the option of selecting

 * Which code repository will be used as a trigger
 * which branches will be affected by a pipeline
 * If a trigger will apply to a Pull Request or not

 Note that you can select another repository other than the one the project itself belongs to. It is possible
 to trigger a build on project A even though a commit happened on project B.

You can also use [Conditional expressions]({{ site.baseurl }}/docs/codefresh-yaml/conditional-execution-of-steps/) at the pipeline level to further fine-tune the way specific steps (or other transitive pipelines) are executed.


## Manage GIT Triggers with Codefresh UI

To add a new GIT trigger, navigate to the Codefresh Pipeline *Configuration* view and expand the *Triggers* section. Press the *Add Trigger* button and select a *GIT* trigger type to add.

{% include image.html
lightbox="true"
file="/images/pipeline/triggers/add-trigger-dialog.png"
url="/images/pipeline/triggers/add-trigger-dialog.png"
alt="Adding new Trigger dialog"
max-width="60%"
%}

You can select the following information:

* *Git Provider* - select that one that is linked with your account
* *Repository* - You can select any repository even something different than the one that is used for the code checkout
* *Commit checkbox* - If enabled will trigger this pipeline for any commit
* *PR checkbox* - If enabled will trigger for any events that belong to a pull request
* *Branch field* - This is a regular expression and will only trigger for branches that match this naming pattern
* *Modified files* - This allows you to constrain the build and trigger it only if the modified files from the commit match this [glob expression](https://en.wikipedia.org/wiki/Glob_(programming))

{% include image.html
lightbox="true"
file="/images/pipeline/triggers/add-git-trigger.png"
url="/images/pipeline/triggers/add-git-trigger.png"
alt="Adding GIT Trigger"
max-width="60%"
%}

The commit checkbox (by default it is enabled) means that this pipeline will run for *any* commit as long as its source branch matches the naming scheme. This includes commits on pull requests.

The PR checkbox (by default it is disabled) means that this pipeline will run only on events that happen on a Pull Request. This includes opening the Pull Request itself as well as any commits that happen to the branch while a Pull request is active.


In most cases you will have either of these checkboxes active. Enabling both at the same time will result in some duplicate builds (e.g. when commits happen to pull request branches).
The concept behind these two checkboxes (and in conjunction with the branch text field) is to allow you to define which pipelines run for various workflows in your organization.

As a simple example you can have a *production* pipeline that runs only on *master* branch (and therefore the branch field says "master") and a *testing* pipeline that runs user acceptance tests where only the Pull Request checkbox is active. This means that User Acceptance tests will run whenever a PR is created or modified. Once it is merged the p*roduction* pipeline will deploy the changes.

In a more advanced example you could add regular expressions in the branch field with names such as *feature-*, *hotfix-* etc and the PR checkbox active on different pipelines. This way you could trigger the pull requests only when they happen on specific branches. So a developer that creates a temporary feature with a name that doesn't match these naming patterns will not trigger those pipelines.

>If the PR checkbox is enabled, Codefresh will also trigger a pipeline if a pull request is rejected/closed. While at first glance this seems counterintuitive, in practice it allows you to tear down test environments when the code from a Pull request is no longer needed. See below for a way to decide exactly what types of webhook events are used for triggers.

The *modified files* field is a very powerful Codefresh feature that allows you to trigger a build only if the
files affected by a commit are in a specific folder (or match a specific naming pattern). This means that
you can have a big GIT repository with multiple projects and build only the parts that actually change.

>Currently the field *modified files* is available only for Github repositories, since Github is the only GIT provider
that sends this information in the webhook. We will support other GIT providers as soon as they add the respective feature.

## Using the Modified files field to constrain triggers to specific folder/files

The *modified field* accepts glob expressions. The paths are relative to the root folder of the project (where the git repository was checked out). Some possible examples are:

```
**/package.json
**/Dockerfile*
my-subproject/**
my-subproject/sub-subproject/package.json
my-subproject/**/pom.xml

```

>You can also use relative paths with dot-slash. Therefore `./package.json` and `package.json` are exactly the same thing. They both refer to the file `package.json` found at the root of the git project that was checked out as part of the build.

Once a commit happens to a code repository, Codefresh will see which files are changed from the git provider and trigger the build **only** if the changed files match the glob expression. If there is no match no build will be triggered.

This is a very useful feature for organizations who have chosen to have multiple projects on the same GIT repository (monorepos). Let's assume for example that a single system has a Java backend, a NestJS frontend and a Ruby-on-Rails internal dashboard.

{% include image.html
lightbox="true"
file="/images/pipeline/triggers/monorepo.png"
url="/images/pipeline/triggers/monorepo.png"
alt="GIT monorepo"
max-width="60%"
%}

Now we can define 3 different pipelines in Codefresh where each one builds the respective project

{% include image.html
lightbox="true"
file="/images/pipeline/triggers/monorepo-pipelines.png"
url="/images/pipeline/triggers/monorepo-pipelines.png"
alt="GIT monorepo pipelines"
max-width="70%"
%}

And then in the GIT trigger for each one we set the modified files field to the following values

* For the *build-nestjs-only* pipeline *MODIFIED FILES* has `my-nestjs-project/**`
* For the *build-java-only* pipeline *MODIFIED FILES* has `my-java-project/**`
* For the *build-rails-only* pipeline *MODIFIED FILES* has `my-rails-project/**`

This way as multiple developers work on the git repository only the affected projects will actually build. A change to the NestJS project will *not* build the Rails project as well. Also if somebody changes *only* the README file and nothing else, no build will be triggered at all (which is a good thing as the source code is exactly the same).

You can also use Glob expressions for files. For example

*  an expression such `my-subproject/sub-subproject/package.json` will trigger a build **only** if the dependencies of this specific project are changed. 
* A pipeline with the expression `my-subproject/**/pom.xml` will trigger only if the Java dependencies for any project that belongs to `my-subproject` actually change. 

Glob expressions have many more options not shown here. Visit the [official documentation](https://en.wikipedia.org/wiki/Glob_(programming)) to learn more. You can also use the [Glob Tester web application](http://www.globtester.com/) to test your glob expressions beforehand so that you are certain they match the 
files you expect them to match.



## Using YAML and the Codefresh CLI to filter specific Webhook events

The default GUI options exposed by Codefresh are just a starting point for GIT triggers and pull requests. Using [Codefresh YAML]({{ site.baseurl }}/docs/codefresh-yaml/what-is-the-codefresh-yaml/) and the [Codefresh CLI plugin](https://codefresh-io.github.io/cli/) you can further create two-phase pipelines where the first one decides
which webwook events will be honored and the second one contains the actual build.


{% include image.html
lightbox="true"
file="/images/pipeline/triggers/two-phase-pipeline.png"
url="/images/pipeline/triggers/two-phase-pipeline.png"
alt="Two phase pipeline"
max-width="80%"
%}

The generic GIT trigger is placed on Pipeline A. This pipeline then filters the applicable webooks using [conditional expressions]({{ site.baseurl }}/docs/codefresh-yaml/conditional-execution-of-steps/). Then it uses the Codefresh CLI plugin (and specifically the [run pipeline capability](https://codefresh-io.github.io/cli/pipelines/run-pipeline/)) to trigger pipeline B that performs build.

Some of the YAML variables that you might find useful (from the [full list]({{ site.baseurl }}/docs/codefresh-yaml/variables/))

* `CF_PULL_REQUEST_ACTION` - open, close, accept etc
* `CF_PULL_REQUEST_TARGET` - target branch of the pull request
* `CF_BRANCH` - the branch that contains the pull request

As an example, here is the `codefresh.yml` file of pipeline A where we want to run pipeline B only when a Pull Requested is opened against a branch named *production*.


`codefresh.yml` of pipeline A
{% highlight yaml %}
{% raw %}
version: '1.0'
steps:
  triggerstep:
    title: trigger
    image: codefresh/cli
    commands:
      - 'codefresh run <pipeline_B> -b=${{CF_BRANCH}}'
    when:
      condition:
        all:
          validateTargetBranch: '"${{CF_PULL_REQUEST_TARGET}}" == "production"'
          validatePRAction: '''${{CF_PULL_REQUEST_ACTION}}'' == ''opened'''
{% endraw %}
{% endhighlight %}


This is the build definition for the first pipeline that has a GIT trigger (with the Pull request checkbox enabled).
It has only a single step which uses conditionals that check the name of the branch where the pull request is targeted to, as well as the pull request action. Only if *both* of these conditions are true then the build step is executed.

The build step calls the second pipeline. The end result is that pipeline B runs only when the Pull Request is opened the first time. Any further commits on the pull request branch will **not** trigger pipeline B (pipeline A will still run but the conditionals will fail).

