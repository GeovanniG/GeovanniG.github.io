---
title: "Creating Code Coverage Reports Using GitHub Actions For .Net Core Applications"
date: 2023-08-19T00:00:00-07:00
draft: false
---

As we progress further into our development, we need a way of streamlining our build process. One way to achieve this is using GitHub actions.

GitHub actions are a tool we can use to automate the building, testing, and deployment of our applications. In this article, we will cover the building and testing aspects. The deployment portion is unique and depends on where you host your application.

## Parts to a GitHub Action

A GitHub action is composed of a few different parts. To begin, we have the triggers or _events_. An event is what triggers the GitHub action to execute. They can be a push, a pull request, a pull, or any [other event](https://docs.GitHub.com/en/actions/using-workflows/events-that-trigger-workflows) supported by GitHub actions.

After specifying the event, we must then list the jobs. A GitHub action can have any number of jobs, including only one, and by default, the jobs run in parallel. Jobs will also need a runner, e.g. `ubuntu-latest`, in which they execute. Lastly, jobs must have a list of actions. The actions are the most crucial aspect of the GitHub action. It is here that we specify which tasks we would like to execute in our GitHub action.

> For a more thorough understanding of GitHub actions, please visit [Understanding GitHub Actions](https://docs.GitHub.com/en/actions/learn-GitHub-actions/understanding-GitHub-actions).

## Creating Our GitHub Action

Now that we have a basic understanding of GitHub actions, we can start developing our first one targeting .NET applications.

We will start off with a basic example that builds and tests our .NET application. The GitHub action is given below:

```yml
name: execute-test-on-merge

run-name: ${{ GitHub.repository }} main or release branch merges

on:
  push:
    branches:
      - main
      - 'releases/**'
  pull_request:
    branches:
      - main
      - 'releases/**'

env:
  SOLUTION_PATH: <path-to-solution>
  CONFIGURATION: DEBUG
  DOTNET_VERSION: 7.x

jobs:
  code_coverage:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Restore packages
        run: dotnet restore ${{ env.SOLUTION_PATH }}
      - name: Build Solution
        run: dotnet build ${{ env.SOLUTION_PATH }} --configuration ${{ env.CONFIGURATION }} --no-restore
      - name: Execute Test
        run: |
            dotnet test ${{ env.SOLUTION_PATH }} --configuration ${{ env.CONFIGURATION }} --no-restore --no-build
```

This pipeline is a great first step in automating our workflow. It is composed of two events: `push` and `pull_request` that target the `main` and `releases/**` branches. It has one job named `code_coverage` and this job has 5 steps. The steps include: checking out the branch, selecting the .NET environment, and building and executing the tests.

In many cases, this pipeline is enough. If our tests fail, the GitHub action will fail and notify, via email, all corresponding subscribers.

However, this pipeline lacks code coverage reports. Since test projects are configured with [`coverlet`](https://GitHub.com/coverlet-coverage/coverlet) by default, we will utilize this tool in conjunction with [`reportgenerator`](https://GitHub.com/danielpalme/ReportGenerator) to add code coverage reports to our GitHub action.

## Using Coverlet To Create Code Coverage Reports

As our codebase expands and evolves, so should our tests. For this reason, it is great to ensure a minimum test coverage is reached.

To add test coverage reports, we can add a few more actions to our pipeline that utilize `coverlet` and `reportgenerator`:

```yml
name: execute-test-on-merge

run-name: ${{ GitHub.repository }} main or release branch merges

on:
  push:
    branches:
      - main
      - 'releases/**'
  pull_request:
    branches:
      - main
      - 'releases/**'

env:
  SOLUTION_PATH: <path-to-solution>
  CONFIGURATION: DEBUG

jobs:
  code_coverage:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: 7.x

      - name: Restore packages
        run: dotnet restore ${{ env.SOLUTION_PATH }}
      - name: Build Solution
        run: dotnet build ${{ env.SOLUTION_PATH }} --configuration ${{ env.CONFIGURATION }} --no-restore
      - name: Execute Test
        run: |
            dotnet test ${{ env.SOLUTION_PATH }} --configuration ${{ env.CONFIGURATION }} --no-restore --no-build \
                        --collect:"XPlat Code Coverage" --logger:"trx;LogFileName=testresults.trx"

      - name: ReportGenerator
        uses: danielpalme/ReportGenerator-GitHub-Action@5.1.21
        with:
          reports: './**/coverage.cobertura.xml'
          targetdir: './coveragereport'
          reporttypes: 'HTML'
        if: ${{ always() }}

      - name: Upload Dotnet Test Results
        uses: actions/upload-artifact@v3
        with:
          name: dotnet-test-results
          path: ./coveragereport
        if: ${{ always() }}
```

This new pipeline slightly improves on the one we had previously. By including `--collect:"XPlat Code Coverage"` in `dotnet test`, we use `coverlet` to generate test results in the form of `coverage.cobertura.xml` files. The test results are then fed into `reportgenerator`, where elegant coverage reports are generated under the `./coveragereport` folder. Lastly, we publish the reports back into the GitHub action as artifacts, where they can be later consumed and analyzed.

Furthermore, we added the following necessary condition: `if: ${{ always() }}`. With this condition added, even if our tests fail, the steps labeled with this condition would still be executed. Otherwise, if this condition was not included and our tests failed, the pipeline would end at the `dotnet test` step.

## Displaying Reports on GitHub Pages

Downloading the artifact can be a tedious task. We can supplement this step by displaying the reports directly on GitHub Pages by using the action `peaceiris/actions-gh-pages@v3` and assigning the `contents: write` permission to the GitHub action:

```yml
name: execute-test-on-merge

run-name: ${{ GitHub.repository }} main or release branch merges

on:
  push:
    branches:
      - main
      - 'releases/**'
  pull_request:
    branches:
      - main
      - 'releases/**'

env:
  SOLUTION_PATH: <path-to-solution>
  CONFIGURATION: DEBUG

jobs:
  code_coverage:
    runs-on: ubuntu-latest

    permissions:
      contents: write

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: 7.x

      - name: Restore packages
        run: dotnet restore ${{ env.SOLUTION_PATH }}
      - name: Build Solution
        run: dotnet build ${{ env.SOLUTION_PATH }} --configuration ${{ env.CONFIGURATION }} --no-restore
      - name: Execute Test
        run: |
            dotnet test ${{ env.SOLUTION_PATH }} --configuration ${{ env.CONFIGURATION }} --no-restore --no-build \
                        --collect:"XPlat Code Coverage" --logger:"trx;LogFileName=testresults.trx"

      - name: ReportGenerator
        uses: danielpalme/ReportGenerator-GitHub-Action@5.1.21
        with:
          reports: './**/coverage.cobertura.xml'
          targetdir: './coveragereport'
          reporttypes: 'HTML'
        if: ${{ always() }}

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.GitHub_TOKEN }}
          publish_dir: ./coveragereport
        if: ${{ always() }}

      - name: Upload Dotnet Test Results
        uses: actions/upload-artifact@v3
        with:
          name: dotnet-test-results
          path: ./coveragereport
        if: ${{ always() }}
```

This does have limitations, as it will only display the most recent coverage reports. However, in most cases, this is enough.

Thanks for reading!