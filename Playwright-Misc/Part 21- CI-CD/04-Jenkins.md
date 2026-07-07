This chapter is especially important because **Jenkins is still the most widely adopted self-hosted CI/CD platform** in large enterprises.

Although GitHub Actions and Azure DevOps are growing rapidly, many banking, telecom, healthcare, insurance, and government organizations continue to rely heavily on Jenkins due to its flexibility, plugin ecosystem, and self-hosted architecture.

This chapter focuses on **enterprise Jenkins usage** rather than simple Freestyle jobs.

----------

# Part 21 – CI/CD

# Chapter 4 – Jenkins

----------

# Introduction

Jenkins is an open-source automation server used to automate:

-   Build
    
-   Testing
    
-   Deployment
    
-   Monitoring
    
-   Release Pipelines
    

Playwright integrates seamlessly with Jenkins, allowing teams to execute UI, API, and end-to-end automation as part of every build.

----------

# Jenkins Workflow

A typical Playwright pipeline looks like this:

```text
Developer Commit

↓

Git Repository

↓

Jenkins Trigger

↓

Agent

↓

Checkout Code

↓

Install Dependencies

↓

Install Browsers

↓

Run Playwright

↓

Generate Reports

↓

Archive Reports

↓

Notification

```

----------

# Jenkins Architecture

Jenkins follows a controller-agent architecture.

```text
             Jenkins Controller
                    │
     ┌──────────────┼──────────────┐
     │              │              │
 Agent 1        Agent 2        Agent 3

 Chromium       Firefox        WebKit

```

----------

## Jenkins Controller

Responsibilities

-   Pipeline orchestration
    
-   Scheduling jobs
    
-   Plugin management
    
-   User management
    
-   Build history
    

The controller should **not** execute heavy Playwright tests in large environments.

----------

## Jenkins Agents

Agents execute actual work.

Examples

```text
Linux Agent

↓

Playwright Tests

-----------------

Windows Agent

↓

Desktop Testing

-----------------

Mac Agent

↓

Safari Tests

```

----------

# Jenkins Jobs

Jenkins supports multiple job types.

Job Type

Recommended

Freestyle

❌ Legacy

Pipeline

✅ Recommended

Multibranch Pipeline

✅ Enterprise

Organization Folder

Large Enterprises

----------

# Freestyle Job

Older approach.

```text
Source Code

↓

Build Step

↓

Execute Shell

↓

Reports

```

Easy to configure but difficult to maintain.

----------

# Pipeline Job

Everything is defined as code.

```text
Jenkinsfile

↓

Git Repository

↓

Version Controlled

```

Advantages

-   Version controlled
    
-   Code review
    
-   Reusable
    
-   Easier maintenance
    

----------

# Jenkinsfile

Stored in the repository.

```text
project/

├── Jenkinsfile

├── package.json

├── playwright.config.ts

└── tests/

```

----------

# Basic Jenkinsfile

```groovy
pipeline {

    agent any

    stages {

        stage('Install') {

            steps {

                sh 'npm ci'

            }

        }

        stage('Playwright') {

            steps {

                sh 'npx playwright test'

            }

        }

    }

}

```

----------

# Pipeline Structure

```text
Pipeline

↓

Stages

↓

Steps

↓

Artifacts

```

----------

# Installing Node.js

Many organizations install Node using Jenkins tools.

```groovy
tools {

    nodejs 'NodeJS-20'

}

```

Or use a preconfigured agent with Node already installed.

----------

# Installing Dependencies

```groovy
stage('Install Packages'){

    steps{

        sh 'npm ci'

    }

}

```

Always prefer

```text
npm ci

```

for CI.

----------

# Installing Browsers

```groovy
stage('Install Browsers'){

    steps{

        sh 'npx playwright install --with-deps'

    }

}

```

Linux agents require browser dependencies.

----------

# Running Playwright

```groovy
stage('Run Tests'){

    steps{

        sh 'npx playwright test'

    }

}

```

----------

# Running Smoke Tests

```groovy
sh 'npx playwright test --grep @smoke'

```

----------

# Running Regression

```groovy
sh 'npx playwright test --grep @regression'

```

----------

# Running Specific Browser

```groovy
sh 'npx playwright test --project=chromium'

```

----------

# Pipeline Stages

Typical enterprise pipeline

```text
Checkout

↓

Install

↓

Build

↓

Smoke

↓

Regression

↓

Reports

↓

Deploy

```

----------

# Parallel Execution

Jenkins supports parallel stages.

Example

```groovy
stage('Parallel Tests'){

    parallel{

        stage('Chromium'){

            steps{

                sh 'npx playwright test --project=chromium'

            }

        }

        stage('Firefox'){

            steps{

                sh 'npx playwright test --project=firefox'

            }

        }

        stage('WebKit'){

            steps{

                sh 'npx playwright test --project=webkit'

            }

        }

    }

}

```

Execution

```text
Chromium

↓

Firefox

↓

WebKit

```

Runs simultaneously.

----------

# Environment Variables

```groovy
environment{

    BASE_URL='https://qa.company.com'

}

```

Inside Playwright

```typescript
process.env.BASE_URL

```

----------

# Jenkins Credentials

Never hardcode credentials.

Jenkins

```text
Manage Jenkins

↓

Credentials

↓

Secret Text

↓

Username Password

```

Pipeline

```groovy
environment{

PASSWORD=

credentials('qa-password')

}

```

----------

# Publishing HTML Report

Archive report.

