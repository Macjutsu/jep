### Jamf Enrollment Primer

Jamf Enrollment Primer attempts to prevent Jamf Pro enrollment delay errors by priming `curl` connections to your Jamf Pro server.

by Kevin M. White

## Introduction

On some networks the Jamf Pro enrollment workflow can experience a significant delay due to an initial failure of the Jamf QuickAdd package to install the `jamf` binary. Fortunately, if the initial Jamf QuickAdd installation fails, Jamf Pro will automatically attempt the installation several minutes later. This second installation almost always succeeds but this introduces an unnecessary delay in Jamf Pro onboarding workflows.

The source of the issue is a TLS-related failure of a `curl` operation in the Jamf QuickAdd package. Although this failure only occurs on some networks, it's highly reproducible after initial startup of a new macOS system. Subsequent attempts of the same `curl` operation often succeeds with only the initial `curl` request failing.

Given that this `curl` operation generally only fails the very first time, the Jamf Enrollment Primer can attempt a similar `curl` operation before the Jamf QuickAdd package is installed. This script is specifically designed to be deployed via a macOS installation package in a Jamf Pro (Automatic Device Enrollment) PreStage.

## Deployment

The Jamf Enrollment Primer script can run by itself but it's designed to be deployed as a script in a macOS PreStage installation package.

**PreStage installation packages must be signed by an Apple Developer Account.**

Pre-built macOS installation packages can be found on the Jamf Enrollment Primer releases page. One of the two provided macOS installation packages is signed by my personal (Kevin White) Apple Developer Account. However, organizations with their own Apple Developer Account would be better served by using their own signature.

