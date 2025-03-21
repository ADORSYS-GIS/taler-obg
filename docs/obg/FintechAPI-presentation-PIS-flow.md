# FintechAPI PIS flow Documentation

## 1. Overview
  
  This document provides details on how Fintech API is used in the Payment Initiation Flow
---

## 2. Understanding the Payment Initiation Service in XS2A

The Payment Initiation Service (PIS) in XS2A (Access to Account) allows third-party providers (TPPs) to initiate payments on behalf of a user directly from their bank account.  

### Key Components in XS2A Payment Initiation:

- **Fintech**:  
  Entity that provides an app (mobile/web) to the PSU for them to initiate the payment (note that in some cases, the Fintech can also be the TPP).
- **TPP (Third-Party Provider)**:  
  A licensed entity that interacts with the bank's API to initiate payments.
- **PSU (Payment Service User)**:  
  The end-user who initiates the payment via a fintech/TPP app.
- **ASPSP (Account Servicing Payment Service Provider)**:  
  The bank or financial institution holding the payer’s account.
- **SCA (Strong Customer Authentication)**:  
  A security mechanism required for user authentication.

---

## 3. API Description

 - **Overview**: This API provides Fintechs with the capability to perform payment initiation calls to the PSU's bank.To communicate with the OpenBankingGateway a fintech UI need to communicate with a fintechAPI on the fintechServer side that will forwards the request to the OPG API described [here]([https://github.com/adorsys/open-banking-gateway/blob/develop/opba-banking-rest-api-ymls/src/main/resources/static/tpp_banking_api_ais.yml](https://github.com/ADORSYS-GIS/taler-obg/blob/main/docs/obg/Open%20Banking%20Gateway%20-%20PIS%20Flow%20Documentation.md)).
 - **Consumers**: FintechUI   
Let see some of the endpoints related to the PIS flow that we are having :  

### a. FinTechAuthorization 

#### i. **POST v1/login**

Simple login interface used to establish a session between PSU and FinTech. Real application will delegate login to an oAuth2 Identity provider.

**- Request Header:**

- `X-Request-ID `: Unique ID that identifies this request through common workflow. Must be contained in HTTP Response as well..

**- Request Body:**

- `Login request`: Just a combination of the username and the password  
Example value  
```
{ "username":"string"
  "password": "string"
}
```

**- Response Header:**

- `X-Request-ID`: Unique ID that identifies this request.
- `Set-Cookie`: SessionCookie used to maintain the login session between the FinTechUI and the FinTechApi. This cookie is validated using an X-XSRF-TOKEN sent with the request header. This cookie must contain a hash reference to the corresponding XSRF-TOKEN -> hash(XSRF-TOKEN). So in order to verify that the request originate from the FinTechUI application, a server component will hash the given XSRF-TOKEN and compare it with the result in the cookie.
Remember that the XSRF-TOKEN value must be extracted from the request parameter named X-XSRF-TOKEN and not from a cookie header. This is important to mention as some implementation might return the XSRF-TOKEN to the FinTechUI in form of a cookie.  
- `X-XSRF-TOKEN`: XSRF parameter used to validate a SessionCookie. This token is produced by the FinTechAPI by either Storing a hash value of the XSRF-TOKEN in the SessionCookie or
Using the XSRF-TOKEN to encrypt the content of the SessionCookie. The XSRF-TOKEN is returned to the FinTechUI as a response header with the header value X-XSRF-TOKEN. It is the responsibility of the FinTechUI will take care of ensuring the persistence of the XSRF-TOKEN.  

**- Response body:**

- `200 code`: The user profile object. Contains information needed to present some sort of dashboard to the user.
  ```
    {
  "userProfile": {
    "name": "string",
    "lastLogin": "2025-03-21T09:18:56.963Z"
  }
  }
  ```
- `401 code`: Access credentials absent or invalid. This happens when

  - Provided credential for login not matching.
  - Endpoint expect a session cookie but none is present.
  - Session cookie is present but associated stateString does not match.  
As a consequence, we reset existing FinTechLoginSessionState if any. Nevertheless we do not reset RedirectSessionCookie.
```
{
  "sessionState": "string",
  "psuMessages": [
    {
      "category": "ERROR",
      "code": "string",
      "path": "string",
      "text": "string"
    }
  ]
}
```


