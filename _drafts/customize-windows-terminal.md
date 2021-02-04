---
layout: post
title: "Customizing the Windows Terminal"
comments: true
date: 2021-02-06
tags: [Terminal, Windows Terminal, Powershell, Scripting]
categories: [Article]
image:
  feature: terminal.png
---

I have been working on a set of scripts to help me set up a new (or recently formatted) machine. While most of the stuff is tailored to my personal use, having a configured and personalized terminal might be useful for others. My idea with this post is to show how you could quickly write up your own script to install and configure the Windows Terminal.

<!--more-->

# Step 1: Install Windows Terminal

First and foremost, we need the Windows terminal. You can get it from the store, but we want to have everything scriped, so let's use [winget](https://docs.microsoft.com/en-us/windows/package-manager/winget/) instead:

```bash
winget install terminal
```

# Step 2: Install dependencies

<!-- -->

```bash
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser -AllowPrerelease
Install-Module PSReadLine -Scope CurrentUser -AllowPrerelease -Force
```

(If `oh-my-posh` fails to install due to the `-AllowPrerelease` option, you'll need to update PowershellGet by running `Install-Module PowershellGet -Force`)

# Step 3: Configure `oh-my-posh`

Now that `oh-my-posh` is installed you can go ahead a picka theme, or make your custom one. (
There's a lot of stuff that can be done with `oh-my-posh`, all of it well documented in their [docs](https://ohmyposh.dev/).) For now I'm just going to set up the default theme.

```bash
Set-PoshPrompt -Theme jandedobbeleer
```

In order for the theme to be loaded everytime we open the terminal, we need to add the `Set-PoshPromt` command to our powershell profile. The following snippet will check if the profile exists and set the theme:

```bash
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
Add-Content $profile "Set-PoshPrompt -Theme jandedobbeleer"
```

# Step 4: Install fonts

Ok, at this stage everything looks nice and colourful but broken: some glyphs are missing. We need to update the font used in the terminal. The Windows Terminal comes with `Cascadia Code`, so if you check your fonts might already have `Cascadia Code PL` installed and might be able to just use that one. You may skip ahead to step 5, or stick around and install something fun off of [NerdFonts](https://www.nerdfonts.com/).

I've decided I want to use the [Caskaydia Cove Nerd Font](https://www.programmingfonts.org/#cascadia-code) (which is essentially and extended version of the original cascadia code font) but still want everything automated, so we need our script to be able to download, extract and install the font.

We start by figuring out where the fonts are hosted, and download and extract the file:

```bash
$DLPath = "https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/CascadiaCode.zip"
$DLFile = '.\terminal\cascadia.zip'

Invoke-WebRequest -Uri $DLPath -OutFile $DLFile
Expand-Archive -Path $DLFile -DestinationPath '.\terminal\cascadia\'
```

Next, we need to install the font files:

```bash
$FONTS = 0x14
$Path = ".\terminal\cascadia\"
$objShell = New-Object -ComObject Shell.Application
$objFolder = $objShell.Namespace($FONTS)
$Fontdir = Get-ChildItem $Path
foreach ($File in $Fontdir) {
    if (!($file.name -match "pfb$")) {
        $try = $true
        $installedFonts = @(Get-ChildItem c:\windows\fonts | Where-Object { $_.PSIsContainer -eq $false } | Select-Object basename)
        $name = $File.baseName

        foreach ($font in $installedFonts) {
            $font = $font -replace "_", ""
            $name = $name -replace "_", ""
            if ($font -match $name) {
                $try = $false
            }
        }
        if ($try) {
            $objFolder.CopyHere($File.fullname)
        }
    }
}
```

You should see a few popups letting you know the fonts are installed.

# Step 5: Updating the Terminal's `settings.json`

Now just open your terminal and add the font to the settings file.

```json
    ...
    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
            "fontSize": 10,
            "useAcrylic": true,
            "acrylicOpacity": 0.85,
            "fontFace": "CaskaydiaCove Nerd Font"
        },
        "list":
        [
            {
                // Make changes here to the powershell.exe profile.
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false
            },
    ...
```

In order to fully automate this, I back up the settings file and then just copy it over as a last step in the script.

```bash
Copy-Item ".\terminal\settings.json" -Destination "C:\Users\<USER>\AppData\Local\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json"  -Force
```

Keep in mind, however, that as new version of the Windows Terminal are released things might change in the settings file, so do check once in a while if it's worth backing up a fresh settings file.

Now you can just put all that together into one `*.ps1` file, and just use that to install and configure the terminal.

One caveat is that Windows doesn't typically let you run unsigned scripts. So you can just tell powershell to bypass that for this one script:

```bash
powershell.exe -ExecutionPolicy Bypass -File C:\my-script.ps1

```

A lot of other custopmizations can be done, but my final script currently looks something like [this](https://github.com/fmmendo/win-provision/blob/master/ohmyposh.ps1).

<!--
```bash
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force -ErrorAction Ignore
``` -->
