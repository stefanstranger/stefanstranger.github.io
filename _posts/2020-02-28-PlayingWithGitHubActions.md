---
layout: post
title: Playing with Github Actions
categories: [CI/CD]
tags: [CI/CD]
comments: true
---

After using Azure DevOps Yaml Pipelines for some time now I wanted to learn more about Github's Actions.

# Introduction

According to the Github documentation, GitHub Actions makes it easy to automate all your software workflows, with world-class CI/CD. Build, test, and deploy your code right from GitHub and code reviews, branch management, and issue triaging work the way you want.

Sounds interesting, let's explore more about Github Actions in this blog post.

## Github Action terminology

When working with Github Actions the following terminology is used. 

### Workflows

Workflows are custom automated processes that you can set up in your repository to build, test, package, release, or deploy any project on GitHub.

### Workflow jobs

A workflow job is a defined task made up of steps.

### Workflow runs

A workflow run is an instance of your workflow that runs when the pre-configured event occurs.

### Runners

The runner is the application that runs a job from a GitHub Actions workflow. The runner can run on the hosted machine pools or run on self-hosted environments.

### Actions

Actions are individual tasks that you can combine to create jobs and customize your workflow. You can create your own actions, and use and customize actions shared by the GitHub community.

## Marketplace

GitHub Marketplace is a central location for you to find actions created by the GitHub community.

Actions created by verified organizations have a badge to show that GitHub has verified the creator of the action. For more information, see "[Verifying your organization's domain.](https://help.github.com/en/github/setting-up-and-managing-organizations-and-teams/verifying-your-organizations-domain)"

You can discover new actions from the workflow editor on GitHub, and from the [GitHub Marketplace page](https://github.com/marketplace/actions/).

## Get started

To get started go to your Github Account open or create a new Repository and click on Actions tab.

![](/assets/hellowordgithubaction.gif)


```yaml
name: CI

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v1
    - name: Run a one-line script
      run: echo Hello, world!
    - name: Run a multi-line script
      run: |
        echo Add other actions to build,
        echo test, and deploy your project.
```

Let's change this to a Hello World PowerShell version which can be triggered via a external webhook:

```yaml
name: Hello World

on: [repository_dispatch]

jobs:
  build:

    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v1
    - name: Run PowerShell Hello World script
      run: Write-Output 'Hello World!'
    - name: Run a multi-line PowerShell script
      if: github.event.action == 'demo'
      run: |
        $psversiontable;
        Get-Process;
```

Result when above Github Action workflow is run:

![](/assets/2020-01-28_16-44-00.png)


## Explanation of Workflow

| Property | Description                                             | Comments                                                                                                                                                                                                    |
| -------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name     | The name of your workflow                               |                                                                                                                                                                                                             |
| on       | The name of the GitHub event that triggers the workflow | You can use the GitHub API to trigger a webhook event called **repository_dispatch** when you want to trigger a workflow for activity that happens outside of GitHub.                                       |
| jobs:    | A workflow run is made up of one or more jobs           |                                                                                                                                                                                                             |
| build:   |                                                         |
| runs-on  | Specifies the GitHub-hosted runner                      | On the [windows-latest](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners#windows-server-2019) runs PowerShell Core version 6.2.3 |
| steps    |                                                         |
| name     |                                                         |
| run      |                                                         |
|          |                                                         |


**References:**
* [Github Actions documentation](https://github.com/features/actions)
* [Help documentation Github Action](https://help.github.com/en/actions/automating-your-workflow-with-github-actions)
* [A curated list of awesome things related to GitHub Actions](https://github.com/sdras/awesome-actions)
* [Workflow syntax for GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)