#### ii. **GET v1/login/oauth2**

Oauth2 callback to authenticate user using some Oauth2 identity provider account. Provider id is set inside state.

**- Query Parameter:**

  - `code` (string): OAuth2 redirect code
  - `state` (string): OAuth2 redirect state
  - `scope` (string): OAuth2 requested scope
  - `error` (string): OAuth2 error

**- Response Header:**

- `X-Request-ID` (string): Unique ID that identifies this request through common workflow.
- `Set-Cookie` (string): SessionCookie used to maintain the login session between the FinTechUI and the FinTechApi. This cookie is validated using an X-XSRF-TOKEN sent with the request header. This cookie must contain a hash reference to the corresponding XSRF-TOKEN -> hash(XSRF-TOKEN). So in order to verify that the request originate from the FinTechUI application, a server component will hash the given XSRF-TOKEN and compare it with the result in the cookie.
Remember that the XSRF-TOKEN value must be extracted from the request parameter named X-XSRF-TOKEN and not from a cookie header. This is important to mention as some implementation might return the XSRF-TOKEN to the FinTechUI in form of a cookie.
- `X-XSRF-TOKEN`: XSRF parameter used to validate a SessionCookie. This token is produced by the FinTechAPI by either Storing a hash value of the XSRF-TOKEN in the SessionCookie or
Using the XSRF-TOKEN to encrypt the content of the SessionCookie. The XSRF-TOKEN is returned to the FinTechUI as a response header with the header value X-XSRF-TOKEN. It is the responsibility of the FinTechUI will take care of ensuring the persistence of the XSRF-TOKEN.
**- Response Body:**

- `200 code`: The user profile object. Contains information needed to present some sort of dashboard to the user.
  ```
    {
  "userProfile": {
    "name": "string",
    "lastLogin": "2025-03-21T09:18:56.963Z"
  }
  }
  ```
- `401 code`: Access credentials absent or invalid. This happens when

  - Provided credential for login not matching.
  - Endpoint expect a session cookie but none is present.
  - Session cookie is present but associated stateString does not match.  
As a consequence, we reset existing FinTechLoginSessionState if any. Nevertheless we do not reset RedirectSessionCookie.
```
{
  "sessionState": "string",
  "psuMessages": [
    {
      "category": "ERROR",
      "code": "string",
      "path": "string",
      "text": "string"
    }
  ]
}
```

#### iii. **POST v1/logout**

If user can be authenticated, user will be logged out.

**- Request header:**

  - `{X-Request-ID}` (string): Unique ID that identifies this request through common workflow. Must be contained in HTTP Response as well.
  - `X-XSRF-TOKEN` (string): XSRF parameter used to validate a SessionCookie or RedirectCookie.

**- Response Body:**

- `200 code`: Nothing returned.
- `401 code`: 
   ```
      {
    "sessionState": "string",
    "psuMessages": [
      {
        "category": "ERROR",
        "code": "string",
        "path": "string",
        "text": "string"
      }
    ]
  }
  ```
   
**- Response header:** (just in case of a 401 code response)
  - `X-Request-ID` (string): Unique ID that identifies this request through common workflow.
  - `Set-Cookie` (string): Delete all relevant cookies

#### iv. **GET v1/{auth-id}/fromConsent/{ok-or-notok}**

Entry point when PSU is redirected back from ConsentAuthorisationApi to the FinTechUI.

**- Path Parameter:**

  - `{redirectCode}` (string): Unique ID identifying the service session.
  - `{ok-or-notok}` (string): Available_values: OK, NOT_OK.

**- Query Parameter:**

  - `{auth-id}` (string): Unique ID identifying the service session.

**- Request Header:**

   - `X-XSRF-TOKEN` (string): XSRF parameter used to validate a SessionCookie or RedirectCookie.
   - `X-Request-ID` (string): Unique ID identifying the request.

