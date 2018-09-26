LSP Authentication
==================

This document covers core technologies and interactions between
services, APIs, and applications interacting with the LSST Science
Platform as a whole.

Core Technologies
=================

InCommon Federation
-------------------

InCommon is an identity federation in the United States that provides a
common framework for identity management and trust across member
institutions. The InCommon Federation’s identity management is built on
top of eduPerson attributes, and the interfaces used to interact with
the federated institutions are typically Shibboleth and SAML attributes.

OAuth2
------

OAuth2 is a framework that enables *users* to authorize *applications*
to retrieve information, either in the form of a token or through the
use of a token, about the user from an identity provider. An identity
provider may be Google, Github or an institution. Typically,
institutions themselves do not implement OAuth2 interfaces, but do
implement interfaces with SAML or Shibboleth.

OAuth2 specifies how you may ask for information about a user. It also
specifies a method, through tokens, which a service may use to request
and validate information about the user.

With OAuth2, all tokens are transferred via the Authorization Header.

OpenID Connect
~~~~~~~~~~~~~~

OpenID Connect is an is an simple authentication layer on top of OAuth2.
OpenID Connect specifies a small set of information about a user which
may be used to authenticate a user using claims implemented in OAuth2.

Dynamic Client Registration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Typically, an service, such as Firefly, operates as an OAuth client to a
service which you may need access to. Due to implementation details of
OAuth, you typically need a client ID and a client secret in order to
register a client to receive information from the OAuth service (CILogon
in our case). This works fine when the client is under full
administrative control, like Firefly, but it falls apart if you want to
deploy a client to a user device, such as a desktop or mobile phone. In
those scenarios, you need to generate per-deployment client IDs and
secrets.

OAuth has a provision for that called Dynamic Client Registration.

In the cases of TOPCAT, Astroquery, or PyVO - in the context of a user
attached to an application, we think that CILogon may ideally support
Dynamic Client Registration. This would allow TOPCAT to work similarly
to a Mobile Application on a phone, for example.

JWT
---

A JSON Web Token (JWT) is a way of encoding specific claims for a
subject in JSON, as well as including signatures (JWS) of the token and
information on validating those signature (JWE/JWK) in the same token.
Included in the JWT specification is also a way of encoding a token
using Base64 in a way that’s friendly for the web.

See also: https://tools.ietf.org/html/rfc7519

CILogon
-------

CILogon is a generic authentication proxy/clearing house for
authentication providers from multiple institutions and services,
especially for institutions federated into the InCommon.org federation,
as well as other services such as Github and Google. It serves as a
common endpoint for these various identity providers and translates
their authentication mechanisms (OAuth2, Shibboleth+SAML, OpenID
Connect) mechanisms to a common authentication mechanism, while also
translating claims, when possible.

CILogon translates authentication information and user claims into
OpenID Connect claims through the OpenID Connect interfaces. Using this,
we typically know what institution a user is from, their email address,
and whether or not they are faculty, staff, or a student. We may use
this information to also map them to an NCSA user, provided that
information has been previously captured, and potentially retrieve
additional claims about that user, *such as LDAP groups they are a
member of.* Should we want additional claims beyond the subject of a
token - claims such as group membership or capabilities, we will need to
deploy a server which we can present a refresh token to that will
provide us with those additional claims. We do not expect this
implementation-specific needs to be included in CILogon.

SciTokens
---------

**SciTokens is our expected implementation of access tokens.**

SciTokens is an implementation of capabilities-based authorizations
based on JWT claims, modeled as ``read`` and ``write`` capabilities, on
named resources. These are implemented in a JWT token.

It’s expected that we will use some form of login flow from CILogon to
eventually translate claims about a user and add them to a SciTokens
token. It’s expected that there will be a service where a user would
present tokens acquired from CILogon to the SciTokens service to acquire
a refresh token and an access token, with a very limited lifetime, from
a SciTokens service. The refresh token may be used to acquire new access
tokens to present to services which will accept them. Services or
applications which frequently may need to operate on behalf of a user
for longer than the lifetime of an access token must acquire a refresh
token, which they can use to acquire access tokens.

When a SciToken is passed to a service, it is always passed according to
the OAuth standard for passing Bearer tokens. This means it’s passed in
the Authorization header, in the form of:

::

   Authorization: Bearer [TOKEN]

Token lifetimes
===============

Access token lifetimes are expected to be from 1 to 24 hours. An exact
number is not available. The SciTokens team recommends token lifetimes
in the tens of minutes, but this seems untenable, especially in the case
where a user may want to execute a job in a batch system in the grid or
a HPC system, where queues are typically several hours long.

Token Claims
============

An Access Token will only have an NCSA user ID in the ``sub`` claim by
default.

