---
title: Terminal
lastmod: 2025-04-11T10:02:23-05:00
---
# Terminal
## Windows
### PowerShell
This is my __profile__ script for [PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows). If you're not familiar with PowerShell, the profile is like the `.bashrc`. My profile depends on four libraries:
* [posh-git](http://dahlbyk.github.io/posh-git/)
* [oh-my-posh](https://ohmyposh.dev/)
* [1Password CLI](https://1password.com/downloads/command-line/)
* [GitHub Copilot CLI](https://githubnext.com/projects/copilot-cli) (More info below)

I also plan on adding some  [VMware PowerCLI](https://www.powershellgallery.com/packages/VMware.PowerCLI) stuff in the future.
```powershell
[console]::InputEncoding = [console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
Import-Module posh-git
oh-my-posh init pwsh --config "~/Documents/PowerShell/theme.omp.json" | Invoke-Expression
op completion powershell | Out-String | Invoke-Expression

Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward

function e {
	param (
		$Path = "."
	)
	
	if ($PSBoundParameters.ContainsKey("Path") -And -Not (Test-Path $Path)) {
		Write-Error "Path '$Path' does not exist."
		return
	}
	
	$item = Get-Item $Path
	if (!$item.PSIsContainer) {
		& "C:/Program Files/Notepad++/notepad++.exe" $Path
		return
	}
	
	& explorer.exe $Path
}

function repos {
	& cd ~/source/repos
}

New-Alias which Get-Command
New-Alias ll Get-ChildItem
New-Alias edit e

# utility function to get name of current branch
function git_current_branch {
	& git branch --show-current
}

# go to root of repo
function grt {
	& cd $(git rev-parse --show-toplevel || echo .)
}

# stage all changes
function gaa {
	& git add --all
}

# set upstream branch to same as local
function gsup {
	& git push --set-upstream origin $(git_current_branch)
}

# interactive rebase
function grbi {
	& git rebase --interactive
}

# trash all local changes (keep local commits)
function grhh {
	& git reset --hard
}

# trash all local changes and commits (reset to what is remote)
function gtrash {
	& git reset --hard origin/$(git_current_branch)
}

function restart
{
	shutdown /r /t 1
}

$PSStyle.FileInfo.Directory = ""

# auto generated copilot aliases
. "C:\Users\rickgray\Documents\PowerShell\gh-copilot.ps1"

clear
```
### GitHub Copilot CLI
Github Copilot CLI has had a few iterations. Now it looks like [this](https://github.com/github/copilot-cli).  To get the clean syntax it used to have and strip out all the extra crap it likes to print, here's my PowerShell function. GPT-5 is significantly faster than claude, so I use that.
```powershell
function ?? {
	[CmdletBinding()]
	param (
		[Parameter(ValueFromRemainingArguments = $true, Position = 0)]
		[string[]]$Prompt
	)
	
	$argsToPass = @(
		'--model',
		'gpt-5',
		'--allow-all-tools'
	)

	if ($Prompt.Count -lt 1) {
		& copilot --model gpt-5 --allow-all-tools
		return
	}

	$Prompt = "I am a sysadmin and software engineer and I have forgotten this common command. Assume PowerShell unless specified otherwise. Produce a single line example command to do the following: $Prompt. Only a single line. It must be as simple and concise as possible. Do not include any explanations or additional text. Do not include any comments. The command must be the first line of output."

	$argsToPass += @(
		'-p',
		($Prompt -join ' ')
	)

	$out = & copilot @argsToPass `
		2>$null `  # Error
		3>$null `  # Warning
		4>$null `  # Verbose
		5>$null `  # Debug
		6>$null	   # Information

	if ($out) {
		Write-Host -NoNewline $out
	}
}
```
### Generate theme
This is the script I use to generate my oh-my-posh theme.
```powershell
$alignment = 'left'
$textColorPrimary = '#ffffff'
$textColorSecondary = '#ff0000'
$style = 'plain'
$type = 'prompt'

$username = @{
	'foreground' = $textColorPrimary
	'style' = $style
	'template' = "<$textColorSecondary>┏[</>{{ .UserName }}<$textColorSecondary>]</>"
	'type' = 'session'
}

$git = @{
	'foreground' = $textColorPrimary
	'style' = $style
	'template' = "<$textColorSecondary>[</>{{ .HEAD }}<$textColorSecondary>]</>"
	'type' = 'git'
}

$root = @{
	'foreground' = $textColorPrimary
	'style' = $style
	'template' = "<$textColorSecondary>[</>⚡<$textColorSecondary>]</>"
	'type' = 'root'
}

$exitCode = @{
	'foreground' = $textColorPrimary
	'style' = $style
	'template' = "<$textColorSecondary>[x</>{{ .Meaning }}<$textColorSecondary>]</>"
	'type' = 'exit'
}

$path = @{
	'foreground' = $textColorPrimary
	'style' = $style
	'template' = "<$textColorSecondary>┖[</>{{ .Path }}<$textColorSecondary>]></>"
	'type' = 'path'
	'properties' = @{
		'style' = 'full'
	}
}

$firstLine = @{
	'alignment' = $alignment
	'segments' = @(
		$username,
		$git,
		$root,
		$exitCode
	)
	'type' = $type
}

$secondLine = @{
	'alignment' = $alignment
	'segments' = @(
		$path
	)
	'type' = $type
	'newline' = $true
}

@{
	'$schema' = 'https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json'
	'blocks' = @(
		$firstLine,
		$secondLine
	)
	'final_space' = $true
	'version' = 2
} | ConvertTo-Json -Depth 5 | Out-File -FilePath .\theme.omp.json
```
## Linux