**- Response header** (in case of a 202 response code)
    - `Location` (string): Entry point for loading the FinTechUI. This is generaly invoked when another application redirects an authenticated user to the FinTechUI. The FinTechUI after a reload must load the xsrfToken from the localStorage and invoke the /v1/{auth-id}/fromConsent/{ok-or-notok} endpoint of the FinTechAPI.
    - `X-Request-ID` (string): Unique ID that identifies this request through common workflow.
    - `X-XSRF-TOKEN` (string): XSRF parameter used to validate a SessionCookie. The XSRF-TOKEN is returned to the FinTechUI as a response header with the header value X-XSRF-TOKEN. It is the responsibility of the FinTechUI will take care of ensuring the persistence of the XSRF-TOKEN.
    - `Set-Cookie` (string): SessionCookie used to maintain the login session between the FinTechUI and the FinTechApi. This cookie is validated using an X-XSRF-TOKEN sent with the request header. This cookie must contain a hash reference to the corresponding XSRF-TOKEN -> hash(XSRF-TOKEN). So in order to verify that the request originate from the FinTechUI application, a server component will hash the given XSRF-TOKEN and compare it with the result in the cookie. Remember that the XSRF-TOKEN value must be extracted from the request parameter named X-XSRF-TOKEN and not from a cookie header. This is important to mention as some implementation might return the XSRF-TOKEN to the FinTechUI in form of a cookie.
    
**- Response header** (in case of a 401 response code)
    - `X-Request-ID` (string): Unique ID that identifies this request through common workflow.
    - `Set-Cookie` (string): Delete all relevant cookies. Example value: "RedirectSession=null; Path=/; Max-Age=0, Session=null; Path=/; Max-Age=0"
    
**- Response header** (in case of a 404 response)
   - `X-XSRF-TOKEN` (string): XSRF parameter used to validate a SessionCookie or RedirectCookie.
   - `X-Request-ID` (string): Unique ID identifying the request.
   - `Set-cookie` (string): describe above.

**- Response body** (in case of a 401 response code):Access credentials absent or invalid. This happens when

   - Provided credential for login not matching.
   - Endpoint expect a session cookie but none is present.
   - Session cookie is present but associated stateString does not match.
  As a consequence, we reset existing FinTechLoginSessionState if any. Nevertheless we do not reset RedirectSessionCookie.
  ```
{
  "sessionState": "string",
  "psuMessages": [
    {
      "category": "ERROR",
      "code": "string",
      "path": "string",
      "text": "string"
    }
  ]
}
  ```
    
**- Response body** (in case of a 404 response code): The specified resource was not found
  ```
{
  "sessionState": "string",
  "psuMessages": [
    {
      "category": "ERROR",
      "code": "string",
      "path": "string",
      "text": "string"
    }
  ]
}
  ```

#### iv. **GET v1/{auth-id}/fromPayment/{ok-or-notok}

Entry point when PSU is redirected back from ConsentAuthorisationApi to the FinTechUI. All the informations are the same as above.

### b. FintechBankSearch

#### i. GET /v1/search/bankSearch 

**- Query Parameter:**

   - `keyword` (string): The bank search input string
   - `start` (integer): The index of the first result
   - `max` (integer): The max number of entries to return with the response.
    
**- Request Header:**

   - `X-XSRF-TOKEN` (string): XSRF parameter used to validate a SessionCookie or RedirectCookie.
   - `X-Request-ID` (string): Unique ID identifying the request.

**- Response Header** (in case of a 202 response code)
   - `X-Request-ID`: same as above
   - `Set-Cookie`: same as above
   - `X-XSRF-TOKEN`: same as above
     
**- Response Header** (in case of a 401 response code)
   - `X-Request-ID`: same as above.
   - `Set-Cookie`: Delete all relevant cookies. Example: "RedirectSession=null; Path=/; Max-Age=0, Session=null; Path=/; Max-Age=0"

**- Response body** (in case of a 202 response code)
    ```
    {
  "bankDescriptor": [
    {
      "bankName": "string",
      "bic": "string",
      "bankCode": "string",
      "uuid": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "profiles": [
        {
          "bankId": "string",
          "bankName": "string",
          "name": "string",
          "bic": "string",
          "uuid": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "services": [
            "string"
          ],
          "externalId": "string",
          "externalInterfaces": "string",
          "protocolType": "string",
          "isSandbox": true
        }
      ]
    }
  ],
  "keyword": "string",
  "start": 0,
  "max": 0,
  "total": 0
}
    ```
