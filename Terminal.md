---
title: Terminal
lastmod: 2024-09-24T16:12:48-05:00
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
	
	if (-Not (Test-Path $Path)) {
		$Path = "."
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

# github copilot cli
function ?? { 
    $TmpFile = New-TemporaryFile
    github-copilot-cli what-the-shell ('use powershell to ' + $args) --shellout $TmpFile
    if ([System.IO.File]::Exists($TmpFile)) { 
        $TmpFileContents = Get-Content $TmpFile
            if ($TmpFileContents -ne $nill) {
            Invoke-Expression $TmpFileContents
            Remove-Item $TmpFile
        }
    }
}
 
function git? {
    $TmpFile = New-TemporaryFile
    github-copilot-cli git-assist $args --shellout $TmpFile
    if ([System.IO.File]::Exists($TmpFile)) {
        $TmpFileContents = Get-Content $TmpFile
            if ($TmpFileContents -ne $nill) {
            Invoke-Expression $TmpFileContents
            Remove-Item $TmpFile
        }
    }
}

function gh? {
    $TmpFile = New-TemporaryFile
    github-copilot-cli gh-assist $args --shellout $TmpFile
    if ([System.IO.File]::Exists($TmpFile)) {
        $TmpFileContents = Get-Content $TmpFile
            if ($TmpFileContents -ne $nill) {
            Invoke-Expression $TmpFileContents
            Remove-Item $TmpFile
        }
    }
}

New-Alias which Get-Command
New-Alias ll Get-ChildItem

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
function ggsup {
	& git branch --set-upstream-to=origin/$(git_current_branch)
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

# https://stackoverflow.com/questions/71198994/how-can-i-prevent-escape-sequences-being-added-to-powershell-output
$PSStyle.OutputRendering = 'Host'

clear
```
### GitHub Copilot CLI
Install the npm package [here](https://www.npmjs.com/package/@githubnext/github-copilot-cli).  The above functions were pulled from [here](https://www.hanselman.com/blog/github-copilot-for-cli-for-powershell).
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
