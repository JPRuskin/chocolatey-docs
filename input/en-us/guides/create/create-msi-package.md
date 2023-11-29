---
Order: 2
xref: howto-create-msi-package
Title: How to Create an MSI Installer Package
Description: Creating a package that installs an MSI
---

When starting out with Chocolatey, why not start simple? Installing an MSI is a popular use of a Chocolatey package, and we'll walk you through how to do just that - assuming you've already run through [Preparing Your Environment for Package Creation](#package-creation-env-placeholder).

### What is an MSI

MSI files are a standardised installer filetype, used by Windows. Unlike EXE installers (which can and will do just about anything), MSIs have _standards_ (or at least they all use the Windows Installer under the hood) and will behave somewhat predictably.

Consequently, they're often appreciated by busy Systems Administrators who want to deploy software to a large number of machines - or anyone looking for the silent install options.

### Using `Install-ChocolateyPackage`

Chocolatey CLI contains a selection of PowerShell functions known as "helper functions", which are available during all Chocolatey scripts.

Chocolatey supports a lot of different types of installer, including MSIs. EXEs, MSIs, and MSUs (all popular install formats) are normally installed using the [`Install-ChocolateyPackage`](#xref:install-chocolateypackage) function - and, in fact, our default template for a new package contains an example of this!

You can inspect the full set of available parameters for this function at the link above, or by opening the script file on your machine - it'll be contained in `%ChocolateyInstall%\helpers\functions`, which you should be able to open in Explorer or VSCode.

At the most basic level, though, you can use it by simply passing in a `PackageName` and `URL` (or `URL64`):

```powershell
$InstallArgs = @{
    PackageName = $env:ChocolateyPackageName
    Url         = "https://github.com/chocolatey/ChocolateyGUI/releases/download/2.1.0/ChocolateyGUI.msi"
}

Install-ChocolateyPackage @InstallArgs
```

> ::Note:: If you hadn't seen it before, `$env:ChocolateyPackageName` is a variable set within Chocolatey scripts to the name of the current package.

Of course, in a real-world package you would also want to add arguments to ensure the software installed silently, 64-bit install media (if available), and checksums to ensure that the URL you downloaded matched with your expectations (if you weren't going to host the installation media within the package).

We'll dive into the additional options in [Creating your Install Script](#creating-your-install-script), later on this page.

#### Creating an MSI Package

In this example, we're going to be creating a new Firefox MSI package.

> :info: For the purposes of the example, we'll provide URLs that work at time of writing. If you were creating a package from scratch, you would want to find the URLs for the installers yourself.
> In this case, we browsed to [Mozilla's download site](https://www.mozilla.org/en-GB/firefox/all/#product-desktop-release) and grabbed the URL of the MSI(s) we wanted.

* [Scenario 1 - Using VSCode](xref:howto-create-msi-package#scenario-one)
* [Scenario 2 - Using PowerShell](xref:howto-create-msi-package#scenario-two)
* [Scenario 3 - Manual Creation](xref:howto-create-msi-package#scenario-three)

<ul class="nav nav-tabs" role="tablist">
    <li class="nav-item">
        <a class="nav-link active" id="scenario-one-tab" data-bs-toggle="tab" href="#scenario-one" role="tab" aria-controls="scenario-one" aria-selected="true">VSCode</a>
    </li>
    <li class="nav-item">
        <a class="nav-link" id="scenario-two-tab" data-bs-toggle="tab" href="#scenario-two" role="tab" aria-controls="scenario-two" aria-selected="false">PowerShell</a>
    </li>
    <li class="nav-item">
        <a class="nav-link" id="scenario-three-tab" data-bs-toggle="tab" href="#scenario-three" role="tab" aria-controls="scenario-three" aria-selected="false">Manual</a>
    </li>
</ul>

::::{.tab-content .text-bg-theme-elevation-1 .p-3 .mb-3 .border-start .border-end .border-bottom .rounded-bottom}
:::{.tab-pane .fade .show .active #scenario-one role=tabpanel aria-labelledby=scenario-one-tab}

#### Using VSCode

There are [package templates](#package-template-placeholder) to create various package types, including an MSI template!

We'll use this to create our new MSI package:

1. Open your tutorials folder in VSCode.
1. Press `Ctrl + Shift + P` (or select `View > Command Palette`).
1. Select `Chocolatey: Install Template Package(s)`.
1. Open the `Command Palette` again, as in the prior step.
1. Select `Chocolatey: Create new Chocolatey package`
1. Give your package a name, e.g. `firefox-msi`, and hit enter.
1. Select `msi` from the template list.

This should result in a new directory, containing the following files:

```text
firefox-msi
├── tools
│   ├── chocolateyBeforeModify.ps1
│   ├── chocolateyInstall.ps1
│   ├── chocolateyUninstall.ps1
│   ├── LICENSE.txt
│   ├── VERIFICATION.txt
├── firefox-msi.nuspec
├── ReadMe.md
```

For this package, you can delete the `chocolateyBeforeModify.ps1`, `LICENSE.txt`, and `VERIFICATION.txt` files from the `tools` directory, and the `ReadMe.md` file from the root of `firefox-msi`.

:::
:::{.tab-pane .fade #scenario-two role=tabpanel aria-labelledby=scenario-two-tab}

#### Using PowerShell

There are [package templates](#package-template-placeholder) to create various package types, including an MSI template!

We'll use this to create our new MSI package:

```powershell
# Change location to your tutorial folder
Set-Location ~\tutorials

# Install the MSI template - this step will need to be run with elevation!
choco install msi.template --confirm --source Chocolatey

# Create a new package source directory using the template
choco new firefox-msi --template msi
```

This should result in a new directory, containing the following files:

```text
firefox-msi
├── tools
│   ├── chocolateyBeforeModify.ps1
│   ├── chocolateyInstall.ps1
│   ├── chocolateyUninstall.ps1
│   ├── LICENSE.txt
│   ├── VERIFICATION.txt
├── firefox-msi.nuspec
├── ReadMe.md
```

For this package, you can delete the `chocolateyBeforeModify.ps1`, `LICENSE.txt`, and `VERIFICATION.txt` files from the `tools` directory, and the `ReadMe.md` file from the root of `firefox-msi`.

:::
:::{.tab-pane .fade #scenario-three role=tabpanel aria-labelledby=scenario-three-tab}

#### Manual Creation of an MSI Package

1. Create a new directory to contain your package files. We normally name this the same as the package ID, e.g. `firefox-msi`.
    a. Create a `firefox-msi.nuspec` file within the new directory. Set the contents, as shown [below](#nuspec-content).
1. Create a `tools` directory within the `firefox-msi` directory.
    a. Create a `chocolateyInstall.ps1` file within the `tools` directory. Set the contents, as shown [below](#chocolatey-install-base-content).

##### Nuspec Content

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- Do not remove this test for UTF-8: if “Ω” doesn’t appear as greek uppercase omega letter enclosed in quotation marks, you should use an editor that supports UTF-8, not this one. -->
<package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
  <metadata>
    <id>firefox-msi</id>
    <version>$version$</version>
    <!-- packageSourceUrl>http://github.com/__REPLACE_YOUR_REPO__/msi-package</packageSourceUrl -->
    <owners>__REPLACE_YOUR_NAME__</owners>
    <title>Firefox (Install)</title>
    <authors>Mozilla Foundation</authors>
    <projectUrl>https://www.mozilla.org/en-GB/firefox/</projectUrl>
    <tags>firefox browser</tags>
    <summary>Mozilla Firefox is a web-browser.</summary>
    <!-- description>__REPLACE__MarkDown_Okay</description -->
  </metadata>
  <files>
    <file src="tools\**" target="tools" />
  </files>
</package>
```

##### Chocolatey Install Base Content

```PowerShell
$ErrorActionPreference = 'Stop'

$toolsDir   = Split-Path $MyInvocation.MyCommand.Definition -Parent

$packageArgs = @{
  packageName   = 'firefox-msi'
  fileType      = 'MSI'
  url           = '' # Download URL, HTTPS preferred
  url64bit      = '' # 64bit URL here (HTTPS preferred) or remove - if installer contains both, just use $URL

  checksum      = ''
  checksumType  = 'sha256' #default is md5, can also be sha1, sha256 or sha512
  checksum64    = ''
  checksumType64= 'sha256'

  silentArgs    = "/qn /norestart /l*v `"$($env:TEMP)\$($packageName).$($env:chocolateyPackageVersion).MsiInstall.log`""
  validExitCodes= @(0, 3010, 1641)
}

Install-ChocolateyPackage @packageArgs
```

You should now have a directory with a file structure that looks like this:

```text
firefox-msi
├── tools
│   ├── chocolateyInstall.ps1
├── firefox-msi.nuspec
```

:::
::::

You'll want to open the `firefox-msi.nuspec` file, and fill out the following fields in the `metadata` section if required:

* **packageSourceUrl**: This can be removed, or updated with your source URL.
* **owners**: This field shows who maintains the package - your name or handle is appropriate!
* **authors**: The authors of the software in question - for the example package, this should be removed or updated to  `Mozilla Foundation`.
* **projectUrl**: The URL of the software in question - for the example package, this should be removed or updated to `https://www.mozilla.org/en-GB/firefox/`.
* **tags**: Tags for the nuget feed you're publishing the package to. These can be updated however you prefer, though the Chocolatey Community Repository has some guidance on what is needed there.
* **summary**: This is a summary of the package contents. For this example, you could add `Mozilla Firefox is a web-browser.`
* **description**: This is a description of the package contents, and generally contains more detail than the summary - including things like package arguments. It supports markdown. We won't be dealing with anything that complex here!

You can now fill or remove any other commented out sections of the nuspec file, if you want.

#### Creating your Install Script

So, you already have a `chocolateyInstall.ps1` script present in your tools directory! However, it's been generated from a template - so will be missing a few details!

As mentioned above, we're going to be using [Mozilla Firefox](https://www.mozilla.org/en-GB/firefox/) for our example package. You can use the following values in the upcoming steps - or, find them yourself:

* **URL**: `https://download-installer.cdn.mozilla.net/pub/firefox/releases/120.0.1/win32/en-GB/Firefox%20Setup%20120.0.1.msi`
* **URL64bit**: `https://download-installer.cdn.mozilla.net/pub/firefox/releases/120.0.1/win64/en-GB/Firefox%20Setup%20120.0.1.msi`
* **Checksum**: `1D871A1364C2EF0371C172548E564102DC698E0FA48E3E72118E92C17F163C7A`
* **Checksum64**: `0FC6959385CB89C03E1C82F991EB8449BC8E2E080075A648A4C65CCDC5CD7D58`

If you open the `chocolateyInstall.ps1` from the newly created `tools` directory, you will see that there are comments to walk you through creating this package, and blank strings to be filled in! Most of these strings are found within the `$packageArgs` hashtable.

Update the following values in the install script:

* **URL**: You will need to provide a URL for the MSI to install.
* **URL64**: If there is a separate 64-bit MSI, you should provide a URL for that here.
* **Checksum**: To ensure you download the correct file, we support providing a checksum (and recommend you use it)
* **Checksum64**: Similar to the URL64, if there is a separate install provided, you will need to calculate a checksum for it and add it here.

To calculate the checksum yourself, download the file and run `Get-FileHash` in a PowerShell window:

```PowerShell
Invoke-WebRequest -Uri 'https://download-installer.cdn.mozilla.net/pub/firefox/releases/120.0.1/win32/en-GB/Firefox%20Setup%20120.0.1.msi' -OutFile '~\Downloads\Firefox Setup 120.0.1.msi' -UseBasicParsing
Get-FileHash '~\Downloads\Firefox Setup 120.0.1.msi'
```

This will output the SHA256 hash for the file. Repeat for each file you want to generate a hash for (or use the values provided above).

You can now remove all commented out sections from this file, and save it.

Your final file should look something like this:

```PowerShell
$ErrorActionPreference = 'Stop'

$packageArgs = @{
  packageName   = 'firefox-msi'
  fileType      = 'MSI'
  url           = 'https://download-installer.cdn.mozilla.net/pub/firefox/releases/120.0.1/win32/en-GB/Firefox%20Setup%20120.0.1.msi'
  url64bit      = 'https://download-installer.cdn.mozilla.net/pub/firefox/releases/120.0.1/win64/en-GB/Firefox%20Setup%20120.0.1.msi'

  checksum      = '1D871A1364C2EF0371C172548E564102DC698E0FA48E3E72118E92C17F163C7A'
  checksumType  = 'sha256'
  checksum64    = '0FC6959385CB89C03E1C82F991EB8449BC8E2E080075A648A4C65CCDC5CD7D58'
  checksumType64= 'sha256'

  silentArgs    = "/qn /norestart /l*v `"$($env:TEMP)\$($packageName).$($env:chocolateyPackageVersion).MsiInstall.log`""
  validExitCodes= @(0, 3010, 1641)
}

Install-ChocolateyPackage @packageArgs
```

You could inject additional configuration after the initial installation, if you wanted - or have this package simply install the software, and have a separate package for [applying configuration](#placeholder-config-package)!

#### Considering Uninstallation

Chocolatey will handle uninstallation of most software installed like this automatically. However, if you find that you have software that is not removed cleanly, you can provide a `chocolateyUninstall.ps1` script to run when the package is removed.

The logic present in the uninstall script provided by the template is a basic version of this.

This can also be used to clean up user data, or to roll-back configuration changes made - but that will be more useful in later package types.

In the case of the example package, Chocolatey handles it well - so you can remove the `tools\chocolateyUninstall.ps1` file if you want.

### Compiling Your Package

You can now use `choco pack` to compile your Chocolatey package into a nupkg file, ready for installation!

1. In VSCode, press `Ctrl + Shift + P` or use the `View` menu and click on `Command Palette`.
1. Select `Chocolatey: Package Chocolatey package(s)` from the prompt.
1. Select `firefox-msi.nuspec` from the prompt.
1. Press Enter, providing no additional input.

You should have a new package generated in your current working directory.

### Installing Your Package

You can now test installation of your package!

Personally, I like to test this in a Windows Sandbox environment, or the [Chocolatey Test Environment](https://github.com/chocolatey-community/chocolatey-test-environment) (particularly if you already have an installation of Firefox present) - but you can just use your machine, if you want. **Beware** that it may affect existing installations (as you're essentially testing in production).

Launch an elevated terminal on your test environment, either using PowerShell or `cmd`. In the new window, run something like the following:

```powershell
choco install firefox-msi --source='Tutorials' --confirm
```

The command should run, and Firefox should be installed! You should be able to find it in the Start Menu, in the install location in Program Files, and within Programs and Features.

### Uninstalling Your Package

You should be able to test the uninstall by running `choco uninstall` on your test environment:

```PowerShell
choco uninstall firefox-msi --confirm
```

The package should be uninstalled, along with the software. It should not be present in the Start Menu, the install location, or Programs and Features.

### Conclusion

At this point, you should have a working MSI package! Congratulations! Hopefully you can apply this to other MSIs!