**- Response body** (in case of a 401 response code) Access credentials absent or invalid. This happens when
      - Provided credential for login not matching.
      - Endpoint expect a session cookie but none is present.
      - Session cookie is present but associated stateString does not match.  
    As a consequence, we reset existing FinTechLoginSessionState if any. Nevertheless we do not reset RedirectSessionCookie.

    ```
           {
        "sessionState": "string",
        "psuMessages": [
          {
            "category": "ERROR",
            "code": "string",
            "path": "string",
            "text": "string"
          }
        ]
      }
    ```

#### i. GET /v1/search/bankSearch 

Request the profile of the bank identified with id (bankProfileId).

**- Request Header**
    - `X-Request-ID`: Unique ID that identifies this request through common workflow. Must be contained in HTTP Response as well.
    - `X-XSRF-TOKEN`: XSRF parameter used to validate a SessionCookie or RedirectCookie.

**- Query parameter** 
    - `bankProfileId`: Identifier of the bank to be loaded.
    
**- Response Header**
    - `X-Request-ID`: 
Continue here
### b. TPPBankSearchApi

This API provides FinTech possibilities to perform search calls to the PSU. Below are the endpoints available:

#### i. **GET v1/banking/search/bank-search**

Issues an incremental bank search request to the TppBankSearchApi.

**- Request Header:**

- `X-Request-ID `: Unique ID that identifies this request through common workflow. Shall be contained in HTTP Response as well.
- `onlyActive`: Controls visibility of inactive banks in the response (false - both active and inactive, true - only active)

#### ii. **GET v1/banking/search/bank-profile**

Issues an incremental bank search request to the TppBankSearchApi.

**- Request Header:**

- `X-Request-ID `: Unique ID that identifies this request through common workflow. Shall be contained in HTTP Response as well.
- `bankProfileId`: Identifier of the bank profile to be loaded.

### c. ConsentAuthorisationApi

Interface used by the PsuUserAgent to present consent authorization services to the PSU.The consent authorization process is triggered by redirecting the PSU from the TppBankingApi over the FinTechApi (On the fintech side) to the /consent/{auth-id} entry point of this ConsentAuthorisationApi. The decision on whether the authorization process is embedded or redirected is taken by this ConsentAuthorisationApi.

#### i. **GET v1/consent/{auth-id}**

This is the entry point for processing a consent redirected by the TppBankingApi to this ConsentAuthorisationApi.

At this entry point, the ConsentAuthorisationApi will use the xXsrfToken to retrieve the RedirectSession from the TppServer. An analysis of the RedirectSession will help decide if the ConsentAuthorisationApi will proceed with an embedded approach or a redirect approach .

**- Path Parameter:**

`{auth-id}` Used to distinguish between different consent authorization processes started by the same PSU. Also included in the corresponding cookie path to limit visibility of the consent cookie to the corresponding consent process.

**- Query Parameter:**

- `xXsrfToken `: XSRF parameter used to retrieve a redirect session. This is generaly transported as a query parameter.

#### ii. **POST v1/consent/{auth-id}/embedded**

Update consent session with PSU auth data whereby requesting remaining challenges for the ongoing authorization process. Returns 202 if one should proceed to some other link. Link to follow is in 'Location' header.

**- Path Parameter:**

`{auth-id}` Used to distinguish between different consent authorization processes started by the same PSU. Also included in the corresponding cookie path to limit visibility of the consent cookie to the corresponding consent process.

**- Query Parameter:**

- `xXsrfToken `: XSRF parameter used to retrieve a redirect session. This is generaly transported as a query parameter.

**- Request header**
- `X-Request-ID`: Unique ID that identifies this request through common workflow. Shall be contained in HTTP Response as well.

#### iii. **POST v1/consent/{auth-id}/deny**

Closes this session and redirects the PSU back to the FinTechApi or close the application window. In any case, the session of the user will be closed and cookies will be deleted with the response to this request.

**- Path Parameter:**

`{auth-id}` Used to distinguish between different consent authorization processes started by the same PSU. Also included in the corresponding cookie path to limit visibility of the consent cookie to the corresponding consent process.

**- Request header**
- `X-Request-ID`: Unique ID that identifies this request through common workflow. Shall be contained in HTTP Response as well.

#### iv. **GET v1/consent/{auth-id}/fromAspsp/{redirectState}/ok**

Redirecting back from ASPSP to ConsentAuthorisationApi after a successful consent authorization. In any case, the corresponding redirect session of the user will be closed and cookies will be deleted with the response to this request.

