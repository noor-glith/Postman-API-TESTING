# Salesforce API Authentication & Account Update

## 1. Overview

This document describes the Salesforce API Postman collection used to
authenticate with Salesforce, retrieve API limits, and update an Account
record. The collection uses the OAuth 2.0 Client Credentials flow and
stores the returned access token in the Postman environment for
subsequent API requests.

## 2. Collection Information

  Property                Value
  ----------------------- ------------------------------------------------
  Collection Name         SalesForce's Api Auth
  Postman Collection ID   `ba5e9c71-b052-4e7f-afe9-41bd807b47d7`
  Schema                  Postman Collection v2.1.0
  Exporter ID             `36645247`
  Collection Link         Postman collection link provided in the source

## 3. Environment Variables

  ---------------------------------------------------------------------------
  Variable                Purpose                 Example / Source
  ----------------------- ----------------------- ---------------------------
  `baseUrl`               Salesforce              `{{baseUrl}}`
                          instance/base URL       

  `grantType`             OAuth grant type        `client_credentials`

  `client_id`             Salesforce OAuth client Configured in Postman
                          ID                      environment

  `client_secret`         Salesforce OAuth client Configured securely in
                          secret                  Postman environment

  `accessToken`           Access token returned   Automatically populated
                          by authentication       

  `apiVersion`            Salesforce API version  e.g. `vXX.X`

  `updateAcId`            Salesforce Account ID   e.g. `001XXXXXXXXXXXXXXX`
                          to update               

  `accountName`           Generated Account name  Automatically generated
                                                  before update
  ---------------------------------------------------------------------------

> **Security note:** `client_secret` should be stored as a secret/secure
> environment variable and should not be committed to source control or
> shared in plain text.

## 4. Request 1 --- Get Access Token

**Purpose:** Authenticate with Salesforce using the OAuth 2.0 Client
Credentials flow and save the returned access token as the Postman
environment variable `accessToken`.

-   **Method:** `POST`
-   **Endpoint:** `{{baseUrl}}/services/oauth2/token`
-   **Authentication:** No Auth
-   **Body type:** `x-www-form-urlencoded`

### Request Body

  Parameter         Value
  ----------------- ---------------------
  `grant_type`      `{{grantType}}`
  `client_id`       `{{client_id}}`
  `client_secret`   `{{client_secret}}`

### Postman Test Validation

The test script validates:

1.  HTTP response status is `200`.
2.  The response contains a non-empty `access_token`.
3.  `token_type` is `Bearer`.
4.  The access token is saved to the `accessToken` environment variable.

### Expected Response

``` json
{
  "access_token": "<token>",
  "token_type": "Bearer"
}
```

## 5. Request 2 --- Get Limits

**Purpose:** Retrieve Salesforce API usage and limit information for the
authenticated Salesforce organization.

-   **Method:** `GET`
-   **Endpoint:** `{{baseUrl}}/services/data/{{apiVersion}}/limits`
-   **Authentication:** Bearer Token --- `{{accessToken}}`

The request uses the collection-level Bearer authentication configured
with `{{accessToken}}`.

## 6. Request 3 --- Update Account

**Purpose:** Update the `Name` field of an existing Salesforce Account
record using the Account ID supplied through `updateAcId`.

-   **Method:** `PATCH`
-   **Endpoint:**
    `{{baseUrl}}/services/data/{{apiVersion}}/sobjects/Account/{{updateAcId}}`
-   **Authentication:** Bearer Token --- `{{accessToken}}`
-   **Content-Type:** `application/json`

### Request Body

``` json
{
  "Name": "{{accountName}}"
}
```

### Pre-request Script

Before sending the PATCH request, the collection generates a unique
Account name using the current timestamp and stores it in `accountName`.

``` javascript
const timestamp = Date.now();

const accountName = `Postman Test Account ${timestamp}`;

pm.environment.set("accountName", accountName);

console.log("Generated account name:", accountName);
```

### Post-request Validation

The test script validates:

-   HTTP response status is `204`.
-   The response body is empty.

## 7. Execution Flow

1.  Configure the Postman environment with:
    -   `baseUrl`
    -   `grantType`
    -   `client_id`
    -   `client_secret`
    -   `apiVersion`
    -   `updateAcId`
2.  Run **Get Access Token - Client Credentials Flow**.
3.  Validate that authentication succeeds and `accessToken` is
    populated.
4.  Run **Get Limits** to verify authenticated Salesforce API access and
    retrieve limits.
5.  Run **Update Account**.
6.  The pre-request script generates a unique `accountName`.
7.  Salesforce updates the Account identified by `updateAcId`.
8.  Validate HTTP `204` and an empty response body.

## 8. Validation Summary

  -------------------------------------------------------------------------
  Request                       Method      Expected Status Key Validation
  --------------- -------------------- -------------------- ---------------
  Get Access                      POST                `200` Access token
  Token                                                     returned; token
                                                            type = Bearer

  Get Limits                       GET                `2xx` Salesforce
                                                            limits are
                                                            returned

  Update Account                 PATCH                `204` No response
                                                            body
  -------------------------------------------------------------------------

## 9. Notes & Recommendations

-   Run the authentication request before requests that require the
    access token.
-   Keep client credentials in secure Postman environment variables.
-   Use a valid Salesforce API version in `apiVersion`.
-   Ensure `updateAcId` refers to an existing Account record and the
    OAuth client has sufficient permissions.
-   For automated performance testing, avoid logging access tokens in
    test output or reports.
-   Consider adding explicit validation for the Get Limits response if
    API limit thresholds are part of the test objective.

## 10. API Flow

``` text
Postman
   |
   | POST /services/oauth2/token
   v
Salesforce OAuth
   |
   | access_token
   v
Postman Environment
   |
   +----> GET /services/data/{apiVersion}/limits
   |
   +----> PATCH /services/data/{apiVersion}/sobjects/Account/{updateAcId}
```
