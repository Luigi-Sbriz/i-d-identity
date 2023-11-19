---
title: "Identity Trust System"
category: std

docname: draft-sbriz-identity-trust-system-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: SEC
workgroup: Web Authorization Protocol
keyword:
 - identity
 - authentication
 - symmetric method
venue:
  group: Web Authorization Protocol
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: Luigi-Sbriz/i-d-identity
  latest: https://example.com/LATEST

author:
 -
    fullname: Luigi Sbriz
    organization: Cybersecurity & Privacy Senior Consultant
    email: your.email@example.com

normative:  

  RFC6749:  
  RFC5322:  
  RFC5321:  
  RFC5234:  

informative:  

  ITU1:
    target: "https://www.itu.int/dms_pub/itu-t/opb/tut/T-TUT-CCICT-2020-PDF-E.pdf"
    title: >
      QTR-RLB-IMEI - Reliability of International Mobile Station Equipment Identity (IMEI), Technical Report
    author:
      org: International Telecommunications Union
    date: 2020-07  
  ITU2:
    target: "https://www.itu.int/rec/T-REC-E.118"
    title: "E.118: The International Telecommunication Charge Card"
    author:
      org: International Telecommunications Union
    date: 2006-05  
  LS1:
    target: "https://www.isaca.org/resources/isaca-journal/issues/2022/volume-2/a-symmetrical-framework-for-the-exchange-of-identity-credentials-based-on-the-trust-paradigm-part-1"
    title: >
      A Symmetrical Framework for the Exchange of Identity Credentials Based on the Trust Paradigm,
      Part 1: Identity Trust Abstract Model, ISACA Journal, vol.2
    author:
      ins: L. Sbriz
      name: Luigi Sbriz
    date: 2022-04  
  LS2:
    target: "https://www.isaca.org/resources/isaca-journal/issues/2022/volume-2/a-symmetrical-framework-for-the-exchange-of-identity-credentials-based-on-the-trust-paradigm-part-2"
    title: >
      A Symmetrical Framework for the Exchange of Identity Credentials Based on the Trust Paradigm,
      Part 2: Identity Trust Service Implementation, ISACA Journal, vol.2
    author:
      ins: L. Sbriz
      name: Luigi Sbriz
    date: 2022-04  
  LS3:
    target: "https://www.isaca.org/resources/isaca-journal/issues/2023/volume-1/how-to-digitally-verify-human-identity"
    title: >
      How to Digitally Verify Human Identity: The Case of Voting, ISACA Journal, vol.1
    author:
      ins: L. Sbriz
      name: Luigi Sbriz
    date: 2023-01  
  LS4:
    target: "https://www.isaca.org/resources/isaca-journal/issues/2023/volume-6/modeling-an-identity-trust-system"
    title: >
      Modeling an Identity Trust System, ISACA Journal, vol.6
    author:
      ins: L. Sbriz
      name: Luigi Sbriz
    date: 2023-11

--- abstract

This document describes the identity trust system, which is an open identity authentication system that does not require federation of authentication domains. It is based on a symmetric authentication protocol and a specific infrastructure to ensure trust in the identity providers (IdPs). Regarding the main components:

1. Symmetric authentication protocol - Both entities MUST recognize each other and are authenticated by their identity provider according to a symmetric scheme. This protocol builds on and extends the OAuth Authorization Framework RFC6749.
2. Trustees' network - A special network, dedicated to creating a protected environment for exchanging authentication messages between IdPs, constitutes the infrastructure to avoid domain federation.
3. Custodian concept - IdPs are divided into two typologies to better protect personal data. A generic IdP (called trustee) for pure digital authentication and a specific IdP (called custodian) under the control of the country's authority with legal right to the individual's real data to ensure physical identity.

--- middle

# Introduction

The current model of access to Internet protected resources requires that the identity of the user, i.e. the ***resource owner***, be authenticated by the resource manager, i.e. the ***service provider***. Authentication can be indirect, this means that there is a trusted third party for both, called ***identity provider***, which performs the authentication process. The mechanism is defined by [RFC6749].

This mechanism is asymmetrical, only the resource owner needs to be recognized but not vice versa. Furthermore, the digital identity has value only within its own digital ecosystem (authentication domain or set of domains in a relationship of trust between each other). Changing digital ecosystem, a new user is needed for the resource owner.

