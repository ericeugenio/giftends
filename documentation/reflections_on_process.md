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

In this document we will reflect on the process of building our system, discuss any issues we faced and how we fixed these issues

See [how_to](how_to.md) and [choices](choices.md) for further explanation on how the services are implemented and the reason of such implementation.

# Expose

## FTP Server

We began our project by first exposing an FTP Server. We did this by creating a Virtual Machine on Microsoft Azure with a Filezilla FTP server. 

We then connected remotely to the VM using Microsoft Remote Desktop and configured the FTP's server properties. After this, we defined users and groups on the FTP server and assigned a home directory with proper permissions.

When testing the server with an FTP client, we ran into some timeout issues. 

The connections were working, however the when trying to send files over the FTP server the operation would time out and then complete successfully.

In order to fix the issue, we first configured the port range in the FTP server. Then we had to make sure these ports are open in the firewall of the Windows VM. Lastly, on the Azure network configuration of the VM, we had to configure port forwarding on the network lever of the vm.

## GraphQL Server
As mentioned in the [how_to](how_to.md) we hosted the GraphQL server using **Azure App Service** via **Azure Extensions** in Visual Studio Code.

## REST Server
For our REST Server we decided to use Firebase Authentication with node. We looked at Passport as alternative option but we came to the conclusion that our integration would be easier if both us and our partner group would benefit from a smoother integration process if we both used the same auth service. Thus, we came to the conclusion to use Firebase as it has pretty straightforward set up as mentioned in the [how_to](how_to.md).

## CDN

## WS Server

## Storage Server


# Integration