**- Path Parameter:**

- `{auth-id}` Used to distinguish between different consent authorization processes started by the same PSU. Also included in the corresponding cookie path to limit visibility of the consent cookie to the corresponding consent process.
- `{redirectState}` Code used to retrieve a redirect session. This is generaly transported as a path parameter due to some banks limitiations (ING ASPSP) instead of being transported as query parameter.

**- Query Parameter**
- `code`: Oauth2 code to exchange for token.

#### v. **GET v1/consent/{auth-id}/fromAspsp/{redirectState}/nok**

Redirecting back from ASPSP to TPP after a failed consent authorization. In any case, the corresponding redirect session of the user will be closed and cookies will be deleted with the response to this request.

**- Path Parameter:**

- `{auth-id}` Used to distinguish between different consent authorization processes started by the same PSU. Also included in the corresponding cookie path to limit visibility of the consent cookie to the corresponding consent process.
- `{redirectState}` Code used to retrieve a redirect session. This is generaly transported as a path parameter due to some banks limitiations (ING ASPSP) instead of being transported as query parameter.

---

## 4. Detailed API Documentation

For detailed explanation about our APIs you can check those files:
- **[TppBankingApi](https://github.com/adorsys/open-banking-gateway/blob/develop/opba-banking-rest-api-ymls/src/main/resources/static/tpp_banking_api_ais.yml)**
- **[TppBankingSearchApi](https://github.com/adorsys/open-banking-gateway/blob/develop/opba-banking-rest-api-ymls/src/main/resources/static/tpp_banking_api_bank_search.yml)**
- **[ConsentAuthorisationApi](https://github.com/adorsys/open-banking-gateway/blob/develop/opba-consent-rest-api/src/main/resources/static/tpp_consent_api.yml)**
- **[PaymentConfirmation](https://github.com/adorsys/open-banking-gateway/blob/develop/opba-banking-rest-api-ymls/src/main/resources/static/tpp_banking_api_token.yml)**

---

## 5. Authentication

Here we are using what we call SCA Approaches to ensure that the PSU is really the one that want to initiate the payment in our scenario. **SCA** stands for Strong Customer Authentication. In general we have 3 SCA approaches:
 - **Embedded approach**: When using the embedded approach, during the payment initiation transaction the PSU will his bank personal security credentials on the TPP/Fintech screen.
 - **Redirect approach**: When using this approach, the PSU will be redirect on the bank's side (e.g Online banking app) to enter his credentials and confirm the payment
 - **Decoupled approach**: Here the PSU will be sent to a separate secondary devince to authenticate during the payment initiation transaction

Those authentication mecanisms are managed inside the Online Banking Gateway project using the **[ConsentAuthorisationApi](#ConsentAuthorisationApi)**

---

## 6. Architecture and workflow

So in this section we will show a simplified architecture of the Open Banking Gateway and explain the workflow when it is come to Payment Initiation Service

![OBG architecture](img/fintech_OBG.png "Architecture").

In this picture all the green components represent a fintech application use by the PSU to initiate the payment.
In our scenario using our fintech-example when the PSU want to initiate a payment that is what is happening
- S1: On the fintech app the PSU will search and select his bank with a call that will go though the fintech API and reach the TPPBankSearchApi on the TPP side (our OBG API)
- 1a: The PSU will initiate a payment on the Fintech UI by providing all the informations we need to have (IBAN,instructed amount,etc...)
- 1b: The Fintech APi will carry informations gave by the PSU via the FintechAPI to the TPPBankingApi and initiate a payment via the ```/v1/banking/pis/payments/{payment-product}``` endpoint
- 1c: The OBG(on the TPP side) will go with the request to the ASPSP side
- 2a,2b,2c:  The consent authorization process is triggered by redirecting the PSU from the TppBankingApi (2a) over the FinTechApi (2b) to the ```/consent/{auth-id}``` entry point of this ConsentAuthorisationApi (2c).
- 4a,4b,4c: The final result of the authorization process is a PsuCosentSession that is returned by the token endpoint of the TppBankingAPi to the FinTechApi (4c). This handle will (PsuCosentSession) will be stored by the FinTechApi and added a PSU identifying information to each service request associated with this PSU.
