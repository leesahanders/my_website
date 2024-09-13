---
title: Deploying to Posit Connect using Azure Pipelines
---

This guide focuses on programmatic deployment using Azure Pipelines as a continuous integration and deployment pipeline. Continuous integration (CI) is the practice of automating the integration of code changes. This automation can entail running different tests or other pre-deployment activities. Continuous deployment (CD) is the practice of automating the deployment of code changes to a test or production environment. Many popular code hosting providers and independent software companies offer CI and CD services. These pipelines allow you to build for specific operating systems/environments, integrate tests and publish to Connect from private repositories without a service account.

The following section examines the deployment of a Shiny for R application to Connect using Azure Pipelines.

## Azure Pipelines example

* [Demo Azure Repos Repository](https://dev.azure.com/trevornederlof/_git/shiny-app-demo-cicd-azure)

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

To learn more about renv, visit the [package documentation website](https://rstudio.github.io/renv/).

Below you can see how the files are organized in the [Demo Azure Repos repository](https://dev.azure.com/trevornederlof/_git/shiny-app-demo-cicd-azure).

```bash
.
├── .Rbuildignore
├── .Rprofile
├── .gitignore
├── README.md
├── app
│   ├── app.R
│   └── manifest.json
├── azure-pipelines.yml
├── deploy-to-connect.sh
├── install-pandoc.sh
├── renv.lock
├── set-rspm.sh
└── shiny-app-demo-cicd-azure.Rproj

1 directory, 12 files
```

The Shiny application is contained in `app/app.R`.

The main file of interest is `azure-pipelines.yml`. This file defines the CI/CD pipeline, and is set up to run on any push to the `main` branch. First, it sets up the proper environment, including restoring the `renv` environment. Second, it publishes the Shiny application to Connect using the Connect API.

```bash
        trigger:
        - main

        pool:
        vmImage: 'ubuntu-22.04'

        container: rstudio/r-base:4.1.2-focal

        variables:
        - group: 'RSC'
        - name: CONTENT_DIRECTORY
        value: app
        - name: UNIQ_NAME
        value: shiny-app-demo-cicd-azure

        steps:
        - task: Bash@3
        displayName: 'Install dependencies'
        inputs:
            targetType: 'inline'
            script: sudo apt-get update && sudo apt-get install -y libcurl4-openssl-dev libssl-dev curl jq

        - task: Bash@3
        displayName: 'Install pandoc'
        inputs:
            filePath: 'install-pandoc.sh'

        - task: Bash@3
        displayName: 'Setup RSPM'
        inputs:
            filePath: 'set-rspm.sh'

        - task: Bash@3
        displayName: 'Remove Rprofile'
        inputs:
            targetType: 'inline'
            script: sudo rm .Rprofile

        - task: Bash@3
        displayName: 'Install renv'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'install.packages("renv")'
            
        - task: Bash@3
        displayName: 'renv consent'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'renv::consent(TRUE)'

        - task: Bash@3
        displayName: 'renv restore'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'renv::restore()'

        - task: Bash@3
        displayName: 'Install rsconnect'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'install.packages("rsconnect")'

        - task: Bash@3
        displayName: 'Write manifest.json'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'rsconnect::writeManifest("$(CONTENT_DIRECTORY)")'

        - task: Bash@3
        displayName: 'Create Upload Deploy'
        inputs:
            filePath: 'deploy-to-connect.sh'
```

The rest of this section reviews the file’s contents so you understand in detail what is happening.

The beginning of `azure-pipelines.yml` specifies the event that triggers the pipeline. In this case, it is a push to the main branch:

```bash
    trigger:
    - main
```

Next, `vmImage` sets the Microsoft-hosted agent for the pipeline, which allows for the specification of what OS should be used on the virtual machine. `vmImage` is listed under `pool:`, as it is choosing a Microsoft-hosted agent pool from the Azure Pipelines pool. There are many options associated with creating and managing agent pools. We can use the `rstudio/r-base:4.1.2-focal` image to provide a container with R installed, since this CI/CD pipeline will be running R code and deploying a Shiny app. Any [Docker Hub](https://hub.docker.com/) image can be used. For a full list of available images with versions of R pre-installed, visit the [rstudio/r-base repository](https://hub.docker.com/r/rstudio/r-base).

```bash
    pool:
    vmImage: 'ubuntu-22.04'

    container: rstudio/r-base:4.1.2-focal
```

To successfully deploy to Connect, this pipeline needs several environment variables. `CONNECT_SERVER` and `CONNECT_API_KEY` should be set up at the pipeline level by going to **Pipelines** on the left side of the screen. Then, click on the **Library** sub-menu item, create a Variable Group, and set each as a Variable. The screenshot below illustrates where to go in the Azure settings. See the [Create an API Key](../../user/api-keys/index.md) and [Find your Connect server URL](../../cookbook/index.md#getting-started) sections for more information on those actions.

![](../img/deploy-cicd/azure_pipeline_environment_variables.png){fig-alt="Azure Pipeline environment variables" class="border"}

We can also set up local environment variables (specific to this project and not containing sensitive credentials). `CONTENT_DIRECTORY` specifies the directory housing the actual application code (app.R in this case). `UNIQ_NAME` specifies the name of the application.

```bash
    variables:
    - group: 'RSC'
    - name: CONTENT_DIRECTORY
    value: app
    - name: UNIQ_NAME
    value: shiny-app-demo-cicd-azure
```

The rest of the `azure-pipelines.yml` file determines a series of steps to be performed, the last of which deploys the Shiny application to the Connect server. For each of these steps, `- task: Bash@3` indicates that the step is a bash command with either a `targetType: 'inline'` followed by `script:` if the code is defined in the `azure-pipelines.yml` script directly, or `filePath:` if it references an external bash script.

The first three steps install system dependencies and set up [Package Manager](https://www.posit.co/products/enterprise/package-manager/) as the default R repository for faster package installation from binaries. If your organization does not have Package Manager, it is recommended to use Posit [Public Package Manager](https://packagemanager.posit.co/client/#/) for access to binaries.

```bash
        steps:
        - task: Bash@3
        displayName: 'Install dependencies'
        inputs:
            targetType: 'inline'
            script: sudo apt-get update && sudo apt-get install -y libcurl4-openssl-dev libssl-dev curl jq

        - task: Bash@3
        displayName: 'Install pandoc'
        inputs:
            filePath: 'install-pandoc.sh'

        - task: Bash@3
        displayName: 'Setup RSPM'
        inputs:
            filePath: 'set-rspm.sh'
```

The next four steps restore the environment. First, the `.RProfile` is removed to avoid conflicts with `renv`. Then we install `renv` and restore the renv environment. Using `renv` is recommended over manually installing packages, as mentioned at the beginning of this section.

```bash
        - task: Bash@3
        displayName: 'Remove Rprofile'
        inputs:
            targetType: 'inline'
            script: sudo rm .Rprofile

        - task: Bash@3
        displayName: 'Install renv'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'install.packages("renv")'
            
        - task: Bash@3
        displayName: 'renv consent'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'renv::consent(TRUE)'

        - task: Bash@3
        displayName: 'renv restore'
        inputs:
            targetType: 'inline'
            script: sudo R -e 'renv::restore()'
```

After the proper environment is set up we create the `manifest.json` file required by Connect for deploying content programmatically. The directory argument is set to `$(CONTENT_DIRECTORY)`, which specifies the directory where our content app.R is located. The value for this variable, “app” was set at the beginning of the pipeline.

```bash
    - task: Bash@3
    displayName: 'Write manifest.json'
    inputs:
        targetType: 'inline'
        script: sudo R -e 'rsconnect::writeManifest("$(CONTENT_DIRECTORY)")'
```

The last step is to run the `deploy-to-connect.sh` script, which interfaces with the Connect API and deploys the Shiny application. This script utilizes both the pipeline defined environment variables and the locally defined variables to determine the server location, API key, content directory, and unique name.

```bash
        - task: Bash@3
        displayName: 'Create Upload Deploy'
        inputs:
            filePath: 'deploy-to-connect.sh'
```