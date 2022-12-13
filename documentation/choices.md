# Choices

By Gil Dobrovinsky, Roos Madelief and Eric Eugenio.

# Table of contents

- [Introduction](#introduction)
- [Exposure](#exposing)
    - [CDN](#cdn)
    - [FTP Server](#ftp-server)
    - [GraphQL Server](#graphql-server)
    - [WS Server](#ws-server)
    - [REST Server](#rest-server)
    - [Storage Server](#storage-server)

# Introduction

In this document we will explain the reasons of the implementation of the different services in our system and the architecture itself.

See [how_to](/how_to.md) for further explanation on how the system is implemented.

# Microservices

# Services

## CDN

## FTP Server
Due to limitations on our Azure Subscription we decided to work with a VM on the **Azure Portal**. As the path is only used to send one file, we chose the cheapest VM option we could find in order to save on costs. We also chose Filezilla for our FTP server due to its efficiency, and due to our familiarity with it. 

## GraphQL Server

## WS Server

## REST Server
For our REST Server we decided to use Firebase Authentication with node. We looked at Passport as alternative option but we came to the conclusion that our integration would be easier if both us and our partner group would benefit from a smoother integration process if we both used the same auth service. Thus, we came to the conclusion to use Firebase as it has pretty straightforward set up as mentioned in the [how_to](how_to.md).

## Storage Server