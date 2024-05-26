---
title: "OAuth Client ID Metadata Document"
abbrev: "Client ID Document"
category: info

docname: draft-parecki-oauth-client-id-metadata-document-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
  - oauth
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  github: "aaronpk/draft-parecki-oauth-client-id-metadata-document"
  latest: "https://aaronpk.github.io/draft-parecki-oauth-client-id-metadata-document/draft-parecki-oauth-client-id-metadata-document.html"

author:
  - fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com
    uri: https://aaronparecki.com
  - fullname: Emelia Smith
    email: emelia@brandedcode.com

normative:
  RFC3986:
  RFC6749:
  RFC6819:
  RFC7591:
  I-D.draft-ietf-oauth-security-topics:

informative:


--- abstract

This specification defines a mechanism through which an OAuth client can
identify itself to authorization servers, without prior dynamic client
registration or other existing registration. This is through the usage of a URL
as a `client_id` in an OAuth flow, where the URL refers to a document containing
the necessary client metadata, enabling the authorization server to fetch the
metadata about the client as needed.

--- middle

# Introduction

In order for an OAuth 2.0 {{RFC6749}} client to utilize an OAuth 2.0
authorization server, the client needs to establish a unique identifier, and
needs to to provide the server with metadata about the application, such as the
application name, icon and redirect URIs. In cases where a client is interacting
with authorization servers that it has no relationship with, manual registration
is impossible.

While Dynamic Client Registration {{RFC7591}} can provide a method for a
previously unknown client to establish itself at an authorization server and
obtain a client identifier, this is not always practical in some deployments and
can create additional challenges around management of the registration data and
cleanup of inactive clients.

This specification describes how an OAuth 2.0 client can publish its
own registration information and avoid the need for pre-registering
at each authorization server.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Client Identifier

This specification defines the client identifier as a URL with the following
restrictions. Client identifier URLs MUST have an "https" scheme, MUST contain a
path component, MUST NOT contain single-dot or double-dot path segments, MUST
NOT contain a fragment component and MUST NOT contain a username or password
component. Client identifier URLs MAY contain a query string component and MAY
contain a port.

This specification places no restrictions on what URL is used as
a client identifier. A short URL is RECOMMENDED, since the URL may
be displayed to the end user in the authorization interface or in
management interfaces. Usage of a stable URL that does not frequently
change for the client is also RECOMMENDED.

# Client Information Discovery

One purpose of registering clients at the authorization server is so that
the authorization server has additional information about the client that
can be used during an OAuth flow, such as presenting information about
the client to the user in an authorization consent screen, for example the
client name and logo.

The authorization server SHOULD fetch the document indicated by the `client_id`
to retrieve the client registration information.

## Client Metadata

The client metadata document URL is a JSON document containing the metadata
of the client. The client metadata values are the values defined in
OAuth Dynamic Client Registration ({{RFC7591}}) section 2.

The client metadata document MUST contain a `client_id` property whose value
MUST compare and match the URL of the document using simple string comparison as
defined in [RFC3986] Section 6.2.1.

The client metadata document MAY define additional properties in the response.
The client metadata document MAY also be served with more specific content types
as long as the response is JSON and conforms to application/<AS-defined>+json.

Other specifications MAY place additional restrictions on the contents of the
client metadata document accepted by authorization servers implementing their
specification, for instance, preventing the registration of confidential clients
by requiring the `token_endpoint_auth_method` property be set to `"none"`.

## Metadata Discovery Errors

If fetching the metadata document fails, the authorization server SHOULD abort the
authorization request.

## Metadata Caching

The authorization server MAY cache the client metadata it discovers at the
client metadata document URL.

TBD: recommend a cache lifetime? considerations about stale data?

The authorization server MUST NOT cache error responses. The authorization
server also MUST NOT cache documents which are invalid or malformed.

TBD: Do we want to define an endpoint through which a document can be validated
by an authorization server, such that a developer can assert that their document
is valid?

## Redirect URL Registration

According to {{I-D.draft-ietf-oauth-security-topics}}, the authorization server
MUST require registration of redirect URLs, and compare redirect URLs with
exact string matching. This client information discovery establishes a
registered redirect URL with the authorization server which is used when
comparing the redirect URL in an authorization request against the registered
redirect URLs.

TBD: Is it exact string matching, or is it still using simple string comparison per [RFC3986]


# Security Considerations

In addition to the security considerations in OAuth 2.0 Core {{RFC6749}}, and OAuth 2.0 Threat Model and Security Considerations {{RFC6819}}, and {{I-D.draft-ietf-oauth-security-topics}} the additional considerations apply.

## Public vs Confidential Clients

Since the client establishes its own registration data at the authorization server,
prior coordination of client credentials is not possible. However, clients MAY establish
credentials at the authorization server by using authentication methods that use
public/private key pairs, by publishing the public key in their metadata document.

For example, the client MAY include the following properties in its metadata document
to establish a public key and the `private_key_jwt` authentication method:

    {
      ...
      "token_endpoint_auth_method": "private_key_jwt",
      "jwks_uri": "https://client.example.com/jwks.json"
      ...
    }

This establishes this client as a confidential client, and any communication with
the authorization server MUST include client authentication of the registered type.

## OAuth Phishing Attacks

Authorization servers SHOULD fetch the `client_id` metadata document provided in the authorization request in order to provide users with additional information about the request, such as the application name and logo. If the server does not fetch the client metadata document, then it SHOULD take additional measures to ensure the user is provided with as much information as possible about the request.

The authorization server SHOULD display the hostname of the `client_id` on the authorization interface, in addition to displaying the fetched client information if any. Displaying the hostname helps users know that they are authorizing the expected application.

If fetching the client metadata document fails for any reason, the `client_id` URL is the only piece of information the user has as an indication of which application they are authorizing.


## Server Side Request Forgery (SSRF) Attacks

Authorization servers fetching the client metadata document and resolving URLs located in the metadata document should be aware of possible SSRF attacks. Authorization servers SHOULD avoid fetching any URLs using private or loopback addresses and consider network policies or other measures to prevent making requests to these addresses. Authorization servers SHOULD also be aware of the possibility that URLs might be non-http-based URI schemes which can lead to other possible SSRF attack vectors.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
