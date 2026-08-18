# LAB7 Final Entry

## Pipeline File

**File:** `.ado/eshoponweb-ci.yml`

> **Important:** Update the resource name if required.
>
> **Pipeline Name:** `eshoponweb-ci`
>
> The pipeline name should be the same as the YAML file name (without the `.yml` extension).

---

## Azure DevOps YAML Pipeline

```yaml
trigger:
  - main

resources:
  repositories:
    - repository: self
      trigger: none

stages:

# Build Stage
- stage: Build
  displayName: Build .NET Core Solution

  jobs:
    - job: Build
      pool:
        vmImage: ubuntu-latest

      steps:
        - task: DotNetCoreCLI@2
          displayName: Restore
          inputs:
            command: 'restore'
            projects: '**/*.sln'
            feedsToUse: 'select'

        - task: DotNetCoreCLI@2
          displayName: Build
          inputs:
            command: 'build'
            projects: '**/*.sln'

        - task: DotNetCoreCLI@2
          displayName: Test
          inputs:
            command: 'test'
            projects: 'tests/UnitTests/*.csproj'

        - task: DotNetCoreCLI@2
          displayName: Publish
          inputs:
            command: 'publish'
            publishWebProjects: true
            arguments: '-o $(Build.ArtifactStagingDirectory)'

        - task: PublishPipelineArtifact@1
          displayName: Publish Artifacts ADO - Website
          inputs:
            targetPath: '$(Build.ArtifactStagingDirectory)'
            artifact: 'Website'
            publishLocation: 'pipeline'

        - task: PublishPipelineArtifact@1
          displayName: Publish Artifacts ADO - Bicep
          inputs:
            targetPath: '$(Build.SourcesDirectory)/infra/webapp.bicep'
            artifact: 'Bicep'
            publishLocation: 'pipeline'

# Deploy Stage
- stage: Deploy
  displayName: Deploy to an Azure Web App

  jobs:
    - deployment: Deploy
      environment: approvals

      pool:
        vmImage: 'windows-latest'

      strategy:
        runOnce:
          deploy:
            steps:
              - task: DownloadBuildArtifacts@1
                inputs:
                  buildType: 'current'
                  downloadType: 'single'
                  artifactName: 'Website'
                  downloadPath: '$(Build.ArtifactStagingDirectory)'

              - task: AzureRmWebAppDeployment@4
                inputs:
                  ConnectionType: 'AzureRM'
                  azureSubscription: 'Ignite-lod52222156(7371a871-8031-4801-b47b-de5486201753)'
                  # Must match the Azure DevOps Service Connection name

                  appType: 'webApp'
                  WebAppName: 'eshoponWebYAML59701271'
                  packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'

                  AppSettings: >
                    -UseOnlyInMemoryDatabase true
                    -ASPNETCORE_ENVIRONMENT Development
```

---

## Pipeline Summary

### Build Stage

The Build stage performs the following actions:

1. Restores NuGet packages.
2. Builds the .NET solution.
3. Executes unit tests.
4. Publishes the web application.
5. Publishes website artifacts.
6. Publishes the Bicep infrastructure template.

### Deployment Stage

The Deploy stage performs the following actions:

1. Downloads the Website artifact.
2. Deploys the application to an Azure Web App.
3. Uses the configured Azure Resource Manager service connection.
4. Sets application settings during deployment.

### Resources

| Resource | Value |
|-----------|--------|
| Pipeline Name | `eshoponweb-ci` |
| Trigger Branch | `main` |
| Environment | `approvals` |
| Agent (Build) | `ubuntu-latest` |
| Agent (Deploy) | `windows-latest` |
| Artifact Name | `Website` |
| Infrastructure Artifact | `Bicep` |
| Azure Web App | `eshoponWebYAML59701271` |
| ASP.NET Environment | `Development` |
| Database Mode | `UseOnlyInMemoryDatabase = true` |

---

## Expected Azure DevOps Structure

```text
.
├── .ado
│   └── eshoponweb-ci.yml
├── src
├── tests
│   └── UnitTests
├── infra
│   └── webapp.bicep
└── *.sln
```

---

## Validation Checklist

- [ ] Pipeline name is **eshoponweb-ci**
- [ ] YAML file is saved as **.ado/eshoponweb-ci.yml**
- [ ] Azure Service Connection exists and matches the configured name
- [ ] Azure Web App **eshoponWebYAML59701271** exists
- [ ] `infra/webapp.bicep` file exists
- [ ] Unit tests are located under `tests/UnitTests`
- [ ] Solution file (`*.sln`) exists in the repository
- [ ] Deployment environment `approvals` is configured in Azure DevOps
