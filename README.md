# API Management (APIM) B2C Integration PoC
This document will describe steps to reproduce a PoC who integrates APIM with Azure B2C using an external IdP (Ping in this case). 
This Proof of Concept includes:
- IdP integration to B2C
- Developer accounts (ping) logging in to the API Dev Portal
- Creation of an App Registration at B2C tenant
- Use bearer token for validation at the API
- Integration on Dev Portal to use the token automatically

# Steps Overview
- [API Management (APIM) B2C Integration PoC](#api-management-apim-b2c-integration-poc)
- [Steps Overview](#steps-overview)
  - [3rd Party Identity (Ping Identity)](#3rd-party-identity-ping-identity)
  - [Add identity to B2C tenant](#add-identity-to-b2c-tenant)
  - [B2C User flow](#b2c-user-flow)
  - [B2C Applications](#b2c-applications)
    - [APIM Dev Portal v2](#apim-dev-portal-v2)
    - [ClientApp](#clientapp)
    - [Permissions to clientApp](#permissions-to-clientapp)
    - [Validation](#validation)
  - [APIM - Developer Identities](#apim---developer-identities)
  - [APIM APIs - Validate Token](#apim-apis---validate-token)
  - [APIM - OAuth Integration](#apim---oauth-integration)
  - [Results](#results)

## 3rd Party Identity (Ping Identity)

Created and application at one instance of Ping for Enterprise.
This is an OIDC Application with the following parameters:

|![](img/ping1.png)
|-

|![](img/ping2.png)|
|-

> We should capture the App Id and well-known openid endpoint for future configurations


## Add identity to B2C tenant

We incorporate Ping as a new identity provider with the following settings

![](img/b2c.png)

> For this PoC we defined mappings with sub, but it will depend on the claims you are receiving from the IdP

## B2C User flow

We need to create a user flow. In this case we created a flow from a SignIn_SignUp template, with the following specifications:

![](img/b2c1.png)

We need to make sure User's Object ID is included:

![](img/b2c2.png)

## B2C Applications

For this integration, we need to create 2 applications on B2C. In this case:
- **APIM Dev Portal v2**: Represents the APIM Portal
- **ClientAPP**: Represents an external application which consume APIs

### APIM Dev Portal v2
![](img/AppA1.png)

![](img/AppA2.png)

We need to create a secret:

![](img/AppA3.png)

>Redirect URIs will be changed later

### ClientApp
![](img/AppB1.png)

![](img/AppB2.png)

We should create an URI for this app (used on following steps):

![](img/AppB3.png)

We create a user_impersonation scope:

![](img/AppB4.png)

### Permissions to clientApp

Now we are adding *ClientApp* permissions to *APIM Dev Portal *app

![](img/AppC1.png)

![](img/AppC2.png)

>Admin consent can be required


### Validation

After the creation of the applications, we should be able to test the user flow:

![](img/UserFlow.png)

3rd Party logging page:

![](img/UserFlow1.png)

After logging in, we should be able to see the resultant token

![](img/UserFlow2.png)

You must keep these values for the following steps

well-known URL:

![](img/UserFlow3.png)

Once you click in that URL, collect **issuer** value

![](img/UserFlow4.png)

## APIM - Developer Identities

Now we will configure identities, to accept developers using their 3rd party credentials (through B2C):

![](img/apim1.png)
Notes:
- Client ID and Secret: Corresponds to *APIM Dev Portal v2*
- Signin and Authority: use your own B2C name
- Redirect URL: configure this URL as reply URL at the *APIM Dev Portal v2*

At this point, you should be able to log in to the dev portal with your external identities (Ping in this case).

## APIM APIs - Validate Token

![](img/apim2.png)

````xml
<policies>
    <inbound>
        <base />
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401">
            <openid-config url="https://<yourtenant>.b2clogin.com/customersorg.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=<policyname>" />
            <audiences>
                <audience>ClientApp - Application ID</audience>
            </audiences>
            <issuers>
                <issuer>issuer url</issuer>
            </issuers>
        </validate-jwt>
        <cors>
            <allowed-origins>
                <origin>*</origin>
            </allowed-origins>
            <allowed-methods>
                <method>GET</method>
                <method>POST</method>
            </allowed-methods>
        </cors>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
````

- Use values collected previously for **openid-config url** and **issuer url**
- For **audience**, use the Application ID of **ClientApp**

## APIM - OAuth Integration

Now we are enabling the **try out** option from dev portal, to use an OAuth token (Bearer)

![](img/apim3.png)

![](img/apim4.png)

After that, we assign OAuth config we created to the APIs we want:

![](img/apim5.png)

## Results

After all this configuration changes, now we should be able to:
- Log in to the Dev Portal with our 3rd party identity
- Access to authenticated APIs (token validation)
- Inject bearer token from the dev portal for testing purposes

Example - Without token:
![](img/test1.png)

![](img/test2.png)

With token (authorization header automatically injected):
![](img/test3.png)

![](img/test4.png)
