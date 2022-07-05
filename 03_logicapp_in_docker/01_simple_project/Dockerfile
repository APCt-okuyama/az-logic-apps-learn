FROM mcr.microsoft.com/azure-functions/node:3.0
ENV AzureWebJobsStorage <set your storage account connect string>
ENV AzureWebJobsScriptRoot=/home/site/wwwroot AzureFunctionsJobHost__Logging__Console__IsEnabled=true FUNCTIONS_V2_COMPATIBILITY_MODE=true
ENV WEBSITE_HOSTNAME localhost
ENV WEBSITE_SITE_NAME myexamplelogicapp
ENV AZURE_FUNCTIONS_ENVIRONMENT Development
COPY . /home/site/wwwroot