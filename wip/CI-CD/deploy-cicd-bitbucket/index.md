---
title: Deploying to Posit Connect using Bitbucket Pipelines
---

This page focuses on programmatic deployment using Bitbucket Pipelines as a continuous integration and deployment pipeline. Continuous integration (CI) is the practice of automating the integration of code changes. This automation can entail running different tests or other pre-deployment activities. Continuous deployment (CD) is the practice of automating the deployment of code changes to a test or production environment. Many popular code hosting providers and independent software companies offer CI and CD services. These pipelines allow you to build for specific operating systems/environments, integrate tests and publish to Connect from private repositories without a service account.

The following section examines the deployment of a Shiny for R application to Connect using Bitbucket Pipelines.

## Bitbucket Pipelines example

* [Demo Bitbucket Repository](https://bitbucket.org/lisa-sandbox-code/shiny-app-demo-cicd-bitbucket/src/main/)

The first thing to consider is how to manage the R packages as dependencies within the CI/CD service. One solution is to do a one-by-one installation of every package the Shiny app uses, however, this gets cumbersome as the app grows bigger. To simplify package management of the environment, it is recommended to use the `renv` package. This package helps reproducibility because, for all packages used by your project, it saves both the package version information and the repository location(s) where you installed the packages from so that it can restore the environment on other computers. You use `renv` in this deployment process to maintain consistency between the development and build environments.

Without `renv`:

```bash
install.packages(c(“shiny”, “jsonlite”, “ggplot2”, “stringr”))
```

With `renv`:

```bash
renv::snapshot() #Development Environment
renv::restore() # CI
```

To learn more about `renv`, visit the [package documentation website](https://rstudio.github.io/renv/).

Below you can see how the files are organized in the [Demo Bitbucket repository](https://bitbucket.org/lisa-sandbox-code/shiny-app-demo-cicd-bitbucket/src/main/).

```bash
.
├── .Rbuildignore
├── .Rprofile
├── .gitignore
├── README.md
├── app.R
├── manifest.json
├── bitbucket-pipelines.yml
├── deploy-to-connect.sh
├── install-pandoc.sh
├── set-rspm.sh
├── renv.lock
└── shiny-app-demo-cicd-bitbucket.Rproj

1 directory, 12 files
```

The Shiny application is `app.R`.

The main file of interest is `bitbucket-pipelines.yml`. This file defines the CI/CD pipeline and is set up to run on any push to the `main` branch. First, it sets up the proper environment, including restoring the `renv` environment. Second, it publishes the Shiny application to Connect using the Connect API.

```bash
image: rstudio/r-base:4.1.2-focal

pipelines:
  default:
    - step:
        name: 'Deploy to Connect'
        deployment: production
        script:
          - sudo apt-get update && sudo apt-get install -y libcurl4-openssl-dev libssl-dev curl jq
          - /bin/bash install-pandoc.sh
          - /bin/bash set-rspm.sh
          - sudo rm .Rprofile
          - sudo R -e 'install.packages("renv")'
          - sudo R -e 'renv::consent(TRUE)'
          - sudo R -e 'renv::restore()'
          - sudo R -e 'install.packages("rsconnect")'
          - sudo R -e 'rsconnect::writeManifest()'
          - /bin/bash deploy-to-connect.sh
```