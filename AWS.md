---
title: AWS
lastmod: 2024-01-02T11:28:03-06:00
---
# AWS
## AWS Auth
### Credential fetching code
You can use a default credentials flag in your `IOptions` configuration for the amazon client environment variables.
```csharp
public async Task<IAmazonS3> CreateClientAsync()
{
	if (_amazonConfig.UseDefaultCredentials)
	{
		return new AmazonS3Client(RegionEndpoint.GetBySystemName(_amazonConfig.Region));
	}
	else
	{
		var stsClient = new AmazonSecurityTokenServiceClient(new AmazonSecurityTokenServiceConfig
		{
			RegionEndpoint = RegionEndpoint.GetBySystemName(_amazonConfig.Region)
		});

		var assumeRoleRequest = new AssumeRoleRequest
		{
			RoleArn = _amazonConfig.RoleArn,
			RoleSessionName = _amazonConfig.RoleSessionName
		};

		var response = await stsClient.AssumeRoleAsync(assumeRoleRequest);
		var credentials = response.Credentials;
		return new AmazonS3Client(credentials, RegionEndpoint.GetBySystemName(_amazonConfig.Region));
	}
}
```
### Credential fetching script
This script depends PowerShell 7 (which you should already be using anyway) because it uses `ConvertFrom-Json -AsHashTable`.