It is possible to improve this process by requiring equal dignity in recognition, i.e. each entity must be recognized by the other. Consequently, the identity recognition process will be symmetrical and presents a great advantage. It is no longer necessary to define a trust between domains or create new users to be able to operate in an ecosystem different from your own.

It is necessary to modify the authentication protocol to implement symmetric recognition of the digital identity. Furthermore, a network infrastructure dedicated to identity providers is required to guarantee confidentiality and reliability in the exchange of messages between them. Additionally, dividing IdPs into two categories improves security. One category will be made up of those who are only authorized to recognize digital identity, while the other category is that of identity providers with the legal authority to also manage real identity.

## Use cases of both authentication schemes

Figure 1 shows a use case describing the components of the classic identity recognition method with asymmetry in the authentication process [RFC6749]. A SVG image is available [here](https://github.com/Luigi-Sbriz/identity/images/1_Asymmetric-depiction.svg). The scenario depicted represents a resource owner who needs to retrieve a resource from the service provider. The identity provider MUST verify the identity of the resource owner before accessing the resource server.

                      ┌───────────┐
                      │Identity   │
                      │           │
                      │Provider   │
                      └─────┬─────┘
                    Digital │ecosystem
                   .........│..............................
                   :  ┌─────┴─────┐                       :
                   :  │Authorizati│                       :
                   :  │           │                       :
                   :  │on Server  │                       :
                   :  └─────┬─────┘                       :
                   :        │                             :
                   :        │                             :
    ┌───────────┐  :  ┌─────┴─────┐        ┌───────────┐  :  ┌───────────┐
    │Resource   │  :  │User       │        │Relying    │  :  │Service    │
    │           ├─────┤           ├────────┤           ├─────┤           │
    │Owner      │  :  │Agent      │        │Party      │  :  │Provider   │
    └───────────┘  :  └───────────┘        └─────┬─────┘  :  └───────────┘
                   :                             │        :
                   :                             │        :
                   :                       ┌─────┴─────┐  :
                   :                       │Resource   │  :
                   :                       │           │  :
                   :                       │Server     │  :
                   :                       └───────────┘  :
                   :......................................:
    
         Figure 1: Abstract Authorization Flow - Asymmetrical Case

Figure 2 shows a use case describing the components necessary to enable the identity authentication process in a symmetrical way that can operate in different digital ecosystems. A SVG image is available [here](https://github.com/Luigi-Sbriz/identity/images/2_Symmetric-depiction.svg). The new scenario depicts two different ecosystems, one for the resource owner and the other for the service provider. This means there MUST be two different identity providers interacting with each other to ensure the authentication process.

                      ┌───────────┐        ┌───────────┐
                      │Identity   │        │Identity   │
                      │           │        │           │
                      │Provider C │        │Provider S │
                      └─────┬─────┘        └─────┬─────┘
                    Digital │ecosystem   Digital │ecosystem
                            │C                   │S
                   .........│.........  .........│.........
                   :  ┌─────┴─────┐  :  :  ┌─────┴─────┐  :
                   :  │Authorizati│  :  :  │Authorizati│  :
                   :  │           ╞════════╡           │  :
                   :  │on Server C│  :  :  │on Server S│  :
                   :  └─────┬─────┘  :  :  └─────┬─────┘  :
                   :        │        :  :        │        :
                   :        │        :  :        │        :
    ┌───────────┐  :  ┌─────┴─────┐  :  :  ┌─────┴─────┐  :  ┌───────────┐
    │Resource   │  :  │User       │  :  :  │Relying    │  :  │Service    │
    │           ├─────┤           ├────────┤           ├─────┤           │
    │Owner      │  :  │Agent      │  :  :  │Party      │  :  │Provider   │
    └───────────┘  :  └───────────┘  :  :  └─────┬─────┘  :  └───────────┘
                   :                 :  :        │        :
                   :                 :  :        │        :
                   :                 :  :  ┌─────┴─────┐  :
                   :                 :  :  │Resource   │  :
                   :                 :  :  │           │  :
                   :                 :  :  │Server     │  :
                   :                 :  :  └───────────┘  :
                   :.................:  :.................:
    
         Figure 2: Abstract Authorization Flow - Symmetrical Case

The two representations are very similar to each other but it is noted that the symmetric protocol introduces direct communication between the identity providers' authentication servers to allow the circular transit of authentication messages. Therefore, no trust is needed between domains. The idea was first exposed in some articles published on ISACA Journal (see [LS1], [LS2], [LS3], [LS4]) with some specific use cases and examples.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Some terms are used with a precise meaning.

- "***resource owner***": 
An entity capable of granting access to a protected resource. When the resource owner is a person, it is referred to as an end-user or an individual. This is sometimes abbreviated as "***RO***". 
- "***service provider***": 
An entity capable of managing access to a protected resource. It is generally a legal person. This is sometimes abbreviated as "***SP***". 
- "***identity provider***": 
An entity capable of managing and recognizing the identity of registered entities. The set of all entities registered by the identity provider is also known as the IdP's digital ecosystem. This is sometimes abbreviated as "***IdP***". 
- "***resource server***": 
The server hosting the protected resources, capable of accepting and responding to protected resource requests using access tokens. The resource server is often accessible via an API. This is sometimes abbreviated as "***RS***". 
- "***client***" or "***user agent***": 
An application making protected resource requests on behalf of the resource owner and with its authorization. The term "client" does not imply any particular implementation characteristics (e.g., whether the application executes on a server, a desktop, or other devices). 
- "***relying party***": 
An application making protected resource authorization on behalf of the service provider and also managing its identity. The "relying party" acts as the "client" but on service provider side. This is sometimes abbreviated as "***RP***". 
- "***authorization server***": 
The server issuing access tokens to the client after successfully authenticating the resource owner and obtaining authorization. This is sometimes abbreviated as "***AS***". 
- "***access token***": 
The concept is the same of the [RFC6749], a tiny piece of code that contains the necessary authentication data, issued by the authorization server. 
- "***identity token***" or "***ID token***": 
The structure is similar to access token but it is used as proof that the user has been authenticated. The ID token may have additional information about the user and, it is signed by the issuer with its private key. To verify the token, the issuer's public key is used.
- "***digital ecosystem***": 
Internet environment composed of all entities based on the same identity provider.

The detail of the information exchanged in the interactions between the authorization server and the requesting client, or between relying party and resource server, or the composition of tokens, is beyond the scope of this specification.

# Symmetric authentication protocol

The symmetric authentication flow is conceptually not too dissimilar from the classic one on a single ecosystem [RFC5234], except that the authentication is dual because the two flows reflect the same operations symmetrically. Both the **client** (*resource owner*) and the **server** (*service provider*) MUST authenticate their identity through their IdP. The details of each basic operation are the same as the corresponding individual ecosystem specification [RFC6749] and maintain alignment with it over time.

Considering two entities, a consumer and a resource provider, each must authenticate to the other's environment. The sequence will be:

***1.*** Entities exchange the access tokens received from their authentication server with each other.  
***2.*** Entities send the received token to their authentication server.  
***3.*** Authentication servers exchange access tokens with each other.  
***4.*** Authentication servers verify tokens with their original.  
***5.*** Authentication servers send the result to their own entity.  
***6.*** Entities are authenticated and can now exchange information.

Conceptually, in a client-server schema, the authentication process begins with the resource owner requesting access to the protected resource to the service provider. Both respond with their access tokens and request their IdP to validate the received token. The IdPs exchange tokens for validation and send the result to their entity. On success, access to the resource is allowed.

Figure 3 shows the abstract depiction of the symmetric authentication sequence. A SVG image is available [here](https://github.com/Luigi-Sbriz/identity/images/3_Symmetric-sequence-diagram.svg).

    ┌───────────┐                                            ┌───────────┐
    │Relying    ├-(1)--Request Authentication--------------->│Authorizati│
    │           │                                            │           │
    │Party      │<-(2)-Return Server Token-------------------┤on Server S│
    └───────────┘                                            └───────────┘
    ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
    │Resource   │      │User       │      │Authorizati│      │Relying    │
    │           │      │           │      │           │      │           │
    │Owner      │      │Agent      │      │on Server C│      │Party      │
    └─────┬─────┘      └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
          |      Request     |                  |                  |
          ├-(3)--Protected-->+-(4)--Send Access Request----------->|
          |      Resource    |                  |                  |
          |                  |<-(5)-Return Server Token------------┘
          |                  |                  |
          |                  ├(6)-Request Client|
          |                  |  Token & Send    |
          |                  |  Server Token--->|
          |                  |                  |
          |<-(7)-Request Credentials via UA-----┤
          |                  |                  |
          └-(8)--Present Credentials via UA---->|
                             |                  |
                             └<-(9)---Return    |
                                Client Token----┘
    ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
    │User       │      │Authorizati│      │Authorizati│      │Relying    │
    │           │      │           │      │           │      │           │
    │Agent      │      │on Server C│      │on Server S│      │Party      │
    └─────┬─────┘      └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
          |                  |                  |                  |
          ├-(10)--Request Protected Resource & Send Client Token-->|
          |                  |                  |                  |
          |                  |<-(12)---Send     |<-(11)---Send     |
          |                  |  Client Token----┤  Client Token----┤
          |                  |                  |                  |
          |                  ├-(13)---Return    |                  |
          |                  |  Server Token--->|                  |
          |                  |                  |                  |
          └<-(14)--Return of |                  └-(15)--Return of  |
             Authorization---┘                     Authorization-->┘
    ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
    │Resource   │      │User       │      │Relying    │      │Resource   │
    │           │      │           │      │           │      │           │
    │Owner      │      │Agent      │      │Party      │      │Server     │
    └─────┬─────┘      └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
          |                  |                  |                  |
          |                  |                  ├(16)Call Protected|
          |                  |                  | Resource-------->|
          |                  |                  |                  |
          └<-(19)-----Return |<-(18)-----Return |<-(17)-----Return |
            Protect.Resource-┘ Protect.Resource-┘ Protect.Resource-┘
    
    Figure 3: Symmetric Authentication Protocol - Sequence diagram

(1) - (2)	The ***RP*** requests authentication to its ***AS*** (marked "S" as server) and receives the server token (access token from the service provider's AS). The relying party must be provided with its own access token to resolve multiple requests.  
(3) - (5)	The ***RO*** requests access to the protected resource via the user agent. The ***UA*** activates the authentication process by requesting access to the ***RP***, which responds by providing the server token. The response may also include the server ID token.  
(6) - (9)	The ***UA*** requests the client token (access token from the client's AS) to its ***AS*** (marked "C" as client), sending also the server token received from ***RP***. The client's ***AS*** requests credentials from the ***RO*** and returns the client token to the ***UA***.  
(10) - (11)	The ***UA*** requests the protected resource from the ***RP*** by sending the client token. Then, relying party requests to service provider's AS to verify the client token.  
(12) - (15)	Both authentication servers must verify that the tokens received match the originals. Then, client's AS informs the ***UA*** of the outcome and the same is done by the service provider's AS to the ***RP***. The outcome sent to the relying party may also include the client ID token.  
(16) - (17)	The ***RP*** notifies the ***RS*** of the legitimate request of '***UA***. The ***RS*** returns the protected resource to ***RP***.  
(18) - (19) The ***RP*** sends the protected resource to ***UA***, which then presents it to the requester ***RO***.

***Notes regarding some steps:***  
(4) If the server token is not available at this time, sequence (1) - (2) will be executed between steps (4) and (5) to provide the server token. Additionally, this change may also be necessary for a periodic refresh of the server token or if the entities are both clients.  
(6) The client's authorization could be performed in advance and the client token stored securely by the user agent for handling multiple authentication requests. This means performing only the server token communication here, avoiding the following steps (7) - (9) because already done.

The verification of the authenticity of the tokens is carried out by the IdPs who exchange messages on a dedicated network to reduce the risk of intrusion. Security is strengthened by the presence of two interfaces for the exchange of tokens, one is for the party in trust and the other is for the opposing party. If one is compromised, the other interrupts the flow avoiding authorization. The trust placed in the mutual validation of messages avoids having to merge authentication domains, leaving great flexibility to the system as a whole.

Identity recognition information resides only with the trusted identity provider. This reduces the need to store too much personal information in Internet registrations. Furthermore, to easily identify which IdP holds the entity's authentication credentials, it can be easily extracted from the username structure if this is defined following the same technique used to compose an email address [RFC5322], that means an username, an @ sign, and a domain name.

# Identity Provider - Trustee Concepts

An IdP is a service provider that verifies the authenticity of a digital identity so that it is truly what it claims to be. The service provided is the guarantee of recognition of the identity of natural or legal persons in digital communication. For this role as an identity guarantor it can also be called an **ID trustee**. When processing personal data, the laws of the country to which the data subject belongs must be considered. It could also provide additional services (e-mail, voice mail, anonymous account,...) but always in full compliance with current legislation and only if they do not present a risk for the data subject.

The infrastructure underlying symmetric communication is a dedicated network to the exchange of authentication messages between IdPs. Ideally, each IdP always has two connectors, one to communicate with its trusted entity and the other to exchange messages with another IdP. With its own entity the mechanism is exactly the one defined by [RFC6749]. With other IdPs, a reserved channel is required for the exchange of tokens, which provides guarantees on the integrity of the messages and their origin. This channel SHOULD have low latency because it represents an additional step compared to the single ecosystem authentication scheme. The intended mechanism for sharing messages is that of a mail server [RFC5321].

The dedicated network for the identity providers is not technically necessary for the authentication protocol but is essential for security, to reduce the risk of fraud or identity theft and, to ensure trust in lawful behavior. There MUST also be an international control body for the IdPs and the IdP network, an authority with the task of governing the overall system, i.e. defining technical standards, or carrying out audits to ensure compliance with the rules, or acting to exclude nodes in case of violation of the rules.

An IdP MUST be a legal person subject to both the laws of the country where it is established and international certifications to protect data treatment and service provision in terms of security and quality.

# Identity Provider - Custodian Concepts

The identity provider must carry out a recognition and registration of the user's personal data before being able to guarantee its identity. The collection of data from legal entities, as they are public, is not particularly critical, while for the natural person the protection provided by the law in force for the holder of the digital identity must be enforced. Consequently, it is useful to divide the set of IdPs into two categories, in the first the set of IdPs (also called trustees) that only manage the operation of digital identities and in the second those that guarantee to the trustees that the identity of the user is real. This last category of IdPs, called custodians (IdCs), should operate under the responsibility of the legal authority that manages the real identity of the individual (i.e. who issues the identity card). IdCs are the only guarantors of the link between physical and digital identity, the only ones to issue a digital identity with legal value. IdCs do not provide guarantees on identities associated with legal entities, which are managed directly by IdPs.

Through the legal digital identity, each user can request the issuing of a new digital identity to the trusted IdP. The latter will request the IdC to confirm the authenticity of the identification data received from the user, that is, the user could send an ID token with the contact information of its IdC. The request will be handled entirely online and will not require any additional data from the data subject other than the identification information that it decides to provide. The new identity will be useful to satisfy the typical needs of Internet transactions by providing precise guarantees.

- The ***identity custodian*** is the guarantor of the existence of the natural person and has the ability to uniquely identify them but only following a formal request from the relevant authority.  
- The ***identity provider*** receives the identification data that the data subject has decided to provide and will match these to the digital identity.  
- The ***service provider*** will have the guarantee that the user is linked to a real person for security, contractual or legal reasons.  
- The ***data subject*** can provide personal information according to their need, also maintaining anonymity.  
- The ***public authority*** that manages the real data will be able to identify the individual with certainty in case of violations of the law (i.e. to protect the service provider).

### Issuing of a New Digital Identity

Figure 4 shows a use case describing the request of a new digital identity from an identity provider. A SVG image is available [here](https://github.com/Luigi-Sbriz/identity/images/4_New-identity-use-case.svg).

                      ┌───────────┐        ┌───────────┐
                      │Identity   │        │Identity   │
                      │           │        │           │
                      │Custodian  │        │Provider   │
                      └─────┬─────┘        └─────┬─────┘
                    Digital │ecosystem   Digital │ecosystem
                            │IdC                 │IdP
                   .........│.........  .........│.........
                   :  ┌─────┴─────┐  :  :  ┌─────┴─────┐  :
                   :  │Authorizat.│  :  :  │Authorizat.│  :
                   :  │           ╞════════╡           │  :
                   :  │Server IdC │  :  :  │Server IdP │  :
                   :  └─────┬─────┘  :  :  └─────┬─────┘  :
                   :        │        :  :        │        :
                   :        │        :  :        │        :
    ┌───────────┐  :  ┌─────┴─────┐  :  :  ┌─────┴─────┐  :
    │Data       │  :  │User       │  :  :  │Relying    │  :
    │           ├─────┤           ├────────┤           │  :
    │Subject    │  :  │Agent      │  :  :  │Party  IdP │  :
    └───────────┘  :  └───────────┘  :  :  └───────────┘  :
                   :.................:  :.................:
    
         Figure 4: Abstract of New Digital Identity Request

Figure 5 shows the abstract representation of the message exchange sequence to request a new digital identity. A SVG image is available [here](https://github.com/Luigi-Sbriz/identity/images/5_New-identity-sequence-diagram.svg).

    ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
    │Data       │      │User       │      │Authorizat.│      │Relying    │
    │           │      │           │      │           │      │           │
    │Subject    │      │Agent      │      │Server IdC │      │Party  IdP │
    └───────────┘      └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
          |                  |                  |                  |
          ├-(1)-Request New->+-(2)--Send New Identity Request----->|
          |     Identity     |                  |                  |
          |                  |<--(3)--Return IdP Token-------------┘
          |                  |                  |
          |                  ├(4)--Request IdC  |
          |                  |     Token & Send |
          |                  |     IdP Token--->|
          |                  |                  |
          |<-(5)-Request Credentials via UA-----┤
          |                  |                  |
          └-(6)--Present Credentials via UA---->|
                             |                  |
                             └<--(7)--Return    |
                                      IdC Token-┘
    ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
    │User       │      │Authorizat.│      │Authorizat.│      │Relying    │
    │           │      │           │      │           │      │           │
    │Agent      │      │Server IdC │      │Server IdP │      │Party  IdP │
    └─────┬─────┘      └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
          |                  |                  |                  |
          ├-(8)--Request New Identity & Send IdC Token------------>|
          |                  |                  |                  |
          |                  |<--(10)--Send IdC |<--(9)--Send IdC  |
          |                  |         Token----┤        Token-----┤
          |                  |                  |                  |
          |                  ├-(11)--Return IdP |                  |
          |                  |     Token & Send |                  |
          |                  |     ID Token---->|                  |
          |                  |                  |                  |
          └<-(12)---Return   |                  └-(13)---Return    |
                    ID Token-┘                     Client Token--->┘
    ┌───────────┐                ┌───────────┐               ┌───────────┐
    │Data       │                │User       │               │Relying    │
    │           │                │           │               │           │
    │Subject    │                │Agent      │               │Party  IdP │
    └─────┬─────┘                └─────┬─────┘               └─────┬─────┘
          |                            |                           |
          └<-(15)------Return ID Token-┘<-(14)-Return Client Token-┘
    
    Figure 5: New Digital Identity Request - Sequence diagram

(1) - (3)	The ***data subject*** requests the new digital identity via the user agent to the identity provider. The ***UA*** activates the authentication process by requesting the new identity to the ***RP***, which responds by providing the IdP token (access token from IdP's AS).  
(4) - (7)	The ***UA*** requests the IdC token (access token from the custodian's AS) to its ***AS***, sending also the IdP token received from ***RP***. The custodian's ***AS*** requests credentials from the ***data subject*** and returns the IdC token to the ***UA***.  
(8) - (9)	The ***UA*** requests the new digital identity from the ***RP*** by sending the IdC token. Then, relying party requests to identity provider's AS to verify the IdC token.  
(10) - (11)	Both authentication servers must verify that the tokens received match the originals. If required by data subject, the custodian's AS will send an additional ID token with personal data.  
(12) - (13)	Custodian's AS informs the ***UA*** of the outcome and, if required, sends also a copy of the ID token. The identity provider's AS sends to the ***RP*** the client token (related the new identity provided by IdP).  
(14) - (15) The ***RP*** sends the client token to ***UA***, which then informs the ***data subject*** of the outcome and, if required, sends a readable copy of the ID token to check the personal information shared.

***Notes regarding some steps:***  
(3) If the relying party does not have the IdP token available, this will be requested from the authentication server after step (2).  
(4) IdC token does not contains any real identity information as default. If requested, an ID token with a standard set of real information can be included during this step.

Any new digital identity with legal value is issued according to the rules defined by the relevant authority. It is presumable that there is also physical recognition of the data subject before the provision of credentials but no reference model is defined in this document.

# Sustainable Digital Identity Trust Schema

For the effectiveness of the identity trust system based on the paradigm of trust towards a third party recognizable as reliable, it is necessary to guarantee a transparent and verifiable mechanism. The objective is to achieve universal participation and it is only possible if trust in the system as a whole is demonstrable. For this reason it is necessary to establish founding principles that guide the rules to guarantee equal dignity and balance in all components of the system. The following nine principles are established:

1. The digital identity can be cancelled or deleted without impacting the physical identity.
2. The digital identity must be linkable to the physical identity in a verifiable manner.
3. Only the authority that legally manages the individual's physical identity can verify this link.
4. The authentication system must be flexible (i.e., able to adapt to technological evolutions or emerging needs).
5. The authentication system must be accessible to all (i.e., without discriminatory costs).
6. The authentication system must be secure (i.e., continuously aligned with security best practices).
7. The authentication system must be privacy-friendly (i.e., not requiring any personal information unless strictly necessary).
8. The authentication system must be resilient (i.e., with availability appropriate for needs and the ability to cope with adversity).
9. The authentication system technology must be open (i.e., able to evolve based on transparent shared standards and verifiable developments).

To guarantee the principles set out, the requirements of the authentication system MUST include the protection of personal data and the guarantee of anonymity for lawful purposes, that is:

1. Ensure mutual recognition to guarantee the identity of the provider to the consumer.
2. Ensure the ability to authenticate the digital identities of consumers and providers against their real-world identity without exposing real data

Naturally, the ability to validate the authenticity of the relationship between digital identity and physical identity lies only with the public authority responsible for managing the citizen's identity.

# Security Considerations

Integrity and resilience are the most critical parameters. Symmetric authentication should avoid the risk of man-in-the-middle attack because it should successfully attack both message flows at the same time. The trustee is a critical component and must be subjected to rigorous checks on compliance with the standards. Referring to the term identification, we mean at least three different types, the device, the digital user and the individual.

- The ***device*** is identified with technical methods suited to the various needs. For example, geolocalization using International Mobile Equipment Identity (IMEI) [ITU1] and Integrated Circuit Card ID (ICCID) [ITU2].
- The ***digital user*** is well managed by [RFC6749] but inside the digital ecosystem. To manage users of multiple domains, either the user registrations are duplicated for each domain involved, or the domains involved are joined in a trust relationship.
- The ***individual*** is well managed with classic physical methods (e.g. photo ID) but the link with the digital identity needs to be improved because the quality is not satisfactory. This topic is beyond the scope of this specification and it was explored for example in [LS3].

An in-depth defense system SHOULD consider all the components involved and in this case not just the pure digital authentication of the user. In this document only the digital user is treated but extensions applicable to mixed situations with multiple types are certainly welcome to improve the overall security profile.

## User registration

It is important to control the amount of data exchanged during the authentication process but also that the data required to issue a new digital identity are the only ones strictly necessary. To assign access credentials to a protected resource to a user, a process of recording the user's identification and contact data is necessary, as they are necessary for the authentication of the digital identity and for the attribution of access rights to the resource. Data provided or exchanged with IdPs MUST comply with the need-to-know principle. Three ways of recording are required.

### Registration with an identity custodian (IdC).
The IdC has the legal authority to retain all personal data essential to complete the process of recognizing the real individual. Consequently it also has full authority over digital identity data and registration is subject to the law of the individual's country of origin.

### Registration with an identity provider (IdP).
The IdP has to guarantee the authenticity of the digital identity and the collection of personal data SHOULD be limited to the sole purpose of this operation. For the registration of a natural person, the IdP requests confirmation from the IdC of the real identity of the applicant before issuing the digital identity. The process is completely online and does not require any physical recognition. For legal entities, the data is provided by the owner or a delegate.

### Registration with a service provider (SP).
The service provider SHOULD know only the data necessary to build the authorization roles to govern access to resources and nothing more. These are provided directly by the user or by an ID token, in addition to those received automatically from their IdP relating to digital identity.

# Conclusions

The identity management system, to operate in different digital ecosystems, should rely on a common authentication protocol that symmetrically performs the same operations in complete transparency, entrusting the decision on recognition to a trusted third party. The trust in the reliability of the recognition carried out by the identity guarantor (IdP or IdC) cannot be based on the technological component alone. It is therefore necessary to involve a supervisory authority of the overall system for technological aspects and the authorities of the data subject's country for data protection.

To improve trust between two entities, in this identification scheme, the objective is to provide three guarantees:

1. The mutual recognition between consumer and supplier.
2. The control and minimization of the personal information processed
3. In case of legal need, the ability to match the digital identities of consumer and supplier against their real-world identity.

Technically, due to the strong synergy obtained it is advisable to maintain alignment with [RFC6749] and the related specifications. 

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

This document was prepared using text editor with Markdown syntax (kramdown-rfc dialect).
