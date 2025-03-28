---
title: Azure Fluid Relay token contract
description: Better understand the JSON Web Token used in Azure Fluid Relay
ms.date: 10/05/2021
ms.topic: reference
ms.service: azure-fluid
---

# Azure Fluid Relay token contract

Requests sent to Azure Fluid Relay should contain a JWT in the authorization header. This token should be [signed by the tenant key](../concepts/authentication-authorization.md).

## Claims

JWTs (JSON Web Tokens) are split into three parts: 

- **Header** - Provides information about how to validate the token, including information about the type of token and how it was signed. 
- **Payload** - Contains all the important data about the user or app that is attempting to call your service. 
- **Signature** - Is the raw material used to validate the token. 

Each part is separated by a period (.) and separately Base64 encoded. 

## Header claims

| Claim | Format | Description  |
|-------|--------|--------------|
| alg   | string | The algorithm used to sign the token. For example, "HS256" |
| typ   | string | This value should always be “JWT.” |

## Payload claims

| Claim      | Format                   | Description |
|------------|--------------------------|-------------|
| documentId | string                   | Generated by Azure Fluid Relay (AFR) service. Identifies the document for which the token is being generated. |
| scope      | string[]                 | Identifies the permissions required by the client on the document or summary. For every scope, you can define the permissions you want to give to the client.  |
| tenantId   | string                   | Identifies the tenant. |
| user       | JSON                     | Identifies users of your application. It can be used by your application to identify your users by using the [Fluid Framework Audience](use-audience-in-fluid.md).<br>`{ id: <user_id>, name: <user_name>, additionalDetails: { email: <email>, date: <date>, }, }`  |
| iat        | number, a UNIX timestamp | "Issued At" indicates when the authentication for this token occurred. |
| exp        | number, a UNIX timestamp | The "exp" (expiration time) claim identifies the expiration time on or after which the JWT must not be accepted for processing. The token lifetime can't be more than 1 hour. |
| ver        | string                   | Indicates the version of the access token. Must be `1.0` . |
| jti        | string                   | *Optional.*  The "jti" (JWT ID) claim provides a unique identifier for the JWT. The identifier value MUST be assigned in a manner that ensures that there is a negligible probability that the same value will be accidentally assigned to a different data object. We encourage you to use this claim to avoid failures due to usage of same token for document creation.  |

## A sample Azure Fluid Relay token

```typescript
{ 
  "alg": "HS256",  
  "typ": "JWT" 
}.{ 
  "documentId": "746c4a6f-f778-4970-83cd-9e21bf88326c", 
  "scopes": [ "doc:read", "doc:write", "summary:write" ],   
  "iat": 1599098963,  
  "exp": 1599098963,  
  "tenantId": "AzureFluidTenantId",  
  "ver": "1.0",
  "jti": "d7cd6602-2179-11ec-9621-0242ac130002"
}.[Signature] 
```

## How can you generate an Azure Fluid Relay token? 

You can use the [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) npm package and sign your token using this method.

```typescript
export function getSignedToken(
    tenantId: string,
    documentId: string,
    tokenLifetime: number = 60 * 60,
    ver: string = "1.0") {
        jwt.sign(
            {
                documentId, 
                user: {
                    displayName: "displayName", 
                    id: "userId", 
                    name: "userName" 
                }, 
                scopes: ["doc:read", "doc:write", "summary:write"], 
                iat: Math.round((new Date()).getTime() / 1000), 
                exp: Math.round((new Date()).getTime() / 1000) + tokenLifetime, //set the expiry date based on your needs but max-limit is one hour.
                tenantId, 
                ver,
                jti: uuid(), 
            },
            "<tenant_key>");
    }
```