In some cases, a SciToken MAY come with read and write claims. In
accordance with the principle of lease-privilege, SciTokens also allows
a user to narrow down those claims with successive calls to a SciTokens
endpoint, trading an existing token for a narrower one with limited
access. This may be especially useful with Grid computing, for example.
It’s important to consider the lifetime of a token in these scenarios to
determine what token may be required.

Interactions
============

All of the following interactions have an assumption that a user is
registered and is already a member of requisite LDAP groups for
accessing LSST resources within NCSA.

All of the following interactions *also* assume that a user using
federated authentication has also associated their account from a third
parting Identity Provider to their account at NCSA, and that CILogon is
able to perform that association and return information about who the
user is at NCSA.

It is also assumed that the Portal and Notebook applications have
registered as OpenID Connect clients to CILogon.

Portal Aspect (Firefly)
-----------------------

When a user first logs into the portal, they will be redirected to the
token server. They may select either NCSA as their Identity Provider or
their home institution. CILogon executes the login, ultimately returning
information about who the user is at NCSA to the portal aspect through
CILogon’s OpenID Connect interface. This providers the Portal aspect
with an access token and a refresh token.

In OAuth2, the SciToken is an access token. A service like Firefly is an
OAuth2 client and will also receive a refresh token (allowing it to
generate additional SciTokens, since the access token is relatively
short-lived). When calls are made to DAX, the SciToken (an access token)
is used as a HTTP Bearer token (e.g.,
``Authorization: Bearer [TOKEN]``).

Portal to DAX
-------------

The Portal will send the access token to a DAX service. The Portal
should configure an HTTP client with an authentication filter that can
check the expiration of the access token, and, if necessary, use the
refresh token to acquire a new access token from the Token server before
issuing a request to the DAX services.

Notebook Aspect
---------------

The Portal and the notebook should share some common session information
about the user, including refresh tokens, to enable smooth transitions
and interoperability between the two. How this is implemented is
undefined.

Once a user is logged in to the Notebook access, a user in the Notebook
aspect can be viewed as a special case of Third Party access where we
have some access to the user’s local environment, so we may be able to
bootstrap an authentication mechanism on behalf of the user which
ensures any necessary tokens are implicitly available in the user’s
environment. For software developed by the LSST project, software which
may use the DAX services, such as the Butler, we will ensure those
applications can be automatically configured based on some form of
information in the user’s Notebook environment. Other third party
software *may* be automatically configured, or they should be
configurable in the same way as if a user was running on their local
machine and not in an LSP instance.

Astroquery/PyVO
~~~~~~~~~~~~~~~

We are targeting Astroquery an PyVO as primary libraries to be used
within the Notebook environment.

DAX
---

Authentication to DAX services is performed by using Access tokens only.
Applications calling into DAX services are responsible for ensuring an
access token is valid and hasn’t expired before calling into a DAX
service.

The Access token received by DAX can, in turn, be forwarded to other
SciToken-aware resources. If Qserv can accept SciTokens directly, this
can be done. Otherwise, if the DAX service must call another service
within the DAC, such as stashing the results of a long-running TAP query
in a user’s workspace after the expiration of an access token, we will
allow the DAX services to perform that operation by through trust of the
DAX services. This may prove complicated in the case of requests that
span multiple LSP instances or DACs. Should that become complicated, it
may be required for the DAX services to issue a new token which can be
honored at different sites.

VOSpace/WebDAV
~~~~~~~~~~~~~~

In general, users must use access tokens to interoperate with these
services. We anticipate VOSpace may possibly be implemented over FTS3.

Due to the management of file transfers in a service like FTS3 being
potentially managed in a batch-like system, we may need access tokens to
live for 24 hours or more. It may also be necessary to have a mechanism
to force acquisition of a new access token before a file transfer
request is submitted.

Third Party
-----------

We expect there to be an explicit flow a user must engage in for all
third party authentication. It’s not clear if a user of third-party
applications will share a common token (e.g. Refresh Token) or if a user
will need to explicitly retrieve tokens for all third party services. At
least in the case of TOPCAT, we are incentivized to make the process as
easy as possible, and we will work with the TOPCAT developer closely to
develop an optimal solution.

In the case of X.509 certifications, for applications such as GSI-SSH, a
certificate is typically written out to a well-defined location in the
system’s temporary disk space (e.g. ``/tmp/x509up_u${UID}``),
``.globus`` for Windows users) for reuse by all GSI-enabled
applications. A similar convention would need to be constructed to allow
multiple third party applications to share a common set of credential,
or we can stash a token in an environment variable.

TOPCAT
~~~~~~

We will work closely with TOPCAT developers to find an optimal solution.
It’s possible that TOPCAT is ideally modeled as mobile application which
acquires a refresh token and manages the access token locally.

.. _astroquerypyvo-1:

Astroquery/PyVO
~~~~~~~~~~~~~~~

In the case of Astroquery, PyVO, or other third party applications, we
expect a user to either explicitly log-in or acquire a token from an
LSST token UI and programmatically configure their clients.
