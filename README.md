README By ChatGPT, partially reviewed by myself!

# 🚀 AWS Amplify Build Optimization

## Prebuilding `node_modules` with Docker + GitHub Actions + ECR

------------------------------------------------------------------------

## 📌 Overview

This repository demonstrates a practical optimization strategy for **AWS
Amplify Node.js builds**.

Instead of running:

``` bash
npm ci
```

inside every Amplify build (which increases build time and cost), this
approach:

-   🐳 Builds a Docker image that installs `node_modules`
-   ⚙️ Uses GitHub Actions to build & push the image to AWS ECR
-   🔁 Triggers only when `package-lock.json` changes
-   📦 Amplify pulls the prebuilt image and reuses dependencies

### 🎯 Result

  Scenario                   Build Time
  -------------------------- ------------------
  Default Amplify (npm ci)   \~11--12 minutes
  Prebuilt image strategy    \~5--6 minutes

This reduces: - Amplify build duration\
- Amplify billed minutes\
- Deployment latency\
- CI wait time

------------------------------------------------------------------------

# 🏗 Architecture

``` mermaid
flowchart LR
    A[GitHub Repository] --> B[GitHub Actions]
    B -->|On package-lock change| C[Build Docker Image]
    C --> D[Push to AWS ECR]
    E[AWS Amplify Build] --> F[Pull Image from ECR]
    F --> G[Reuse node_modules + Build App]
```

------------------------------------------------------------------------

# 📂 Repository Structure

This repository contains only the files required to demonstrate the
optimization pattern:

    Dockerfile.build
    .github/workflows/amplify-build-image.yml
    package.json
    package-lock.json
    amplify.yaml

Each file plays a specific role in the optimization flow.

------------------------------------------------------------------------

# 🐳 Dockerfile.build

The Dockerfile installs dependencies once and bakes them into the image.

``` dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .
```

Key idea: - Dependencies are installed inside Docker - The resulting
image contains a ready-to-use `node_modules`

------------------------------------------------------------------------

# ⚙️ GitHub Actions Workflow

Located in:

    .github/workflows/amplify-build-image.yml

This workflow:

-   Runs only when `package-lock.json` changes
-   Builds the Docker image
-   Pushes it to AWS ECR
-   Tags it as `latest`

Example simplified job:

``` yaml
on:
  push:
    paths:
      - 'package-lock.json'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/amazon-ecr-login@v2
      - run: |
          docker build -f Dockerfile.build -t $ECR_REGISTRY/my-app:latest .
          docker push $ECR_REGISTRY/my-app:latest
```

This ensures dependency images are rebuilt only when necessary.

------------------------------------------------------------------------

# 📦 Amplify Build (amplify.yaml)

Amplify no longer runs `npm ci`.

Instead, it pulls the prebuilt image:

``` yaml
version: 1
applications:
  - frontend:
      phases:
        preBuild:
          commands:
            - docker pull $ECR_REGISTRY/my-app:latest
        build:
          commands:
            - npm run build
```

This avoids reinstalling dependencies inside Amplify.

------------------------------------------------------------------------

# 💡 Why This Saves Money

Amplify charges for build minutes.

Installing dependencies during every build: - Consumes significant
time - Increases CI duration - Increases cost

By moving dependency installation to GitHub Actions and only rebuilding
when the lockfile changes, we: - Reduce repeated work - Make builds
deterministic - Cut build time nearly in half

------------------------------------------------------------------------

# 🧠 When to Use This Pattern

This technique is ideal when:

-   Your project has heavy dependencies
-   Amplify builds are frequent
-   `npm ci` is the slowest step
-   You want predictable builds tied to `package-lock.json`

------------------------------------------------------------------------

# 📢 LinkedIn Context

This repository accompanies the following idea:

> "While having fun developing my new personal full stack (AWS Amplify)
> app, I discovered that avoiding `npm ci` inside Amplify builds and
> instead prebuilding dependencies in a Docker image (triggered only
> when `package-lock.json` changes) can reduce build time from \~12
> minutes to \~6 minutes."

------------------------------------------------------------------------

# 🤝 Contribution

This repository is intended to demonstrate a CI/CD optimization
technique.\
Feel free to fork, adapt, and improve.

------------------------------------------------------------------------

# 📬 Author

Created by **Fares Belhaouas**\
AWS Amplify enthusiast & full-stack developer
