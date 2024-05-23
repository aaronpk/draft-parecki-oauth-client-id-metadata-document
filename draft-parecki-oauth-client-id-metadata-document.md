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
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com
    url: https://aaronparecki.com
 -
    fullname: Emelia Smith
    email: emelia@brandedcode.com

normative:
  RFC6749:
  I-D.draft-ietf-oauth-security-topics:

informative:
  RFC7591:


--- abstract

This specification defines an endpoint an OAuth client can use to host its client metadata, as well as the use of this URL as a `client_id` in an OAuth flow. This enables OAuth clients to work with authorization servers where it has no prior relationship, and enables the authorization server to fetch metadata about the client.


--- middle

# Introduction

In order for an OAuth 2.0 {{RFC6749}} client to utilize an OAuth 2.0
authorization server, the client needs to establish a unique
identifier, and needs to to provide the server with metadata about
the application, such as the application name and icon.  In cases
where a client is interacting with authorization servers that it has
no relationship with, manual registration is impossible.

While Dynamic Client Registration {{RFC7591}} can provide a method for a previously
unknown client to establish itself at an authorization server and
obtain a client idenfier, this is not always practical in some deployments
and can create additional challenges around management of the registration
data and cleanup of inactive clients.

This specification describes how an OAuth 2.0 client can publish its
own registration information and avoid the need for pre-registering
at each authorization server.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Client Identifier

This specification defines the client identifier as a URL with
the following restrictions. Client identifier URLs MUST have
an "https" scheme, MUST contain a path component, MUST NOT
contain single-dot or double-dot path segments, MAY contain a query
string component, MUST NOT contain a fragment component, MUST NOT
contain a username or password component, and MAY contain a port.


# Client Information Discovery

One purpose of registering clients at the authorization server is so that
the authorziation server has additional information about the client that
can be used during an OAuth flow, such as presenting information about
the client to the user in an authorziation consent screen, for example the
client name and logo.

## Redirect URL Registration

According to {{I-D.draft-ietf-oauth-security-topics}}, the authorization server
MUST require registration of redirect URLs, and compare redirect URLs with
exact string matching. This client information discovery establishes a
registered redirect URL with the authorization server which is used when
comparing the redirect URL in an authorization request against the registered
redirect URLs.


# Security Considerations

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


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
