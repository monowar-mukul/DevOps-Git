## This is LAB7 Final entry              .ado/eshoponweb-ci.yml [ Update resource name]


# NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
# trigger:
# - main

resources:
  repositories:
    - repository: self
      trigger: none

stages:
- stage: Build
  displayName: Build .Net Core Solution
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

- stage: Deploy
  displayName: Deploy to an Azure Web App
  jobs:
    - deployment: Deploy
      environment: approvals
      pool:
        vmImage: "windows-latest"
      strategy:
        runOnce:
          deploy:
            steps:
              - task: DownloadBuildArtifacts@1
                inputs:
                  buildType: "current"
                  downloadType: "single"
                  artifactName: "Website"
                  downloadPath: "$(Build.ArtifactStagingDirectory)"
              - task: AzureRmWebAppDeployment@4
                inputs:
                  ConnectionType: "AzureRM"
                  azureSubscription: 'Ignite-lod52222156(7371a871-8031-4801-b47b-de5486201753)' # must match service connection name
                  appType: "webApp"
                  WebAppName: 'eshoponWebYAML59701271'
                  packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                  AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
