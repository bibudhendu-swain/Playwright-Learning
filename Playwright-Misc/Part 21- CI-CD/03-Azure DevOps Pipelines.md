This chapter is especially valuable because **Azure DevOps is still the dominant CI/CD platform in many enterprise organizations**, particularly in banking, insurance, healthcare, telecom, and government sectors.

----------

# Part 21 – CI/CD

# Chapter 3 – Azure DevOps Pipelines

----------

# Introduction

Azure DevOps Pipelines is Microsoft's cloud-based Continuous Integration and Continuous Delivery (CI/CD) service.

It enables teams to automatically:

-   Build applications
    
-   Execute Playwright tests
    
-   Generate reports
    
-   Publish artifacts
    
-   Deploy applications
    
-   Manage releases
    

For enterprise projects, Azure DevOps often serves as the central automation platform.

----------

# Azure DevOps Pipeline Workflow

A typical Playwright pipeline follows this flow:

```text
Developer Commit

↓

Azure Repos / GitHub

↓

Azure Pipeline Triggered

↓

Agent Starts

↓

Install Dependencies

↓

Install Playwright Browsers

↓

Run Tests

↓

Generate Reports

↓

Publish Artifacts

↓

Notify Team

```

----------

# Azure DevOps Components

Component

Purpose

Azure Repos

Source Control

Azure Pipelines

CI/CD

Azure Test Plans

Test Management

Azure Artifacts

Package Management

Azure Boards

Agile Planning

Playwright mainly integrates with **Azure Pipelines**.

----------

# Pipeline Types

Azure DevOps supports:

```text
Classic Pipeline

(YAML Pipeline)

```

Modern projects should use:

✅ YAML Pipelines

Advantages:

-   Version Controlled
    
-   Reviewable
    
-   Reusable
    
-   Easy to Maintain
    

----------

# Pipeline File

Stored inside repository.

```text
azure-pipelines.yml

```

Project

```text
project/

├── tests/

├── playwright.config.ts

├── package.json

└── azure-pipelines.yml

```

----------

# Basic Pipeline

```yaml
trigger:

- main

pool:

  vmImage: ubuntu-latest

steps:

- checkout: self

- task: NodeTool@0

  inputs:

    versionSpec: '20.x'

- script: npm ci

- script: npx playwright install --with-deps

- script: npx playwright test

```

----------

# Pipeline Structure

```text
Trigger

↓

Agent

↓

Checkout

↓

Install

↓

Run Tests

↓

Publish

```

----------

# Pipeline Triggers

Automatic execution

```yaml
trigger:

- main

```

Multiple branches

```yaml
trigger:

- main

- develop

- release/*

```

Pull Request

```yaml
pr:

- main

```

----------

# Hosted vs Self-Hosted Agents

Azure supports two types of agents.

## Microsoft Hosted Agent

```text
Azure

↓

Fresh VM

↓

Execute Pipeline

↓

Destroyed

```

Advantages

-   No maintenance
    
-   Latest tools
    
-   Easy setup
    

Disadvantages

-   Limited customization
    
-   Startup time
    

----------

## Self Hosted Agent

```text
Company Server

↓

Agent Installed

↓

Pipeline

↓

Reuse

```

Advantages

-   Faster
    
-   Custom Software
    
-   Internal Network Access
    

Disadvantages

-   Maintenance required
    

----------

# Agent Selection

```yaml
pool:

  vmImage: ubuntu-latest

```

Windows

```yaml
pool:

  vmImage: windows-latest

```

macOS

```yaml
pool:

  vmImage: macos-latest

```

----------

# Checkout Repository

```yaml
- checkout: self

```

Downloads source code onto the agent.

----------

# Installing Node.js

```yaml
- task: NodeTool@0

  inputs:

    versionSpec: '20.x'

```

Always use an LTS version unless your project requires another version.

----------

# Install Dependencies

```yaml
- script: npm ci

  displayName: Install Packages

```

Why `npm ci`?

-   Faster
    
-   Clean installation
    
-   Uses package-lock.json
    
-   Best practice for CI
    

----------

# Install Browsers

```yaml
- script:

    npx playwright install --with-deps

  displayName:

    Install Browsers

```

Linux agents require browser dependencies.

----------

# Execute Tests

```yaml
- script:

    npx playwright test

  displayName:

    Run Playwright Tests

```

----------

# Pipeline Variables

Instead of hardcoding values,

use variables.

```yaml
variables:

  BASE_URL:

    https://qa.company.com

```

Access

```typescript
process.env.BASE_URL

```

----------

# Variable Groups

Azure DevOps supports shared variables.

```text
Pipeline

↓

Variable Group

↓

Environment

↓

Playwright

```

Example

```yaml
variables:

- group: QA-Variables

```

Useful for:

-   URLs
    
-   Credentials
    
-   Environment Names
    

----------

# Secure Variables

Mark sensitive variables as

```text
Secret

```

Examples

```text
PASSWORD

API_KEY

CLIENT_SECRET

```

Secrets are masked in logs.

----------

# Azure Key Vault Integration

Large organizations rarely store secrets directly in pipelines.

Instead

```text
Azure Key Vault

↓

Pipeline

↓

Playwright

```

Benefits

-   Centralized secrets
    
-   Rotation
    
-   Security
    
-   Compliance
    

----------

# Running Different Test Suites

Smoke

```yaml
- script:

    npx playwright test --grep @smoke

```

Regression

```yaml
- script:

    npx playwright test --grep @regression

```

Sanity

```yaml
- script:

    npx playwright test --grep @sanity

```

----------

# Multiple Browsers

```yaml
- script:

    npx playwright test --project=chromium

```

Or

```yaml
- script:

    npx playwright test --project=firefox

```

----------

# Matrix Strategy

