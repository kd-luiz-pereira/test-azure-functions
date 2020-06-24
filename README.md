# Create function

Execute the following command to create your functions, remember that each `FunctionApp` can contain any number of functions but only within the same runtime.

```sh
$ func init FunctionApp --dotnet
$ cd FunctionApp
$ func new Func1 --template "HTTP Trigger"
$ func new Func2 --template "HTTP Trigger"
```

# Set Anonymous 

Just for the propose of this tutorial we going to set all functions permissions to `Anonymous`

```csharp
  ...
  public static async Task<IActionResult> Run(
              [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
              ILogger log)
          {
  ...
```

# Deploy from local

```sh
$ func azure functionapp publish <function-app-name> --charp
```

# Pipeline config refs


(Yaml file config)[https://docs.microsoft.com/pt-pt/azure/azure-functions/functions-how-to-azure-devops?tabs=csharp]
(Plug AppFunctions on AzurePipeline)[https://docs.microsoft.com/pt-pt/azure/azure-functions/functions-continuous-deployment]

Final yml sample
```yaml
pool:
  name: Azure Pipelines
  demands: npm

steps:
- task: DotNetCoreCLI@2
  displayName: 'dotnet build'
  inputs:
    projects: '*.csproj'
    arguments: '--configuration Release --output publish_output'

- task: Npm@1
  displayName: 'npm install'
  inputs:
    workingDir: '$(System.DefaultWorkingDirectory)'
    verbose: false

- task: ArchiveFiles@2
  displayName: 'Archive files'
  inputs:
    rootFolderOrFile: '$(System.DefaultWorkingDirectory)/publish_output'
    includeRootFolder: false

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'

- task: AzureFunctionApp@1
  displayName: 'Azure Function App Deploy: kd-luiz-pereira-funcs'
  inputs:
    azureSubscription: 'Free Trial (5b5a3a8a-f273-45c3-bd79-bbf09eda2359)'
    appType: functionApp
    appName: 'kd-luiz-pereira-funcs'
    package: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'

```