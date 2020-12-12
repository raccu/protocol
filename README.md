# Raccu Protocol Specification

**Authors:** [Rato Kuzmanić](https://github.com/ratokuzmanic) and [Tomislav Radanović](https://github.com/TommyRadan)</br>
**Version:** 1.0.0</br>
**Date:** October 31, 2018

## Contents
1. [Introduction](#1-introduction)   
1.1. [Background](#11-background)   
1.2. [Goals](#12-goals)   
1.3. [Contributing](#13-contributing)   
1.4. [Disclaimer](#14-disclaimer)
2. [Definitions](#2-definitions)
3. [Register](#3-register)   
3.1. [Register via email](#31-register-via-email)   
4. [Authentication](#4-authentication)   
4.1. [Attestation structure](#41-attestation-structure)   
4.2. [Login based on the authentication flow](#42-login-based-on-the-authentication-flow)   
5. [Reset credentials](#5-reset-credentials)
6. [Delete account](#6-delete-account)
7. [Requirements](#7-requirements)  
7.1. [Verification codes](#71-verification-codes)   
7.2. [Challenge and tokens](#72-challenge-and-tokens)   
7.3. [Digital signatures](#73-digital-signatures)  
7.4. [Attestations](#74-attestations)  
7.5. [Infrastructure concerns](#75-infrastructure-concerns)     
8. [List of figures](#8-list-of-figures)   
9. [Abbreviations](#9-abbreviations)

## 1. Introduction
This document provides a formal specification of the Raccu protocol. All readers are assumed to have working knowledge in cryptography and computer networking. As this is a high-level overview, implementation details are omitted intentionally.

### 1.1. Background
Raccu is a centralized secure token service (STS) protocol for public-key cryptography authentication based on a smartphone biometry. The protocol is architecturally inspired by the likes of OAuth 2.0, OpenID, and W3C’s WebAuthn while it conceptually resembles FIDO Alliance’s Client-to-Authenticator Protocol (CTAP), namely FIDO UAF standards.

### 1.2. Goals
The core vision of Raccu is to provide a user with a uniform method of authentication possessing the following properties:
1. Smaller complexity and lesser cognitive load than in a traditional email and password authentication system.
2. Single registration process for authenticating to multiple web services and applications.
3. Authentication process for issuing an attestation is independent of the type of registration used to create an account.
4. Preserving data and origin integrity of the attestation through public key infrastructure (PKI) based chain of trust.
5. Requiring user’s consent via independent biometric authorization process before granting an attestation.

### 1.3. Contributing
Anyone can contribute to the protocol or to its specification. To learn more about contributing, please visit our official repository on GitHub located at https://www.github.com/raccu/protocol.

### 1.4. Disclaimer
Implementation of certain elements of this specification may require licenses under third party intellectual property rights, including without limitation, patent rights. The authors and any other contributors to this specification are not, and shall not be held, responsible in any manner for identifying or failing to identify any or all such third party intellectual property rights.

THIS SPECIFICATION IS PROVIDED “AS IS” AND WITHOUT ANY WARRANTY OF ANY KIND, INCLUDING, WITHOUT LIMITATION, ANY EXPRESS OR IMPLIED WARRANTY OF NON-INFRINGEMENT, MERCHANTABILITY OR FITNESS FOR A PARTICULAR PURPOSE.

## 2. Definitions
Throughout the specification a following list of terms is widely used. In this section their meaning and assumptions made on them are given in details.

**Email**: Email client that allows the user to gain read access rights to the inbox associated with the email address they provided. It is assumed that the user is the only person with this privilege.

**Auth**: Central authentication provider that serves as a secure token store
(STS) and conforms to the Raccu Auth Provider API Specification. Valid
and trusted certificate issued by a certificate authority (CA) as a part of PKI
for this entity is implied.

**Mobile**: An implementation of the mobile application that communicates
with the corresponding Auth instance conforming to the Raccu Mobile Ap-
plication Specification. Ability to authenticate the user via a fingerprint
scanner on the host smartphone is implied.

**Browser**: An arbitrary web browser in charge of rendering quick response
(QR) codes and dispatching appropriate messages as specified by the Raccu
Client Component Specification.

**Attestation**: Digitally signed claim in a form of a token that confirms
holder’s identity. Issued by Auth in a format defined in this specification.

**Server**: An end web service or application that consumes an attestation.
Authorization service within the Server is assumed. Valid and trusted cer-
tificate issued as a part of PKI for this entity is implied.

The keywords ”MUST”, ”MUST NOT”, ”REQUIRED”, ”SHALL”, ”SHALL NOT”, ”SHOULD”, ”SHOULD NOT”, ”RECOMMENDED”, ”NOT RECOMMENDED”, ”MAY”, and ”OPTIONAL” in this document are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals, as shown here.

## 3. Register
Registration is an act of sharing the proof of identity between the user and the Auth. Even though the current version of the protocol defines only email-based authentication, the protocol has been designed with extensibility in mind, meaning that other types of verification claims can be added, if needed.

### 3.1. Register via email
The flow of registering an account via email is shown in Figure 1 and successful registration flow is defined as follows:
1. A user MUST provide a valid email.
2. When Auth receives a valid registration request, email containing the
verification code MUST be dispatched to the Email and a token MUST
be returned to the Mobile.
3. Upon receiving the verification code and entering it into the Mobile,
the Mobile MUST provide a method to generate, store, and retrieve
public key and an interface for signing an arbitrary message with the
matching private key.
4. Mobile MUST authenticate the user locally using a built-in biometry.
Upon authenticating, the Mobile MUST finish an account creation by
submitting a token, email, generated public key, and a digital signature
of all the previous parameters, with a REQUIRED addition of the
verification code, to the Auth.

> FIGURE 1 PLACEHOLDER

If the registration via email fails for any reason related to the semantics of the protocol, an attempt to register MUST be discarded and all the related data should be deleted. All the following reasons are considered to be a semantic of the protocol:
- Email is already associated with another account.
- Token does not exist.
- Token is not associated with a given email.
- Verification code entered into Mobile does not match the expected verification code.
- The signature cannot be verified by a given public key.
- The signature is invalid.

## 4. Authentication
In this section, a flow for authenticating the user by Auth and its use in logging in at an independent endpoint is defined.

The authentication flow is given in Figure 2 and successful authentication is defined as follows:
1. Browser MUST fetch an authentication challenge from the Auth. It is REQUIRED for the challenge to be combined with the domain name and displayed as a QR code for the user to scan it with the Mobile.
2. When the user scans an aforementioned QR code with the Mobile, the Mobile MUST authenticate the user locally using a built-in biometry while disclosing the domain that requires the authentication. Upon authenticating, the Mobile will send the challenge, domain name, and user’s email alongside the signature of those parameters to the Auth.
3. Auth SHOULD notify the Browser that the authentication is successful and that the attestation is ready.
4. Browser MUST request the attestation from the Auth by providing a challenge that the Auth issued as a reference to the authentication session.

> FIGURE 2 PLACEHOLDER

If the authentication fails for any reason related to the authentication as described by the protocol, an attempt to authenticate MUST be discarded.

All the following reasons are considered to be an authentication error:
- An email is not associated with any account.
- Challenge does not exist.
- The signature cannot be verified by an associated public key.
- The signature is invalid.

### 4.1. Attestation structure
An attestation that Auth issues to the user upon authenticating has a structure defined in a Figure 3.

> FIGURE 3 PLACEHOLDER

Within the structure, a _version_ represents a version of the protocol under which an attestation was issued, while _iat_ stands for _issued at_ and represents a moment at which an attestation was issued. The _domain_ contains a domain name of the web service or application that requested the authentication, while _issuer_ contains a domain name of the Auth instance. When it comes to the user information, the _email_ field contains the user’s email and a _claim_ field signifies a type of registration process that the user underwent upon account creation. In the first version of the protocol, only email verification claim is available, but this field was added with future changes in mind. Lastly, the _signature_ field contains a digital signature of all the other fields, signed by the issuer with their private key that matches their certified public key.

### 4.2. Login based on the authentication flow
When a user tries to connect to the Server utilizing the Raccu protocol, a flow shown on the Figure 4 is RECOMMENDED.

> FIGURE 4 PLACEHOLDER

If the login is successful, the Server SHOULD communicate with the Browser in a manner of its implementation. Both authentication and authorization from that point forward to the session expiry, as defined by the Server, MUST be within the Server’s scope. The attestation used to initially authenticate the user MUST be deleted by both the Browser and the Server.

## 5. Reset credentials
In this section, a flow for resetting credentials in case a user loses their smartphone, they no longer have access to it, or they wish to reset credentials for any other reason is defined.

The reset credentials flow is given in Figure 5 and a successful flow is defined as follows:
1. Mobile MUST request a verification code from the Auth by submitting the email address of the user which is seeking a credentials reset.
2. Upon receiving a reset credentials request, the Auth MUST dispatch an email with the verification code to the Email and a token MUST be returned to the Mobile.
3. When the user enters verification code into the Mobile, the Mobile MUST provide a method to generate, store, and retrieve public key and an interface for signing an arbitrary message with the matching private key.
4. Mobile MUST authenticate the user locally using a built-in biometry. Upon authenticating, Mobile MUST send the token, user’s email, newly generated public key, and a digital signature of all the previous parameters, with a REQUIRED addition of the verification code, to the Auth to finish the credentials reset.

> FIGURE 5 PLACEHOLDER

If the credentials reset fails for any reason related to the protocol, an attempt to reset credentials MUST be discarded. All the following reasons are considered to be related to the protocol:

- An email is not associated with any account.
- Token does not exist.
- Token is not associated with a given email.
- Verification code entered into Mobile does not match the expected verification code.
- The signature cannot be verified by a given public key.
- The signature is invalid.

## 6. Delete account
In this section, a flow for deleting an account by the user is defined.

Delete account flow is given in Figure 6 and a successful flow is defined as follows:
1. Mobile MUST request a verification code from the Auth by submitting the email address of the user which is trying to delete the account and a token MUST be returned to the Mobile.
2. Upon receiving a request to delete an account, the Auth MUST dispatch an email with the verification code to the Email.
3. When the user enters verification code into the Mobile, the Mobile MUST authenticate the user locally using a built-in biometry. Upon authenticating, the Mobile MUST send the token, user’s email, and a signature of the token, user’s email, and verification code to the Auth to complete the process of deleting the account.

> FIGURE 6 PLACEHOLDER

If the account deletion is successful, the Mobile MUST delete any record of user’s email, public key, and private key. The Auth SHOULD delete any record of user’s email, public key, and private key.

If the account deletion fails for any reason related to the protocol, an attempt to delete an account MUST be discarded. All the following reasons are considered to be related to the protocol:
- An email is not associated with any account.
- Token does not exist.
- Token is not associated with a given email.
- Verification code entered into Mobile does not match the expected verification code.
- The signature cannot be verified by the associated public key.
- The signature is invalid.

## 7. Requirements
In this section, a list of requirements and arguments for their inclusion is given. Requirements are grouped into semantic units based on the part of the protocol that they refer to.

### 7.1. Verification codes
1. Verification code MUST be at least six characters long, contain a character space of at least ten characters, and be _short-lived_ while it SHALL NOT be predictable. It is left up to the implementer to define _short-lived_, but it is RECOMMENDED to decline verification codes issued over five minutes ago.<br/>
*Argument*: Verification code should posses a certain entropy and a level of randomness, just as well as a limited lifetime in order to reduce the likelihood of a successful brute-force attack and Man-in-the-Middle (MITM) attack.
2. Verification code MUST NOT be revealed to third parties or sent over the network, except when it’s sent to the Email by the Auth.<br/>
*Argument*: Verification code represents a shared secret between the entities and revealing it to other parties or unnecessarily sending it over a (potentially insecure) network poses a confidentiality breach risk.
3. Verification code MUST be reset after a several unsuccessful verification code entries.<br/>
*Argument*: This is to reduce a chance of a successful brute-force attack.

### 7.2. Challenges and tokens
4. A challenge that the Auth generates MUST rely on a cryptographically secure random number generator (CSRNG) and MUST NOT be predictable.<br/>
*Argument*: If the challenge can be predicted or guessed, an unauthorized party might fetch the attestation or perform an attack from MITM vector of attacks, for instance a relay attack.
5. A challenge SHALL NOT be reused.<br/>
*Argument*: If the challenge is reused, the system becomes susceptible to the replay attack. Due to that, the challenge should be viewed as a nonce.
6. A token MUST NOT be predictable, SHOULD have a limited lifetime 13 and MUST be a universally unique identifier (UUID).<br/>
*Argument*: This is to reduce the chances of a targeted denial of service (DOS) attack.

### 7.3. Digital signatures
7. Mobile MUST NOT expose private key directly.<br/>
*Argument*: Since the digital signature is the basis of authentication, protecting the confidentiality of the private key is of utmost importance.
8. Mobile MUST always require a biometric authentication to allow access to digital signing capability.<br/>
*Argument*: This is to avoid cross-site scripting (XSS) attacks or other illegal attempts to get the user’s digital signature without their explicit approval.
9. Mobile MUST store generated keys in a trusted execution environment (TEE).<br/>
*Argument*: This is to avoid a root attack or other system-level attacks that target storage memory or execution environment.

### 7.4. Attestations
10. An attestation MUST be checked for the time of issuing and old attestations MUST NOT be accepted. It is left up to the implementer to define old, but it is RECOMMENDED to decline an attestation that was issued over three minutes ago.<br/>
*Argument*: Since the attestation proves the holder’s identity, an attacker that obtains a valid attestation might pose as the user identified by the attestation. Given that the login flow is not dependent upon data entry, a three minute time-to-live (TTL) period is considerate enough to let the network exchanges happen.
11. An attestation MUST NOT be continuously reused after the login succeeds.<br/>
*Argument*: An attestation serves as an initial proof of identity and a continuous exchange of the attestation increases the chances of its hijacking. After the initial authentication, the Server should keep track of the user’s identity and their permissions within the system.

### 7.5. Infrastructure concerns
12. The communication between all entities MUST rely on a secure channel that provides confidentiality and verifies integrity. It is RECOMMENDED to use up-to-date version of Transport Level Security (TLS) protocol with a well-tested and computationally secure cipher suite.<br/>
*Argument*: Entities exchange user’s personal information and transmitting them over a public channel would pose a privacy concern, as well as create new vectors of attack, namely social engineering, phishing, and targeted brute-force attacks. Furthermore, the challenge is used to identify an authentication session, and later on an attestation is used to authenticate the user, leading to the possibility of hijacking should the channel remain public.

## 8. List of Figures
1. Register via email protocol flow
2. Authentication protocol flow
3. Structure of an attestation
4. Login protocol flow
5. Reset credentials protocol flow
6. Delete account protocol flow

## 9. Abbreviations
**API** Application Programming Interface<br/>
**CA** Certificate Authority<br/>
**CSRNG** Cryptographically Secure Random Number Generator<br/>
**CTAP** Client-to-Authenticator Protocol<br/>
**DOS** Denial of Service<br/>
**FIDO** Fast IDentity Online<br/>
**IAT** Issued At<br/>
**MITM** Man-in-the-Middle<br/>
**PKI** Public Key Infrastructure<br/>
**QR** Code Quick Response Code<br/>
**STS** Secure Token Service<br/>
**TEE** Trusted Execution Environment<br/>
**TLS** Transport Level Security<br/>
**TTL** Time to Live<br/>
**UAF** Universal Authentication Framework<br/>
**UUID** Universally Unique Identifier<br/>
**W3C** World Wide Web Consortium<br/>
**XSS** Cross-site Scripting<br/>