# Open Banking Gateway PIS Flow Documentation

## 1. Overview

This document provides details on how the Payment Initiation Service is managed inside the OBG application. The PIS flow involves three main endpoints: TppbankingApi, tppbanksearch, and ConsentAuthorizationApi.

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

The Open Banking Gateway provides APIs for Fintechs to use, allowing users to initiate payments. We have three main APIs described below:

### a. TPPBankingApi

This API provides Fintechs with the capability to perform payment initiation calls to the PSU's bank. Below are the endpoints available:

#### i. **POST v1/banking/pis/payments/{payment-product}**

This method is used to initiate a payment at the ASPSP.

**- Path Parameter:**

`{payment-product}` Available values:
  - `sepa-credit-transfers`
  - `instant-sepa-credit-transfers`
  - `target-2-payments`
  - `cross-border-credit-transfers`
  - `pain.001-sepa-credit-transfers`
  - `pain.001-instant-sepa-credit-transfers`
  - `pain.001-target-2-payments`
  - `pain.001-cross-border-credit-transfers`

**- Request Header:**

- `Service-Session-Password`: Password to encrypt Fintech data. Deprecated, use `Fintech-Data-Password` instead.
- `Fintech-User-ID`: An End-User identifier, unique per physical person.
- `Fintech-Redirect-URL-OK`: URL where the PSU will be redirected after authorization.

**- Response Header:**

- `X-Request-ID`: Unique ID that identifies this request.
- `Service-Session-ID`: Unique ID identifying the service session.
- `Authorization-Session-ID`: Unique ID for the authorization session.
- `Redirect-Code`: Unique ID acting as a password protecting consent specification.
- `Location`: URL where the User must be redirected to perform the next step.

#### ii. **GET v1/banking/pis/payments/{payment-product}**

Returns the content of a payment object.

**- Path Parameter:**

Same `{payment-product}` values as in the `POST` endpoint above.

**- Request Header:**

- `Service-Session-Password`: Password to encrypt Fintech data.

**- Response Header:**

- `X-Request-ID`: Unique ID identifying the request.
- `Service-Session-ID`: Unique ID identifying the service session.

#### iii. **GET v1/banking/pis/payments/{payment-product}/status**

Checks the transaction status of a payment initiation.

**- Path Parameter:**

Same `{payment-product}` values as above.

**- Request Parameter:**

- `Service-Session-Password`: Password to encrypt Fintech data.
- `X-Request-ID`: Unique ID identifying the request.

**- Response Header:**

- `X-Request-ID`: Unique ID identifying the request.
- `Service-Session-ID`: Unique ID identifying the service session.

#### iv. **GET v1/banking/pis/{Service-Session-ID}/status**

Retrieves the session status, including external status providers if necessary. Used to understand the current state of the entire payment (e.g., completed, not started, error, etc.).

**- Path Parameter:**

- `{Service-Session-ID}`: Unique ID identifying the service session.

**- Request Header:**

- `Service-Session-Password`: Password to encrypt Fintech data.
- `X-Request-ID`: Unique ID identifying the request.

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
