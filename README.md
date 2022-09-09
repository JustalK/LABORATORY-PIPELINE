# LABORATORY-PIPELINE

This laboratory is where I tested how to properly use the pipeline from **GitHub**, **Bitbucket** and **GitLab**. I hav read all the documentation of each of them and tested how to use the different features present. I try created a pipeline that I can use professionally for each of the version control. I have also created small example for each feature and explain deeply how to setup the more important part with screenshots.


Warning: The case of the name of the branches are important for any pipelines

## Plan of the presentation

I explain with all the details how I build the project and my way of working.

1. [GitHub](#github)
2. [Bitbucket](#bitbucket)
3. [GitLab](#gitlab)
## GitHub

#### Experiences

![./documentation/4.png](./documentation/4.png)

I have created many actions on this repositories for testing how it exactly works. Those are the different things that has been tested:

- Running a bash script
- Testing the concurrency by grouping actions
- Testing the dispatch for manually running action with inputs
- Playing with the context
- Testing the output and sharing information between jobs
- Creating a scheduled actions
- Playing with the secrets by environment and global secrets
- Creating a pipeline for CI

#### Errors & fixes

Some information that might be useful if you get a permission error with the scripts. You can set the permission with the commands below:

```bash
$ git update-index --chmod=+x ./.github/scripts/hello.sh
```

#### Triggerring action manually

For starting an action by CURL:

```bash
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: Bearer <BEARER TOKEN>" "https://api.github.com/repos/justalk/LABORATORY-PIPELINE/actions/workflows/dispatch.yml/dispatches" -d '{"ref":"feature/check-github-pipeline","inputs":{"testVariable":"KEVIN", "environment":"stagging"}}'
```

#### Defining secret

Everything happen in the `Setting` tabs, in the `Secret` menu.

![./documentation/3.png](./documentation/3.png)

However for defining secret be environment, we can only define the secret from inside the environment.

![./documentation/1.png](./documentation/1.png)

![./documentation/2.png](./documentation/2.png)

#### Continuous Integration

I have created a **React** project using **NX**. I created an action testing the builds and tests on two different jobs using a matrix around the os. I have put a rule on concurrency for blocking multiple action to run if I keep pushing. I am using npx because you cannot access directly the command nx. I added the workflow_dispatch for manually running the pipeline when I want.

```yml
name: CI

on:
  workflow_dispatch:
  push:
  pull_request:

permissions: read-all

concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-22.04, ubuntu-20.04]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "16.x"
      - name: Install CI
        working-directory: ./project
        run: npm ci
      - name: Build the apps
        working-directory: ./project
        run: npx nx build
      - name: Run the test
        working-directory: ./project
        run: npx nx test
```

## Bitbucket

I went through the entire documentation for testing the possibilities compared to GitHub. Everything can be found in the `bitbucket-pipeline.yml`. I tested:

- how to use artifacts
- How to use the caches
- How to create step in parallel
- Run a script after completion
- Manual trigger step and creating manual pipeline
- Naming of PR, default and branches of pipeline
- How to use secret

#### Skipping the CI when push

Just add the message `[skip ci]` in the commit to skip the CI.

![./documentation/5.png](./documentation/5.png)

#### Pipeline custom

By using `custom`, you can create manual pipelines with variables. Those pipelines can be triggered in the `pipeline` pages:

![./documentation/7-8.png](./documentation/7-8.png)

#### Manual trigger

Step can be put as `trigger: manual` which let only the push possible through the web interface. When a step require a manual intervention, the symbol change a bit:

![./documentation/13.png](./documentation/13.png)

If you go in the pipeline, a button `run` will be available and ready to be pressed for starting the next step:

![./documentation/14.png](./documentation/14.png)

#### Secrets

For defining secrets, go inside the `Repository Settings` which is at the bottom of the repository you are working on. At the bottom, in the section `pipeline`, there are two menu: `Repository Variables` and `Deployement`.

The repository variables will be accessible through all the pipeline while the one from deployement will only be available if the `deployement` field in the pipeline match the one define in the web.

![./documentation/9.png](./documentation/9.png)

![./documentation/10.png](./documentation/10.png)

#### Caching

- Any cache which is older than 1 week will be cleared automatically and repopulated during the next build.
- Warning: If the cache define is empty, it wont be cached. So careful when the workdirectory is different from where we cache. For example, the `node_modules` in this repository will be in the `project` folder. If I define the cache such as `node`, it wont be cache because the node_modules will be expected at the root. This iw hy I change the definition of my cache at the bottom.

You can see the download of the cache:

![./documentation/11.png](./documentation/11.png)

And the cache can be download or deleted from the main pipelines page:

![./documentation/12.png](./documentation/12.png)

#### Artifacts

Artifacts is kinda like cache at one difference, it will be only available in the current pipeline. A cache is however available through the different pipeline.

When you define an artifact, it can be seen in the logs.

![./documentation/15.png](./documentation/15.png)

The artifacts can be download in the tab `artifact` present in the pipeline.

![./documentation/16.png](./documentation/16.png)

#### Continuous Integration

![./documentation/17.png](./documentation/17.png)

A simple CI would be as followed. In the first step, I will install the dependencies and build the app. The compiled project will be put in an artifact while the dependencies put in the cache for the next pipelines. The second step will validate the project. In my case, I only have unit test and the linter. I did that in parallel since one job does not block the other one. Last step will be for the deployement (see next point).

```yml
  branches:
    master:
      - step:
          name: "Install the dependencies"
          caches:
            - frontendnode
          script:
            - cd project
            - npm ci
            - npx nx build
          artifacts:
            - project/dist/**
          after-script:
            - echo "Installation and build of the project done"
      - parallel:
          - step:
              name: "Test the linting"
              caches:
                - frontendnode
              script:
                - cd project
                - npx nx lint
              after-script:
                - echo "Linting OK"
          - step:
              name: "Test the application"
              caches:
                - frontendnode
              script:
                - cd project
                - npx nx test
              after-script:
                - echo "Test OK"
      - step:
          name: "Deployement"
          trigger: manual
          script:
            - echo "Step to do for deploying SCP"
          after-script:
            - echo "Deployement OK"
```

#### Continuous Deployement

If we want to deploy the website at the end of the pipeline, we need to use the following step. You also certainly need to define multiple secret depending of the environment you want to deploy on.

```yml
- step:
      name: Deploy to XXX
      deployment: XXX
      script:
      - echo "Deploying to XXX environment"
      - pipe: atlassian/ssh-run:0.2.2
        variables:
            SSH_USER: $SERVER_USER
            SERVER: $SERVER_IP
            COMMAND: 'bash ./deploy.sh'
      - pipe: atlassian/slack-notify:2.0.0
        variables:
          WEBHOOK_URL: 'https://hooks.slack.com/services/yyyy/xxxx/zzzz'
          MESSAGE: 'A random message'
```

```bash
git pull
echo '## deploy.sh: Installing the packages in any case there is new one'
npm install
echo '## deploy.sh: Seeding the database'
npm run seed
echo '## deploy.sh: Reloading the nodes'
pm2 reload all
```

## GitLab

Since I like to keep money in my pocket, the first step is to register a runner. I will need all the information for connecting the runner, go in your repository, click on the `Settings` at the bottom and expand the menu `Runner`:

![./documentation/20.png](./documentation/20.png)

You will need the information for completing the following command.

```bash
$ sudo gitlab-runner register
```

![./documentation/18.png](./documentation/18.png)

If everything went well, you should see you runner below. Dont forget to desactivate the `Shared Runner`.

![./documentation/19.png](./documentation/19.png)

#### Experiences

I have created multiple yaml where I have been testing the following experiences.

- link between pipeline
- Collapse of message
- inheritance of pipeline
- manual pipeline
- rules
- external file
- parallele
- matrix

in the log of the pipeline, you can use color using the bash color system.
Bash Color: https://misc.flogisoft.com/bash/tip_colors_and_formatting

#### Secrets

The secret of the pipeline can be setup in the variable section located in `Settings` -> CI/CD -> Variables

![./documentation/21.png](./documentation/21.png)

When you add a variable, a popup open where you can select the environment if needed and if the variable can be read in the pipeline or not.

![./documentation/22.png](./documentation/22.png)

The result will be as follow once two variables has been added.

![./documentation/23.png](./documentation/23.png)

#### Caches

As I can see under, the cache has been created at the end of the job.

![./documentation/24.png](./documentation/24.png)

The cache is located locally, under Docker volumes: /var/lib/docker/volumes/<volume-id>/_data/<user>/<project>/<cache-key>/cache.zip.
I can see the volume name in the pipeline and when I check in the location on my computer, I can found my cache.

![./documentation/25.png](./documentation/25.png)

You can clear cache using the button in the pipeline page.

![./documentation/26.png](./documentation/26.png)

#### Settings

Since pipeline are paid by the minutes of usage, it's important to check the options: Auto-cancel redundant pipeline
It stop the pipelines to be queue when we push before another pipeline has been finished.

![./documentation/27.png](./documentation/27.png)

#### Continuous Integration

On gitlab, we can make reference between projects. So I will create a simple `.gitlab-ci.yml` that will include a template from another project exclusively used by the DevOps for the pipelines.

![./documentation/graph.png](./documentation/graph.png)

```yml
stages:
  - build
  - test
  - deploy

include:
  - project: "justal.kevin/devops"
    file: "/templates/rules.yml"
    ref: "master"

build:
  variables:
    appName: laboratory-gitlab
```

And in the DevOps project, I will create the pipeline for continuous integration:

```yml
stages:
  - build
  - check
  - deploy

image: node:16

build:
  stage: build
  cache:
    key: build-cache
    paths:
      - ./project/node_modules
  artifacts:
    paths:
      - ./project/dist
    expire_in: 1 week
  script:
    - cd project
    - npm install
    - npx nx build
  after_script:
    - echo "Building DONE"

check 1/2:
  stage: check
  cache:
    key: build-cache
    paths:
      - ./project/node_modules
  artifacts:
    paths:
      - ./project/dist
  script:
    - cd project
    - npx nx lint
  after_script:
    - echo "Check Linter DONE"


check 2/2:
  stage: check
  cache:
    key: build-cache
    paths:
      - ./project/node_modules
  artifacts:
    paths:
      - ./project/dist
  script:
    - cd project
    - npx nx test
  after_script:
    - echo "Check Test DONE"

deploy:
  stage: deploy
  rules:
    - when: manual
      allow_failure: true
  script:
    - echo "Step to do for deploying"
  after_script:
    - echo "Deployement DONE"
```