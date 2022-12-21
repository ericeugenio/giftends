# Choices

By Gil Dobrovinsky, Roos Lindeboom and Eric Eugenio.

# Table of contents

- [Introduction](#introduction)
- [Architecture](#architecture)
- [Microservices](#microservices)
    - [CDN](#cdn)
    - [FTP Server](#ftp-server)
    - [GraphQL Server](#graphql-server)
    - [WS Server](#ws-server)
    - [REST Server](#rest-server)
    - [Storage Server](#storage-server)

# Introduction

In this document we will explain the reasons of the implementation of the different services in our system and the architecture itself.

See [how_to](/docs/how_to.md) for further explanation on how the system is implemented.

# Architecture

## Why

We believe the most important choice in our system its the architecture as it models all the design. So far, all the mebmbers of our group, have been developing systems with monolithic architectures.

This type of architecture is great for begginings, requires almost no set up and configurations as everything is deployed in one place, thus is also cheaper. However as systems beggins to scale up, monolithic architecture flaws start to appear:

- Scaling becomes harder, remember, everything is hosted in one place. Imagine our system receives more clients in on sub-system than an other, we would have to scale up everything while only one part requires it, making the whole system more expensive. 

- Integrations becomes harder, probably libraries of one sub-systems become incompatible with other libraries of other sub-systems, etc.

- With new updates, the whole system need to be redeployed, not to consider that if the hosting device fails the whole system fails.

Coursing a Integration course, we decided it was our time to abandon monolithic architectures and begin microservices architecture, that solves most of the problems mentioned above. Furthermore, it makes it easire for each of use to work on a service.

After deciding the architecture to implement it is time to explain how the architecture was designed.

## How

Due to the simplicity of the system and considering our limited azure knowledge. The architecture we implemented is a simplified microservice architecture, consisiting of an API gateway forwarding requests to the different microservices that communicate with each other.

In order to only accept request through the API gateway, all of the microservices are inside a virtual network, which entry point is the API Gateway. Moreover microservices are configure to only accept request within the virtual network.

More on the overview of this architecture can be seen in [overview_of_the_system](/docs/overview_of_the_system.md).

# Microservices

After understanding why we decided to implement microserivces and how it was done, we will explain why each service was implemented as it is.

## CDN

CDN Exposure is quite straightforward in Azure, thus no important choices qere required during the implementation of this service.

## FTP Server

Due to limitations on our Azure Subscription we had to work with a Windows VM configured to host an FTP server using FileZilla Server. Therefore we had no big choices in this service. However due to the requirements of the FTP server (receiving a single file from time to time) the solution we had was mor than enough, as well as cheaper compared to other VMs. 

## GraphQL Server



## WS Server

> **TODO:** to be implemented...

## REST Server

For our REST Server we decided to use Firebase Authentication with node. We looked at Passport as alternative option but we came to the conclusion that our integration would be easier if both us and our partner group would benefit from a smoother integration process if we both used the same auth service. Thus, we came to the conclusion to use Firebase as it has pretty straightforward set up as mentioned in the [how_to](/docs/how_to.md).

## Storage Server

> **TODO:** to be implemented...
