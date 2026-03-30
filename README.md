### Jamf Enrollment Primer

Jamf Enrollment Primer attempts to prevent Jamf Pro enrollment delay errors by priming `curl` connections to your Jamf Pro server.

by Kevin M. White

## Introduction

On some networks the Jamf Pro enrollment workflow can experience a significant delay due to an initial failure of the Jamf QuickAdd package to install the `jamf` binary. Fortunately, if the initial Jamf QuickAdd installation fails, Jamf Pro will automatically attempt the installation several minutes later. This second installation almost always succeeds but this introduces an unnecessary delay in Jamf Pro onboarding workflows.

The source of the issue is a TLS-related failure of a `curl` operation in the Jamf QuickAdd package. Although this failure only occurs on some networks, it's highly reproducible after initial startup of a new macOS system. Subsequent attempts of the same `curl` operation often succeeds with only the initial `curl` request failing.

__Excerpts from the install.log during enrollment on an affected system:__
```
...
2026-03-30 13:46:26-07 Mac-mini installd[674]: PackageKit: ----- Begin install -----
2026-03-30 13:46:26-07 Mac-mini installd[674]: PackageKit: request=PKInstallRequest <1 packages, destination=/, MDMManagedAppInstall=YES>
2026-03-30 13:46:26-07 Mac-mini installd[674]: PackageKit: packages=(
	    "PKLeopardPackage <id=com.jamfsoftware.osxenrollment, version=1.0, url=file:///var/folders/zz/zyxvpxvq6csfxvn_n0000044000011/C/com.apple.appstore/D1C4849E-8DCA-4DAA-9AE3-52A37BDF0EF0/quickadd.pkg#QuickAdd.pkg>"
	)
...
2026-03-30 13:46:27-07 Mac-mini package_script_service[1060]: postinstall: This is a lightweight package. Downloading jamf binary from https://macjutsu.jamfcloud.com/bin/jamf.gz to /Library/Application Support/JAMF/tmp on Mac OS X 26.4;
...
2026-03-30 13:46:57-07 Mac-mini package_script_service[1060]: postinstall: * SSL connection timeout
2026-03-30 13:46:57-07 Mac-mini package_script_service[1060]: postinstall: * Closing connection
2026-03-30 13:46:57-07 Mac-mini package_script_service[1060]: postinstall: Download failed with error code: 28
...
~ 5 minutes later
...
2026-03-30 13:51:29-07 Mac-mini installd[674]: PackageKit: ----- Begin install -----
2026-03-30 13:51:29-07 Mac-mini installd[674]: PackageKit: request=PKInstallRequest <1 packages, destination=/, MDMManagedAppInstall=YES>
2026-03-30 13:51:29-07 Mac-mini installd[674]: PackageKit: packages=(
	    "PKLeopardPackage <id=com.jamfsoftware.osxenrollment, version=1.0, url=file:///var/folders/zz/zyxvpxvq6csfxvn_n0000044000011/C/com.apple.appstore/AC84D2FA-6B04-402D-81A5-9399ED4E1B4B/quickadd.pkg#QuickAdd.pkg>"
	)
...
2026-03-30 13:51:29-07 Mac-mini package_script_service[1060]: postinstall: This is a lightweight package. Downloading jamf binary from https://macjutsu.jamfcloud.com/bin/jamf.gz to /Library/Application Support/JAMF/tmp on Mac OS X 26.4;
...
2026-03-30 13:51:30-07 Mac-mini package_script_service[1060]: postinstall: *  SSL certificate verify ok.
...
2026-03-30 13:51:30-07 Mac-mini package_script_service[1060]: postinstall: Download complete. Unzipping jamf binary...
...
```

## Solution

Given that this `curl` operation generally only fails the very first time, the Jamf Enrollment Primer can attempt a similar `curl` operation before the Jamf QuickAdd package is installed. This script is specifically designed to be deployed via a macOS installation package in a Jamf Pro (Automatic Device Enrollment) PreStage.

__Excerpts from the install.log during enrollment with Jamf Enrollment Primer:__
```
2026-03-30 14:24:16-07 Mac-mini installd[717]: PackageKit: ----- Begin install -----
2026-03-30 14:24:16-07 Mac-mini installd[717]: PackageKit: request=PKInstallRequest <1 packages, destination=/, MDMManagedAppInstall=YES>
2026-03-30 14:24:16-07 Mac-mini installd[717]: PackageKit: packages=(
	    "PKLeopardPackage <id=jamfenrollmentprimer, version=1, url=file:///var/folders/zz/zyxvpxvq6csfxvn_n0000044000011/C/com.apple.appstore/2993A961-DEF5-408E-B168-0EFCA8DFF9AA/JamfEnrollmentPrimer-KMW-SIGNED.pkg#payload.pkg>"
	)
...
2026-03-30 14:24:16-07 Mac-mini package_script_service[1081]: PackageKit: Executing script "postinstall" in /Library/InstallerSandboxes/.PKInstallSandboxManager/53249675-9CB7-45A4-9B90-95057C53A4D3.activeSandbox/Scripts/jamfenrollmentprimer.KWfgru
2026-03-30 14:24:16-07 Mac-mini package_script_service[1081]: ./postinstall: Mon Mar 30 14:24:16 Mac-mini postinstall[1083]: Status: Attempting initial connection to Jamf Pro service at macjutsu.jamfcloud.com...
2026-03-30 14:24:23-07 Mac-mini package_script_service[1081]: ./postinstall: Mon Mar 30 14:24:23 Mac-mini postinstall[1083]: Success: Initial connection to Jamf Pro took 2 attempt(s) and 7 second(s) to complete.
...
~ 6 seconds later
...
2026-03-30 14:24:26-07 Mac-mini installd[717]: PackageKit: ----- Begin install -----
2026-03-30 14:24:26-07 Mac-mini installd[717]: PackageKit: request=PKInstallRequest <1 packages, destination=/, MDMManagedAppInstall=YES>
2026-03-30 14:24:26-07 Mac-mini installd[717]: PackageKit: packages=(
	    "PKLeopardPackage <id=com.jamfsoftware.osxenrollment, version=1.0, url=file:///var/folders/zz/zyxvpxvq6csfxvn_n0000044000011/C/com.apple.appstore/FEC10E62-04ED-420F-9A23-20B120607EDD/quickadd.pkg#QuickAdd.pkg>"
	)
...
2026-03-30 14:24:26-07 Mac-mini package_script_service[1081]: postinstall: This is a lightweight package. Downloading jamf binary from https://macjutsu.jamfcloud.com/bin/jamf.gz to /Library/Application Support/JAMF/tmp on Mac OS X 26.4;
...
2026-03-30 14:24:26-07 Mac-mini package_script_service[1081]: postinstall: *  SSL certificate verify ok.
...
2026-03-30 14:24:26-07 Mac-mini package_script_service[1081]: postinstall: Download complete. Unzipping jamf binary...
...
```

## Deployment

The Jamf Enrollment Primer script can run by itself but it's designed to be deployed as a script in a macOS PreStage installation package.

**PreStage installation packages must be signed by an Apple Developer Account.**

[Pre-built macOS installation packages can be found on the Jamf Enrollment Primer releases page.](https://github.com/Macjutsu/jep/releases) One of the two provided macOS installation packages is signed by my personal (Kevin White) Apple Developer Account. However, organizations with their own Apple Developer Account would be better served by using their own signature.

