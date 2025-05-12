# Azure-web-app-for-active-directory

‣ I created an Azure web app that uses Azure Active Directory (Microsoft Entra ID) authentication.

 ![completed-azure-active-directory](https://github.com/user-attachments/assets/cbefebb5-643c-4510-8593-53b01778e808)

name: Build and deploy ASP app to Azure Web App - MyWebApp-51205482

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest
    permissions:
      contents: read #This is required for actions/checkout

    steps:
      - uses: actions/checkout@v4

      - name: Setup MSBuild path
        uses: microsoft/setup-msbuild@v1.0.2

      - name: Setup NuGet
        uses: NuGet/setup-nuget@v1.0.5

      - name: Restore NuGet packages
        run: nuget restore

      - name: Publish to folder
        run: msbuild /nologo /verbosity:m /t:Build /t:pipelinePreDeployCopyAllFilesToOneFolder /p:_PackageTempDir="\published\"

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v4
        with:
          name: ASP-app
          path: '/published/**'

  deploy:
    runs-on: windows-latest
    needs: build
    environment:
      name: 'Production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}
    
    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v4
        with:
          name: ASP-app
      
      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v3
        with:
          app-name: 'MyWebApp-51205482'
          slot-name: 'Production'
          package: .
          publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE_92D5F50CAA704AB7A9B45EB5990BD328 }}

