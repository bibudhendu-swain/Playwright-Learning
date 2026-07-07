This is one of the most valuable chapters in the entire handbook because **Docker has become the standard execution environment for Playwright**.

In modern enterprises, Playwright tests are increasingly executed inside Docker containers rather than directly on developer machines or CI agents. This ensures consistency, eliminates environment-specific issues, and simplifies CI/CD pipelines.

This chapter goes beyond basic Docker commands and focuses on **using Docker effectively with Playwright**.

----------

# Part 21 – CI/CD

# Chapter 5 – Docker & Playwright

----------

# Introduction

One of the biggest challenges in automation is the classic problem:

> "It works on my machine."

Different developers may have:

-   Different Node.js versions
    
-   Different browser versions
    
-   Different operating systems
    
-   Different libraries
    
-   Missing dependencies
    

Docker solves this by packaging the application, Playwright, browsers, and dependencies into a single portable container.

----------

# What is Docker?

Docker is a containerization platform.

Instead of installing software directly on your machine,

everything runs inside an isolated container.

```text
Application

↓

Docker Image

↓

Docker Container

```

Containers are lightweight, portable, and reproducible.

----------

# Why Docker for Playwright?

Without Docker

```text
Developer A

↓

Chrome 138

Node 20

Windows

------------------

Developer B

↓

Chrome 140

Node 22

Linux

```

Result

```text
Different Behavior

```

With Docker

```text
Playwright Docker Image

↓

Same Node Version

↓

Same Browser Version

↓

Same Libraries

↓

Same Environment

```

Every developer and CI pipeline uses the same environment.

----------

# Docker Architecture

```text
Playwright Project

↓

Docker Image

↓

Docker Container

↓

Execute Tests

↓

Generate Reports

```

----------

# Docker Components

Component

Purpose

Image

Blueprint

Container

Running instance

Dockerfile

Image definition

Registry

Stores images

Volume

Persistent storage

Network

Communication

----------

# Official Playwright Docker Image

Microsoft maintains official Docker images.

Example

```text
mcr.microsoft.com/playwright:v1.55.0-noble

```

These images already include:

-   Chromium
    
-   Firefox
    
-   WebKit
    
-   System dependencies
    
-   Node.js
    
-   Playwright
    

No browser installation required.

----------

# Benefits of Official Images

```text
No Browser Installation

↓

No Dependency Problems

↓

Consistent Execution

↓

Fast CI Setup

```

Highly recommended.

----------

# Project Structure

```text
project/

├── Dockerfile

├── package.json

├── playwright.config.ts

├── tests/

└── docker-compose.yml

```

----------

# Basic Dockerfile

```dockerfile
FROM mcr.microsoft.com/playwright:v1.55.0-noble

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]

```

----------

# Dockerfile Breakdown

```dockerfile
FROM

```

Base image

↓

```dockerfile
WORKDIR

```

Working directory

↓

```dockerfile
COPY

```

Copy files

↓

```dockerfile
RUN

```

Install dependencies

↓

```dockerfile
CMD

```

Default command

----------

# Build Docker Image

```bash
docker build -t playwright-demo .

```

Creates

```text
Docker Image

↓

playwright-demo

```

----------

# Run Container

```bash
docker run playwright-demo

```

Container executes

```text
Playwright Tests

↓

Exit

```

----------

# Interactive Container

```bash
docker run -it playwright-demo bash

```

Useful for:

-   Debugging
    
-   Running commands
    
-   Inspecting files
    

----------

# Mounting Volumes

Without volumes,

reports disappear when the container exits.

Use

```bash
docker run \
-v $(pwd)/playwright-report:/app/playwright-report \
playwright-demo

```

Workflow

```text
Container

↓

Generate Report

↓

Host Machine

↓

playwright-report/

```

Reports persist after container removal.

----------

# Environment Variables

Pass environment values.

```bash
docker run \
-e BASE_URL=https://qa.company.com \
playwright-demo

```

Inside Playwright

```typescript
process.env.BASE_URL

```

----------

# Docker Compose

Useful when multiple services are required.

Example

```yaml
version: "3"

services:

  playwright:

    build: .

    volumes:

      - ./playwright-report:/app/playwright-report

```

Run

```bash
docker compose up

```

----------

# Why Docker Compose?

Example

```text
Playwright

↓

Application

↓

Database

↓

Redis

```

All services start together.

----------

# Running Specific Tests

```bash
docker run playwright-demo \
npx playwright test login.spec.ts

```

----------

# Running Smoke Suite

```bash
docker run playwright-demo \
npx playwright test --grep @smoke

```

----------

# Running Specific Browser

```bash
docker run playwright-demo \
npx playwright test --project=chromium

```

----------

# Debugging Inside Container

```bash
docker exec -it <container-id> bash

```

Useful for:

-   Checking files
    
