---
title: Terminal
lastmod: 2024-07-26T14:01:07-05:00
---
# Terminal
```powershell
function core {
	& cd C:\Repos\LeadCrumb\DC.Atlas\Core.Api
	& dotnet run
}

function inven {
	& cd C:\Repos\Atlas-InventoryService\Inventory.Api
	& dotnet run
}

function web {
	& cd C:\Repos\LeadCrumb\WebCRM
	& npm run watch
}

function tests {
	& cd C:\Repos\LeadCrumb\DC.Atlas\DC.Database.Mock
	& docker compose -f tests.docker-compose.yml up -d
	& cd ~
}

function creds {
	C:\Repos\Atlas-DataProcessingService\Get-AwsCredentials.ps1
}

# https://drivecentric.atlassian.net/wiki/spaces/BM/pages/959479839/Using+Entity+Framework+Core
$env:ASPNETCORE_ENVIRONMENT='Local'
```
## Nuget Restore
```powershell
dotnet restore solution.sln --no-cache
```
## Test Amazon Lambda
```powershell
dotnet lambda package -pl .\Inventory.Services.PdfGeneration\ -o .\test.zip -f net8.0
```