```groovy
post{

always{

archiveArtifacts(

artifacts:'playwright-report/**'

)

}

}

```

Developers can download the report after execution.

----------

# Publish JUnit

```groovy
post{

always{

junit 'reports/results.xml'

}

}

```

Jenkins displays

-   Passed
    
-   Failed
    
-   Trends
    

----------

# Archive Screenshots

```groovy
archiveArtifacts(

artifacts:'test-results/**'

)

```

Includes

-   Screenshots
    
-   Videos
    
-   Traces
    

----------

# Build Parameters

Manual execution.

```text
Browser

Environment

Suite

```

Pipeline

```groovy
parameters{

choice(

name:'Browser',

choices:['chromium','firefox']

)

}

```

Execute

```groovy
sh "npx playwright test --project=${params.Browser}"

```

----------

# Scheduled Builds

Nightly regression

```groovy
triggers{

cron('H 2 * * *')

}

```

----------

# Multibranch Pipeline

Automatically builds

```text
main

develop

feature/login

release

```

Each branch gets its own pipeline.

----------

# Docker with Jenkins

Instead of installing browsers on agents,

use Playwright Docker images.

```groovy
agent{

docker{

image

'mcr.microsoft.com/playwright:v1.55.0-noble'

}

}

```

Advantages

-   Consistent environment
    
-   Faster setup
    
-   Easier maintenance
    

----------

# Shared Libraries

Large enterprises avoid duplicating pipeline code.

```text
Pipeline

↓

Shared Library

↓

Common Functions

```

Example

```groovy
playwrightPipeline()

```

Reusable across hundreds of repositories.

----------

# Enterprise Jenkins Architecture

```text
Git

↓

Jenkins Controller

↓

Linux Agents

↓

Playwright

↓

Reports

↓

Artifacts

↓

Slack

↓

Dashboard

```

----------

# Recommended Folder Structure

```text
jenkins/

├── Jenkinsfile

├── smoke.groovy

├── regression.groovy

├── release.groovy

└── shared-library/

```

----------

# Complete Enterprise Jenkinsfile

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                checkout scm

            }

        }

        stage('Install') {

            steps {

                sh 'npm ci'

            }

        }

        stage('Install Browsers') {

            steps {

                sh 'npx playwright install --with-deps'

            }

        }

        stage('Run Tests') {

            steps {

                sh 'npx playwright test'

            }

        }

    }

    post {

        always {

            archiveArtifacts(
                artifacts: 'playwright-report/**'
            )

            junit 'reports/results.xml'

        }

    }

}

```

----------

# Jenkins Plugins for Playwright

Common plugins:

Plugin

Purpose

HTML Publisher

Publish HTML reports

JUnit

Display test results

Pipeline

Pipeline support

NodeJS

Manage Node versions

Credentials Binding

Secure credentials

Slack Notification

Slack integration

Blue Ocean

Modern pipeline UI

----------

# Enterprise Best Practices

-   Use **Pipeline as Code** with `Jenkinsfile`.
    
-   Prefer **Docker agents** for consistent Playwright environments.
    
-   Store credentials in **Jenkins Credentials**, not in source code.
    
-   Execute browser projects in parallel to reduce runtime.
    
-   Archive HTML reports, traces, videos, and screenshots.
    
-   Use Shared Libraries to standardize pipelines across repositories.
    
-   Separate Smoke, Regression, and Release pipelines.
    

----------

# Common Mistakes

### ❌ Using Freestyle Jobs

Freestyle jobs don't scale well.

Prefer

```text
Pipeline

+

Jenkinsfile

```

----------

### ❌ Installing Everything Every Build

Instead of

```text
Install Node

Install Browsers

Install Dependencies

```

consider Docker images or preconfigured agents.

----------

### ❌ Hardcoding Credentials

Never do

```groovy
PASSWORD='admin123'

```

Use

```text
Jenkins Credentials

```

----------

### ❌ Running Everything Sequentially

Use

```text
Parallel Stages

```

for

-   Browsers
    
-   Test Suites
    
-   Environments
    

----------

### ❌ Not Publishing Reports

Always archive:

-   HTML Report
    
-   Screenshots
    
-   Videos
    
-   Traces
    
-   JUnit XML
    

----------

# Interview Questions

### Q1. Why is Pipeline as Code preferred over Freestyle Jobs?

Because it is version-controlled, reusable, reviewable, and easier to maintain.

----------

### Q2. Why should Playwright run on Jenkins agents instead of the controller?

The controller should focus on orchestration. Running browser automation on agents improves scalability, performance, and stability.

----------

### Q3. Why use Docker with Jenkins?

Docker provides a consistent execution environment, eliminates dependency issues, and simplifies Playwright browser installation.

----------

### Q4. What is the purpose of Jenkins Shared Libraries?

They allow organizations to centralize reusable pipeline logic and avoid duplicating Jenkinsfile code across multiple repositories.

----------

### Q5. Which Jenkins plugins are commonly used with Playwright?

-   HTML Publisher
    
-   JUnit
    
-   Pipeline
    
-   NodeJS
    
-   Credentials Binding
    
-   Slack Notification
    
-   Blue Ocean
    

----------

# Summary

Jenkins remains one of the most powerful and flexible CI/CD platforms for enterprise Playwright automation. By combining Pipeline as Code, distributed agents, Docker containers, parallel execution, secure credential management, artifact publishing, and shared libraries, teams can build scalable and maintainable automation pipelines capable of supporting large-scale enterprise testing.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzI1MzM0NzE4XX0=
-->