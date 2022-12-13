# Reflection of Process

By Gil Dobrovinsky, Roos Lindeboom and Eric Eugenio.

# Table of contents (in chronological order)

- [Introduction](#introduction)
- [Expose](#expose)
    - [FTP Server](#ftp-server)
    - [GraphQL Server](#graphql-server)
    - [REST Server](#rest-server)
    - [CDN](#cdn)
    - [WS Server](#ws-server)
    - [Storage Server](#storage-server)
- [Integration](#integration)

# Introduction

In this document we will reflect on the process of building our system, discuss any issues we faced and how we fixed these issues or else we we plan to fix these issues.

See [how_to](/docs/how_to.md) and [choices](/docs/choices.md) for further explanation on how the services are implemented and the reason of such implementation.

# Expose

## FTP Server

Please see [how_to](/docs/how_to.md) for a step-by-step walkthrough on FTP Server configuration. Here, we will focus on issues and steps taken to fix these issues

When testing the server with an FTP client, we ran into some timeout issues. 

The connections were working, however the when trying to send files over the FTP server the operation would time out and then complete successfully.

In order to fix the issue, we first configured the port range in the FTP server. Then we had to make sure these ports are open in the firewall of the Windows VM. Lastly, on the Azure network configuration of the VM, we had to configure port forwarding on the network lever of the vm.

## GraphQL Server
As mentioned in the [how_to](/docs/how_to.md) we hosted the GraphQL server using **Azure App Service** via **Azure Extensions** in Visual Studio Code.

> **TODO:** to be implemented...




## REST Server
As mentioned in [choices](/docs/choices.md), we decided to use Firebase due to ease of integration with our group. Firebase had a very straight forward set up process. A simple read of the documentation as well as a small article helped us navigate through this path quite quickly.

## CDN
> **TODO:** to be implemented...


## Profile Picture Path

For the Profile Picture Path an idea we had was using Gravatar. By npm installing gravatar this allows us to use global profile pictures in the Gravatar database with the email used to signup to our website. For example, if a user has a profile picture on their Gmail account, Gravatar allows the user to automatically have their profile picture uploaded to our website.

## WS Server
> **TODO:** to be implemented...

## Storage Server
> **TODO:** to be implemented...

# Integration
> **TODO:** to be implemented...

