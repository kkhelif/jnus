# GitHub Action - JSON Variables
Are you tired of repeating and duplicating environment variables for environment specific deployment steps and having to maintain multiple workflows dependent on the same set of variables? The JSON variables action lets you define all you variable in a single json file, scope to environments and substitute/reuse in new variables, hence simplifying variable configuration maintenance.

Based on a json file, this GitHub Action will:
  1. Render variables based on the environment context specified in the workflow
  2. Substitute variables in other variables
  3. Set variables as environment variables.

# Getting started

Add a variable file as json anywhere in the repository.
The format of the file should be:

```json
{
  "Variables": [
    {
      "Name": "HostName",
      "Value": "someDevHostName",
      "Scope": {
        "Environment": [
          "Dev"
        ]
      }
    },
    {
      "Name": "HostName",
      "Value": "someDevTestHostName",
      "Scope": {
        "Environment": [
          "DevTest"
        ]
      }
    }
  ]
}
```


Now in your environment specific job, you want to render this set of variables as environment variables, add the following action. The configFile parameter is the full path to the variable file, relative to the repository root path. Scope is the environment name, which is case sensitive. The secrets parameter need to be either a serialized json string of e.g. the secrets context object, or a user defined serialized object with the same structure:

```json
- name: Set environment specific variables
  uses: jnus/jsonvariables@v0.1
  with:
    scope: Dev
    configFile: 'variables.minimal.json'
    secrets: '${{toJson(secrets)}}'
```
 
The environment specific variables defined variables.json will be created as environment variables ready to use for poking e.g. appsettings.json file or as parameters for deploying to misc. compute targets. ~~Note the job must contain an Environment declaration and will only scoped to the particular job.~~

# Variable substitution.
The JSON variables action can do variable substitution, hence simplifying the variable maintenance. It's a flexible concept for adjusting the environment variables based on the environment context of your e.g. deployment job. By using a simple expression syntax, variables can be combined and thereby reducing the complexity of maintaining multiple environments. 

The following variable 'Url' reference the 'HostName' variable and based on the environment context, renders the environment variable as either https://someDevHostName or https://someDevTestHostName. 

```json
{
  "Variables": [
    {
      "Name": "HostName",
      "Value": "someDevHostName",
      "Scope": {
        "Environment": [
          "Dev"
        ]
      }
    },
    {
      "Name": "HostName",
      "Value": "someDevTestHostName",
      "Scope": {
        "Environment": [
          "DevTest"
        ]
      }
    },
    {
      "Name": "Url",
      "Value": "https://#{HostName}.com",
      "Scope": {}
    }
  ]
}
```

# Filter Expressions
Variable substitution supports filter expressions, to filter or manipulate substitution values. Currently the following filter expressions are supported

| Filter Expression | Syntax   |Description|
|-------------------|----------|-----------|
| ToLower | #{Environment \| ToLower } | Sets the entire value to lower casing|
| ToUpper | #{Environment \| ToUpper }| Sets the entire value to upper casing|

# Action Context Variables
When configuring the json variables, you have access to a series of context variables, which can be used as substitutions.  

| Variable | Syntax | Description |
|----------|----------|------------|
| Context.Environment | #{Context.Environment} | Will substitute current environment context specified as the input parameter 'scope'  | 
| ${{secrets.*}} | ${{secrets.A_REPO_SECRET}} | All the secrets, whether it is organization, environment or repository secrets, are accessible with the Github expression syntax ${{secrets.[variable name]}} | 



# How to use environment specific environment variables
The context specific environment variables, rendered in this action, can be reference as any other environment variable in the current job and implicitly be used by other actions such as microsoft/variable-substitution.

The following job, deploys a web app and uses the rendered 'HostName' variable to target a particular resource in Azure and 'Url' as the fqdn for the web app:

```yaml
deploy_to_dev:
    runs-on: ubuntu-latest
        ...
    environment: 
      name: Dev
      url: ${{env.Url}}
    steps:
      - name: Set environment specific variables
        uses: jnus/jsonvariables@main
        with:
            scope: Dev
            configFile: 'variables.minimal.json'
            secrets: '${{ toJson(secrets) }}'
        ...
      - name: Deploy to Azure WebApp
        uses: azure/webapps-deploy@v2
        id: web-app
        with:
          app-name: ${{env.HostName}}-api-wa
          package: ./webapp/bin/Release/net5.0/publish
```

# Status
Action is not yet published and currently being tested. It is not intended for production usage yet. Certain features are missing before v1.0 can be created
- Read environment context directly in action and not as a parameter
- Read secrets context directly in action and not as a parameter
- Support for multiple json files for e.g. sub-moduling global variable sets. 

Topic regarding this -> https://github.community/t/variable-management-in-github/200400?u=jnus