```powershell
if ((Get-Command -Name aws.exe | measure).Count -eq 0)
{
	Write-Output "AWS CLI not installed."
	Write-Output "Download and install from https://awscli.amazonaws.com/AWSCLIV2.msi"
	exit 1;
}

if ((Test-Path $pwd\Properties\launchSettings.json) -eq $false)
{
	Write-Output "Run this script from the root of an API project."
	exit 1;
}

# Constants
$configFile = "$env:USERPROFILE\.aws\config"
$settingsFile = "$env:USERPROFILE\.aws\settings"
$startUrl = "https://example.awsapps.com/start#"
$AwsMyRoleId = "123456789000"
$region = "us-east-1"
$format = "json"
$roleName = "MyRole"
$grantType = "urn:ietf:params:oauth:grant-type:device_code"
[datetime]$epoch = '1970-01-01 00:00:00'

$profileContent = @"
[default]
sso_session = sso
sso_account_id = $AwsMyRoleId
sso_role_name = $roleName
region = $region
output = $format
[sso-session sso]
sso_start_url = $startUrl
sso_region = $region
sso_registration_scopes = sso:account:access
"@

# Create the config file
# This can also be done manually by running "aws configure sso"
if (!(Test-Path $configFile -PathType Leaf))
{
	$null = New-Item -Path $configFile -Force
	$profileContent | Set-Content $configFile
}

# login to AWS SSO
# 3 hour session
# tracking this session expiry is difficult so we will just re-login every time
Write-Output "Logging into AWS SSO; allow access on browser..."
$null = aws sso login
Write-Output "Successfully logged into AWS SSO."

# initialize the settings file
If (!(Test-Path $settingsFile))
{
	$null = New-Item -Path $settingsFile -Force
	$settings = New-Object -TypeName PSObject
	$settings | Add-Member -MemberType NoteProperty -Name ClientId -Value ""
	$settings | Add-Member -MemberType NoteProperty -Name ClientSecret -Value ""
	$settings | Add-Member -MemberType NoteProperty -Name ClientExpiry -Value $epoch
	$settings | Add-Member -MemberType NoteProperty -Name AccessToken -Value ""
	$settings | Add-Member -MemberType NoteProperty -Name AccessTokenExpiry -Value $epoch
	$settings | ConvertTo-Json -depth 32 | Set-Content $settingsFile
}

# Read the settings file
$settings = Get-Content $settingsFile -Raw | ConvertFrom-Json
$clientId = $settings.ClientId
$clientSecret = $settings.ClientSecret
$clientExpiry = $settings.ClientExpiry
$accessToken = $settings.AccessToken
$accessTokenExpiry = $settings.AccessTokenExpiry

# Get your account Id
$accountId = aws sts get-caller-identity --query Account --output text
if (-not [bool]$accountId) { Write-Output "Unable to get account Id." exit 1; }

# Register a new client
# 3 month expiry
if ($clientExpiry -lt (Get-Date))
{
	Write-Output ""
	Write-Output "Registering client with AWS."
	$hostname = hostname
	$client = aws sso-oidc register-client --client-name $hostname --client-type public | ConvertFrom-Json
	if (-not [bool]$client) { Write-Output "Unable to get register client." exit 1; }
	$clientId = $client.clientId
	$clientSecret = $client.clientSecret
	$clientExpiry = $epoch.AddSeconds($client.clientSecretExpiresAt)
	Write-Output "Client successfully registered."
}

# Set up the authorization code flow
# 7 hour expiry
if ($accessTokenExpiry -lt (Get-Date))
{
	Write-Output ""
	Write-Output "Starting device authorization code flow."
	$device = aws sso-oidc start-device-authorization --client-id $clientId --client-secret $clientSecret --start-url $startUrl | ConvertFrom-Json
	if (-not [bool]$device) { Write-Output "Unable to get start device authorization." exit 1; }
	$deviceCode = $device.deviceCode
	start $device.verificationUriComplete
	Write-Output "Before continuing, authorize this device by allowing access on browser..."
	Write-Output "Press [Enter] when completed."
	$null = [System.Console]::ReadKey()
	$createdToken = aws sso-oidc create-token --client-id $clientId --client-secret $clientSecret --grant-type $grantType --device-code $deviceCode | ConvertFrom-Json
	if (-not [bool]$createdToken) { Write-Output "Unable to get create token." exit 1; }
	$accessToken = $createdToken.accessToken
	$accessTokenExpiry = (Get-Date).AddSeconds($createdToken.expiresIn)
	Write-Output "Successfully set up authorization code flow for this device."
}

# Get the role credentials
Write-Output ""
Write-Output "Generating role credentials."
$credentials = aws sso get-role-credentials --role-name $roleName --account-id $accountId --access-token $accessToken | ConvertFrom-Json
if (-not [bool]$credentials) { Write-Output "Unable to get role credentials." exit 1; }
$key = $credentials.roleCredentials.accessKeyId
$secret = $credentials.roleCredentials.secretAccessKey
$token = $credentials.roleCredentials.sessionToken
$tokenExpiry = $epoch.AddSeconds($credentials.roleCredentials.expiration / 1000)
Write-Output "Successfully generated role credentials."

# Write credentials to launchsettings.Json
Write-Output ""
Write-Output "Writing credentials to launchsettings.json."
$launchSettings = Get-Content $pwd\Properties\launchSettings.json -Raw | ConvertFrom-Json -AsHashTable
if (-not [bool]$launchSettings) { Write-Output "Unable to get launchsettings.json." exit 1; }
foreach ($profileKey in $launchSettings.profiles.Keys)
{
	$profile = $launchSettings.profiles[$profileKey]

	if ($profile.ContainsKey("environmentVariables") -eq $false) { $profile.Add("environmentVariables", @{}) }
	if ($profile.environmentVariables.ContainsKey("AWS_ACCESS_KEY_ID") -eq $false) { $profile.environmentVariables.Add("AWS_ACCESS_KEY_ID", "") }
	if ($profile.environmentVariables.ContainsKey("AWS_SECRET_ACCESS_KEY") -eq $false) { $profile.environmentVariables.Add("AWS_SECRET_ACCESS_KEY", "") }
	if ($profile.environmentVariables.ContainsKey("AWS_SESSION_TOKEN") -eq $false) { $profile.environmentVariables.Add("AWS_SESSION_TOKEN", "") }
	
	$profile.environmentVariables.AWS_ACCESS_KEY_ID = $key
	$profile.environmentVariables.AWS_SECRET_ACCESS_KEY = $secret
	$profile.environmentVariables.AWS_SESSION_TOKEN = $token
}
$launchSettings | ConvertTo-Json -depth 32 | Set-Content $pwd\Properties\launchSettings.json
Write-Output "Successfully wrote credentials to launchsettings.json."

# Save the settings file
$settings.ClientId = $clientId
$settings.ClientSecret = $clientSecret
$settings.ClientExpiry = $clientExpiry
$settings.AccessToken = $accessToken
$settings.AccessTokenExpiry = $accessTokenExpiry
$settings | ConvertTo-Json -depth 32 | Set-Content $settingsFile

Write-Output ""
Write-Output "Ready! Your session token will expire on $tokenExpiry."
Write-Output ""
```