Run browsers in parallel.

```yaml
strategy:

  matrix:

    chromium:

      browser: chromium

    firefox:

      browser: firefox

    webkit:

      browser: webkit

```

Run

```yaml
- script:

    npx playwright test --project=$(browser)

```

----------

# Publish HTML Report

```yaml
- task: PublishPipelineArtifact@1

  inputs:

    targetPath:

      playwright-report

    artifact:

      HTMLReport

```

----------

# Publish Test Results

JUnit XML

```yaml
- task:

    PublishTestResults@2

  inputs:

    testResultsFormat:

      JUnit

    testResultsFiles:

      reports/results.xml

```

Azure DevOps displays:

-   Passed
    
-   Failed
    
-   Duration
    
-   Trends
    

----------

# Publish Videos

```yaml
- task:

    PublishPipelineArtifact@1

  inputs:

    targetPath:

      test-results

```

Includes

-   Videos
    
-   Screenshots
    
-   Traces
    

----------

# Multi-Stage Pipeline

Enterprise pipelines often use multiple stages.

```text
Build

↓

Smoke Tests

↓

Regression

↓

Deploy QA

↓

Deploy Staging

↓

Production

```

----------

# Example

```yaml
stages:

- stage: Build

- stage: Test

- stage: Deploy

```

----------

# Deployment Gates

Production deployment often requires approval.

```text
Regression Passed

↓

Manager Approval

↓

Deploy Production

```

Automation improves confidence but human approval may still be required.

----------

# Scheduled Regression

Nightly pipeline

```yaml
schedules:

- cron: "0 2 * * *"

```

Meaning

```text
Daily

↓

02:00 UTC

```

----------

# Parallel Jobs

Azure executes

```text
API Tests

↓

UI Tests

↓

Performance Tests

```

simultaneously.

This reduces execution time.

----------

# Pipeline Artifacts

Typical artifacts

```text
playwright-report/

test-results/

screenshots/

videos/

traces/

```

Developers download them after execution.

----------

# Enterprise Azure Architecture

```text
Developer

↓

Azure Repos

↓

Azure Pipeline

↓

Playwright

↓

HTML

↓

JUnit

↓

Artifacts

↓

Teams

↓

Dashboard

```

----------

# Recommended Folder Structure

```text
azure/

├── smoke.yml

├── regression.yml

├── nightly.yml

├── release.yml

└── templates/

      playwright-template.yml

```

Using YAML templates reduces duplication across pipelines.

----------

# Complete Enterprise Pipeline

```yaml
trigger:

- main

pool:

  vmImage: ubuntu-latest

variables:

- group: QA-Variables

steps:

- checkout: self

- task: NodeTool@0

  inputs:

    versionSpec: '20.x'

- script: npm ci

- script: npx playwright install --with-deps

- script: npx playwright test

- task: PublishPipelineArtifact@1

  inputs:

    targetPath: playwright-report

    artifact: HTMLReport

- task: PublishTestResults@2

  inputs:

    testResultsFormat: JUnit

    testResultsFiles: reports/results.xml

```

----------

# Best Practices

-   Use YAML pipelines instead of Classic pipelines.
    
-   Prefer Microsoft-hosted agents unless self-hosted agents are required for internal network access or specialized software.
    
-   Store credentials in Variable Groups or Azure Key Vault.
    
-   Use `npm ci` for deterministic builds.
    
-   Publish HTML reports, JUnit results, screenshots, videos, and traces.
    
-   Split pipelines into stages (Build, Test, Deploy).
    
-   Reuse YAML templates to avoid duplication across pipelines.
    

----------

# Common Mistakes

### ❌ Hardcoding Credentials

Bad

```yaml
PASSWORD: admin123

```

Good

```text
Variable Group

or

Azure Key Vault

```

----------

### ❌ Single Huge Pipeline

Instead of

```text
Everything

↓

One File

```

Use

```text
Templates

↓

Reusable YAML

```

----------

### ❌ No Artifact Publishing

Without artifacts,

debugging becomes difficult.

Always publish:

-   HTML Reports
    
-   Screenshots
    
-   Videos
    
-   Traces
    

----------

### ❌ Running Everything Sequentially

Leverage:

-   Stages
    
-   Matrix builds
    
-   Parallel jobs
    

to reduce execution time.

----------

### ❌ Ignoring Deployment Gates

Critical production deployments should include approval gates or automated quality checks before release.

----------

# Interview Questions

### Q1. What is the difference between Microsoft-hosted and self-hosted agents?

-   **Microsoft-hosted agents** are temporary virtual machines managed by Microsoft.
    
-   **Self-hosted agents** run on infrastructure managed by your organization and can access internal networks and custom software.
    

----------

### Q2. Why should `npm ci` be used in Azure Pipelines?

It provides clean, reproducible, and faster dependency installation based on the lock file.

----------

### Q3. Why use Variable Groups?

They centralize reusable configuration values and reduce duplication across pipelines.

----------

### Q4. Why integrate Azure Key Vault?

To securely manage and rotate secrets without storing them in pipeline definitions.

----------

### Q5. Why should HTML reports and JUnit results both be published?

-   **HTML reports** provide rich debugging information.
    
-   **JUnit results** integrate with Azure DevOps test reporting and pipeline summaries.
    

----------

# Summary

Azure DevOps Pipelines provides a powerful platform for executing Playwright tests in enterprise environments. By combining YAML pipelines, hosted or self-hosted agents, secure variable management, artifact publishing, matrix strategies, and multi-stage deployments, teams can build scalable, maintainable, and secure CI/CD workflows. Integrating Playwright with Azure DevOps ensures automated quality gates, rich reporting, and rapid feedback throughout the software delivery lifecycle.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTExNDkxNjQwOV19
-->