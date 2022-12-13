# Overview of the System

By Gil Dobrovinsky, Roos Lindeboom and Eric Eugenio.

# Table of contents

- [Introduction](#introduction)
- [Architecture Diagram](#architecture-diagram)
- [Implemented by](#implemented-by)

# Introduction

In this document we will explain the architecture of our system and who developed each part.

See [how_to](/docs/how_to.md) and [choices](/docs/choices.md) for further explanation on how the services are implemented and the reason of such implementation.

> **Note:** integration part is a single App Service and an Azure Function with a timer trigger, thus we believe it is not really useful to develop a diragram for it.
>
> In this document we will only focus on the exposure part.

# Architecture Diagram

As explained in [choices](/docs/choices.md), We decided to develop our system using a microservices architecture, in this section we are going to exaplin from a simplified view:

![Giftends Architecture Diagram](/docs/assets/images/architecture.png)
*Giftends Architecture Diagram*

Of course, this a simple overview of the system but is gives a great understanding of how is organised. There are three main isolated entry points in the system:

- CDN
- FTP Server
- API Gateway

We will focus on explaining the API Gateway entry point as the other are quite self-explanatory. Following the microservies pattern, we place an API Gateway between our services and clients acting as a reverse proxy routing clients requests. It aslo performs authentication in the specified routes using the JWT token generated in the Auth service. Moreover, all the microservices are in a virtual network removing access to all external clients, in this way they are only accessible thorugh the API Gateway.

Other patterns from a microservie architecture we implemented is interservice communication. Each service is independent but still, they communicate with each other. Concretely, we have three interservice communications:

- auth-friends: whenever a user is created or an invite is accepted auth service will need to call a backend API exposed by friends service to update the products.db.

- friends-products: whenever a user adds a product to it's wishlist, data from products will need to be retrieves, again through an internal API.

- products-ftp: whenever the products.db file is received in the FTP server it has to be updated in the products service, this is done through an **Azure Logic App**.

Lastly, we will briefly explain each service:

## CDN

This service consists of a storage container and an **Azure CDN endpoint** to expose the container where the logo is stored.

## FTP Server

This service consists of an **Azure Windows VM** hosting an FTP server.

## Products Service

This service consists of an **Azure App Service** exposing both an http server and a ws server as explained in the [how_to](/docs/how_to.md) document. Data is retrieved from a sqlite file stored in the same app service and there is an extra redis cache layer to improve performance.

## Friends Service

Similarly to products service, this service consists of an **Azure App Service** hosting an http server and a ws server. Data is also stored in a sqlite file hosted in the same App Service.

## Auth Service

This service consists of a set of **Azure Functions** to expose several endpoints for authentication purposes.

# Implemented by

In this section we will point out who implemented each part of the system:

| Part        | Member      |
| ----------- | ----------- |
| CDN         | Roos        |
| FTP Server  | Gil         |
| Products    | Eric        |
| Friends     | Roos        |
| Auth        | GIl         |
| APIM        | Eric        |