-   Running Playwright manually
    
-   Viewing logs
    

----------

# Copying Reports

If reports were not mounted,

copy them.

```bash
docker cp \
container:/app/playwright-report \
./playwright-report

```

----------

# Using Host Network

Sometimes required for internal applications.

```bash
docker run --network host playwright-demo

```

This option is primarily available on Linux. On macOS and Windows, Docker Desktop handles networking differently.

----------

# Running Headed Tests

Containers usually execute

```text
Headless

```

Headed execution requires additional configuration such as an X server or VNC.

Most CI pipelines use headless mode.

----------

# Docker Layers

Efficient Dockerfiles improve build performance.

Good order

```dockerfile
COPY package*.json

RUN npm ci

COPY .

```

Reason

```text
Dependencies Cached

↓

Faster Rebuild

```

----------

# Multi-Stage Build

Example

```dockerfile
FROM node:20 AS build

FROM mcr.microsoft.com/playwright:v1.55.0-noble

```

Advantages

-   Smaller images
    
-   Better security
    
-   Cleaner builds
    

----------

# Running in CI

Pipeline

```text
CI Pipeline

↓

Docker Build

↓

Run Container

↓

Playwright Tests

↓

Reports

```

Same image locally and in CI.

----------

# Docker Registry

Images are commonly stored in

```text
Docker Hub

Azure Container Registry

GitHub Container Registry

Amazon ECR

```

Pipeline

```text
Build

↓

Push Image

↓

CI Pulls Image

↓

Execute

```

----------

# Kubernetes Overview

Large organizations execute Playwright containers in Kubernetes.

```text
Kubernetes

↓

Pod

↓

Playwright Container

↓

Reports

```

Although Playwright doesn't require Kubernetes, container orchestration platforms make scaling and scheduling easier.

----------

# Enterprise Architecture

```text
Developer

↓

Git

↓

CI Pipeline

↓

Docker Image

↓

Playwright Container

↓

Reports

↓

Artifacts

↓

Dashboard

```

----------

# Recommended Docker Strategy

```text
Official Playwright Image

↓

Dockerfile

↓

Docker Compose

↓

CI Pipeline

↓

Registry

↓

Production

```

----------

# Complete Enterprise Dockerfile

```dockerfile
FROM mcr.microsoft.com/playwright:v1.55.0-noble

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npx playwright install

CMD ["npx","playwright","test"]

```

> **Note:** When using the official Playwright image, `npx playwright install` is often unnecessary because browsers are already included. Include it only if your project requires additional browser installation or verification.

----------

# Best Practices

-   Use the official Playwright Docker image whenever possible.
    
-   Mount report directories as volumes so reports survive container shutdown.
    
-   Pass configuration through environment variables rather than modifying images.
    
-   Optimize Docker layer caching by copying dependency files before application files.
    
-   Keep Docker images small and focused.
    
-   Use Docker Compose when testing against multiple dependent services.
    
-   Prefer headless execution in CI.
    

----------

# Common Mistakes

### ❌ Installing Browsers Every Time

The official Playwright image already includes browsers.

Avoid unnecessary installation steps.

----------

### ❌ Storing Reports Inside Container

Without volume mounts,

reports disappear when containers are removed.

----------

### ❌ Hardcoding Environment Values

Bad

```dockerfile
ENV BASE_URL=https://qa.company.com

```

Better

```bash
docker run \
-e BASE_URL=...

```

----------

### ❌ Using Large Images

Install only required dependencies.

Remove unnecessary tools.

----------

### ❌ Ignoring Layer Caching

Poor Dockerfile ordering increases build time significantly.

----------

# Interview Questions

### Q1. Why use Docker with Playwright?

Docker provides a consistent execution environment across developer machines, CI/CD pipelines, and production infrastructure.

----------

### Q2. Why are Microsoft's official Playwright Docker images recommended?

Because they already include Playwright, supported browser binaries, Node.js, and required system dependencies, reducing setup complexity.

----------

### Q3. Why should reports be stored in mounted volumes?

Mounted volumes preserve reports after the container exits, making them available for debugging and CI artifact publishing.

----------

### Q4. What is the purpose of Docker Compose?

Docker Compose allows multiple related services (such as Playwright, an application server, and a database) to be started and managed together.

----------

### Q5. Why should environment variables be passed at runtime?

It keeps Docker images reusable across different environments (QA, Staging, Production) without rebuilding the image.

----------

# Summary

Docker provides a portable, consistent, and reproducible execution environment for Playwright automation. By using the official Playwright Docker images, mounting report volumes, passing configuration through environment variables, and integrating containers into CI/CD pipelines, teams can eliminate environment-specific issues and build reliable, scalable automation infrastructure. Containerized Playwright execution has become the standard approach for modern enterprise testing.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MDQ3MzMxMDVdfQ==
-->