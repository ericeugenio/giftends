# Reflection of Process

By Gil Dobrovinsky, Roos Lindeboom and Eric Eugenio.

# Table of contents (in chronological order)

- [Introduction](#introduction)
- [Exposure](#exposing)
    - [FTP Server](#ftp-server)
    - [GraphQL Server](#graphql-server)
        - [Basics](#basics)
        - [Subscriptions](#bonus-subscriptions)
    - [REST Server](#rest-server)
        - [Firebase](#firebase)
        - [Endpoints](#endpoints)
    - [CDN](#cdn)
    - [WS Server](#ws-server)
    - [Storage Server](#storage-server)
- [Integration](#integration)
# Introduction

In this document we will reflect on the process of building our system, discuss any issues we faced and how we fixed these issues

See [how_to](/how_to.md) and [choices](/choices.md) for further explanation on how the services are implemented and the reason of such implementation.


# FTP Server

We began our project by first exposing an FTP Server. We did this by creating a Virtual Machine on Microsoft Azure with a Filezilla FTP server. 

We then connected remotely to the VM using Microsoft Remote Desktop and configured the FTP's server properties. After this, we defined users and groups on the FTP server and assigned a home directory with proper permissions.

When testing the server with an FTP client, we ran into some timeout issues. 

The connections were working, however the when trying to send files over the FTP server the operation would time out and then complete successfully.

In order to fix the issue, we first configured the port range in the FTP server. Then we had to make sure these ports are open in the firewall of the Windows VM. Lastly, on the Azure network configuration of the VM, we had to configure port forwarding on the network lever of the vm.

# GraphQL Server

