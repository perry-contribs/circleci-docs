---
layout: classic-docs
title: "Node.js - JavaScript Tutorial"
short-title: "JavaScript"
description: "Building and Testing with JavaScript and Node.js on CircleCI 2.0"
categories: [language-guides]
order: 5
---

This document provides a walkthrough of the [`.circleci/config.yml`]({{ site.baseurl }}/2.0/configuration-reference/) file for a Node.js sample application.

* TOC
{:toc}

## Quickstart: Demo JavaScript Node.js Reference Project

We maintain a reference JavaScript Node.js project to show how to build an Express.js app on CircleCI 2.1:

- <a href="https://github.com/CircleCI-Public/circleci-demo-javascript-express" target="_blank">Demo JavaScript Node Project on GitHub</a>
- [Demo JavaScript Node Project building on CircleCI](https://circleci.com/gh/CircleCI-Public/circleci-demo-javascript-express){:rel="nofollow"}

In the project you will find a CircleCI configuration file <a href="https://github.com/CircleCI-Public/circleci-demo-javascript-express/blob/master/.circleci/config.yml" target="_blank">`.circleci/config.yml`</a>. This file shows best practice for using CircleCI 2.1 with Node projects.

## Pre-Built CircleCI Docker Images

We recommend using a CircleCI pre-built image that comes pre-installed with tools that are useful in a CI environment. You can select the Node version you need from Docker Hub: <https://hub.docker.com/r/circleci/node/>. The demo project uses an official CircleCI image.

Database images for use as a secondary 'service' container are also available.

## Build the Demo JavaScript Node Project Yourself

A good way to start using CircleCI is to build a project yourself. Here's how to build the demo project with your own account:

1. Fork the project on GitHub to your own account
2. Go to the [Add Projects](https://circleci.com/add-projects){:rel="nofollow"} page in CircleCI and click the Build Project button next to the project you just forked
3. To make changes you can edit the `.circleci/config.yml` file and make a commit. When you push a commit to GitHub, CircleCI will build and test the project.


## Sample Configuration

Following is the `.circleci/config.yml` file in the demo project with comments.

{% raw %}
```yaml
version: 2.1 # use CircleCI 2.1
jobs: # a collection of steps
  build: # runs not using Workflows must have a `build` job as entry point
    working_directory: ~/mern-starter # directory where steps will run
    docker: # run the steps with Docker
      - image: circleci/node:10.16.3 # ...with this image as the primary container; this is where all `steps` will run
      - image: mongo:4.2.0 # and this image as the secondary service container
    steps: # a collection of executable commands
      - checkout # special step to check out source code to working directory
      - run:
          name: update-npm
          command: 'sudo npm install -g npm@latest'
      - restore_cache: # special step to restore the dependency cache
          # Read about caching dependencies: https://circleci.com/docs/2.0/caching/
          key: dependency-cache-{{ checksum "package-lock.json" }}
      - run:
          name: install-npm-wee
          command: npm install
      - save_cache: # special step to save the dependency cache
          key: dependency-cache-{{ checksum "package-lock.json" }}
          paths:
            - ./node_modules
      - run: # run tests
          name: test
          command: npm test
      - run: # run coverage report
          name: code-coverage
          command: './node_modules/.bin/nyc report --reporter=text-lcov'
      - store_artifacts: # special step to save test results as as artifact
          # Upload test summary for display in Artifacts: https://circleci.com/docs/2.0/artifacts/ 
          path: test-results.xml
          prefix: tests
      - store_artifacts: # for display in Artifacts: https://circleci.com/docs/2.0/artifacts/ 
          path: coverage
          prefix: coverage
      - store_test_results: # for display in Test Summary: https://circleci.com/docs/2.0/collect-test-data/
          path: test-results.xml
      # See https://circleci.com/docs/2.0/deployment-integrations/ for deploy examples
```
{% endraw %}
---

## Config Walkthrough


Every `config.yml` starts with the [`version`]({{ site.baseurl }}/2.0/configuration-reference/#version) key.
This key is used
to issue warnings about breaking changes.

```yaml
version: 2.1
```

A run is comprised of one or more [jobs]({{ site.baseurl }}/2.0/configuration-reference/#jobs).
Because this run does not use [workflows]({{ site.baseurl }}/2.0/configuration-reference/#workflows),
it must have a `build` job.

Use the [`working_directory`]({{ site.baseurl }}/2.0/configuration-reference/#job_name) key
to specify where a job's [`steps`]({{ site.baseurl }}/2.0/configuration-reference/#steps) run. By default, the value of `working_directory` is `~/project`, where `project` is a literal string.

The steps of a job occur in a virtual environment called an [executor]({{ site.baseurl }}/2.0/executor-types/).

In this example, the [`docker`]({{ site.baseurl }}/2.0/configuration-reference/#docker) executor is used
to specify a custom Docker image. The first image listed becomes the job's [primary container]({{ site.baseurl }}/2.0/glossary/#primary-container). All commands for a job execute in this container.

```yaml
jobs:
  build:
    working_directory: ~/mern-starter
    docker:
      - image: circleci/node:4.8.2
      - image: mongo:3.4.4
```

After choosing containers for a job,
create [`steps`]({{ site.baseurl }}/2.0/configuration-reference/#steps) to run specific commands.

Use the [`checkout`]({{ site.baseurl }}/2.0/configuration-reference/#checkout) step
to check out source code. By default, source code is checked out to the path specified by `working_directory`.

To save time between runs, consider [caching dependencies or source code]({{ site.baseurl }}/2.0/caching/).

Use the [`save_cache`]({{ site.baseurl
}}/2.0/configuration-reference/#save_cache) step to cache certain files or
directories. In this example, we cache `node_modules` using a checksum of the
`package.json` as the cache-key.

Use the [`restore_cache`]({{ site.baseurl }}/2.0/configuration-reference/#restore_cache) step
to restore cached files or directories.

{% raw %}
```yaml
    steps:
      - checkout
      - run:
          name: update-npm
          command: 'sudo npm install -g npm@latest'
      - restore_cache:
          key: dependency-cache-{{ checksum "package.json" }}
      - run:
          name: install-npm-wee
          command: npm install
      - save_cache:
          key: dependency-cache-{{ checksum "package.json" }}
          paths:
            - ./node_modules
```
{% endraw %}

Now that dependencies are installed we can run the test suite
and upload the test results as an artifact (made available on the CircleCI web app).

```yaml
      - run:
          name: test
          command: npm test
      - run:
          name: code-coverage
          command: './node_modules/.bin/nyc report --reporter=text-lcov'
      - store_artifacts:
          path: test-results.xml
          prefix: tests
      - store_artifacts:
          path: coverage
          prefix: coverage
      - store_test_results:
          path: test-results.xml
```

Success! You just set up CircleCI 2.1 for a Node.js app. Check out our project’s [Job page](https://circleci.com/gh/CircleCI-Public/circleci-demo-javascript-express){:rel="nofollow"} to see how this looks when building on CircleCI.

## See Also
{:.no_toc}

- See the [Deploy]({{ site.baseurl }}/2.0/deployment-integrations/) document for example deploy target configurations.
- Refer to the [Examples]({{ site.baseurl }}/2.0/examples/) page for more configuration examples of public JavaScript projects.
- If you're new to CircleCI 2.0, we recommend reading our [Project Walkthrough]({{ site.baseurl }}/2.0/project-walkthrough/) for a detailed explanation of our configuration using Python and Flask as an example.
