---
layout: post
title: Github Action to create an Azure DevOps Bug Work Item for failing Github Workflow actions
categories: [CI/CD]
tags: [CI/CD]
comments: true
---

I developed a new Github Action to create an Azure DevOps Bug Work Item for failing Github Workflow actions.

For Azure DevOps there are already some [Extensions](https://marketplace.visualstudio.com/items?itemName=AmanBedi18.CreateBugTask) that create a Bug when a release is failing, but I wanted to have something simular in Github Workflows.

# How does it works?

## Github Action Azure DevOps Bug Work Item

With the Github Action Azure DevOps Bug Work Item you can create an Azure DevOps Bug Work Item for failed Github Workflow Actions.

Screenshot Github Workflow with failed Action

![Failed Workflow Screenshot](/assets/FailedWorkflow.png)

Screenshot Bug created in Azure DevOps Backlog

![Azure DevOps Bug Work Item screenshot](/assets/AzDBugWorkItem.png)

## Azure DevOps Bug Work Items

As your team identifies code defects or bugs, they can add them to the backlog and track them similar to requirements. Or, they can schedule them to be fixed within a sprint along with other tasks.

More information can be found [here](https://docs.microsoft.com/en-us/azure/devops/organizations/settings/show-bugs-on-backlog?view=azure-devops)

### Sample workflow that uses Azure DevOps Bug Work Item

```yaml
name: Error

on: [push]

jobs:
  error:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v1
      - name: Throw error in PowerShell script
        run: |
          throw 'something failed';
      - name: The job has failed
        uses: stefanstranger/azuredevops-bug-action@v1
        if: failure()
        with:
          OrganizationName: "stefanstranger"
          PAT: "PAT"
          ProjectName: "Contoso"
          AreaPath: "Contoso\\Automation"
          IterationPath: "Contoso"
          GithubToken: "GithubToken"
          WorkflowFileName: "error.yml"
        env:
          PAT: ${{ secrets.PAT}}
          GithubToken: ${{ secrets.githubtoken}}

```

| Input            | Description                                                               |
| ---------------- | ------------------------------------------------------------------------- |
| OrganizationName | Azure DevOps Organization name                                            |
| PAT              | Github Secret Name for PAT stored as Github environment variable          |
| ProjectName      | Azure DevOps Project name                                                 |
| AreaPath         | Azure DevOps Area Path for the Bug Work Items                             |
| IterationPath    | Azure DevOps Iteration Path for the Bug Work Item                         |
| GithubToken      | Github Secret Name for Github Token stored as Github environment variable |
| WorkflowFileName | Github Workflow file name                                                 |

### Configure Personal Access Token (PAT)

For creating the Azure DevOps Bug Work Item we need to create an Azure DevOps Personal Access Token.

More information about creating an Azure DevOps Personal Access token can be found [here](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page)

The minimal scope authorization for Work Items is Read, write & manage.

<img src="/assets/PAT.png" alt="PAT configuration" width="700" height="533"/>

Add the PAT output as [a secret](https://aka.ms/create-secrets-for-GitHub-workflows) (let's say with the name `PAT`) in the GitHub repository.

![PAT Github Secret screenshot](/assets/PATGHSecret.png)

### Configure Github Token

This Github Action uses the [Github REST API](https://developer.github.com/v3/) to retrieve detailed information about the failed Github Workflow Run.

Create a Github Personal Access Token via the [Github Developer Settings](https://github.com/settings/tokens).

Your Github Token needs the following permissions:

* repo
    * repo:status
    * repo_deployment
    * public_repo
    * repo:invite
* workflow

Add the Github Token [a secret](https://aka.ms/create-secrets-for-GitHub-workflows) (let's say with the name `githubtoken`) in the GitHub repository.

![Github Token Secret screenshot](/assets/GithubTokenSecret.png)

Hope you enjoy this new Github Action!  

**References**

- [Github Repository](https://github.com/stefanstranger/azuredevops-bug-action)

- [Action in Github MarktPlace](https://github.com/marketplace/actions/github-action-to-create-an-azure-devops-bug-workitem-when-a-workflow-fails)