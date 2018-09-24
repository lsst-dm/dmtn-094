LSP Authentication
==================

Core Technologies
=================

OAuth2
------

OAuth2 is a framework that enables *users* to authorize *applications*
to retrieve information, either in the form of a token or through the
use of a token, about the user from an identity provider. An identity
provider may be Google, Github or something else. CILogon functions as
sort of a meta identity provider which implements the OAuth API.

OAuth2 specifies how you may ask for information about a user. It also
specificies a method, through tokens, which a service may use to request
and validate information about the user.

With OAuth2, all tokens are transferred via the Authorization Header.

OpenID Connect
~~~~~~~~~~~~~~

OpenID Connect is an is an simple authentication layer on top of OAuth2.
OpenID Connect specifies a small set of information about a user which
may be used to authenticate a user using claims implemented in OAuth2.

Dynamic Client Registration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the cases of TOPCAT, Astroquery, or PyVO - in the context of a user
attached to an application, we think that CILogon would ideally support
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
authentication providers from multiple institutions and services
(e.g. Github, Google, NCSA, Stanford). It serves as a common endpoint
for the various providers and translates their authentication mechanisms
to a common authentication mechanism, while also translating claims when
possible. We use CILogon to transform authentication information and
claims into OAauth-style claims through CILogon’s OpenID Connect
interface. With this, we typically know what institution a user is from,
their email address, and whether or not they are faculty, staff, or a
student. We may use this information to also map them to an NCSA user,
provided that information has been previously captured, and potentially
retrieve additional claims about that user, *such as LDAP groups they
are a member of.*

SciTokens
---------

SciTokens is an implementation of capabilities-based authorizations
based on JWT claims, modeled as ``read`` and ``write``, on top of a very
simple JWT token.

It’s expected that we will use some form of login flow from CILogon to
eventually translate claims about a user and add them to a SciTokens
token. It’s expected that there will be a service where a user would
present tokens acquired from CILogon to the SciTokens service to acquire
a refresh token and an access token, with a very limited lifetime, from
a SciTokens service. The refresh token may be used to acquire new access
tokens to present to services which will accept them. **This implies
that services or applications which frequently may need to operate on
behalf of a user for longer than the lifetime of an access token must
acquire a refresh token**.

Token lifetimes
===============

Access token lifetimes are expected to be from 1 to 24 hours. An exact
number is not available.

Refresh tokens effectively never expire. They can be used to acquire new
Access tokens.

Token Claims
============

A SciToken will only have an NCSA user ID in the claim by default.

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

When a user first logs into the portal, they will be redirected to
CILogon. They may select either NCSA as their Identity Provider or their
home institution. CILogon executes the login, ultimately returning
information about who the user is at NCSA to the portal aspect through
CILogon’s OpenID Connect interface. This providers the Portal aspect
with an access token and a refresh token.

Portal to DAX
-------------

The Portal will send the access token to a DAX service. The Portal
should configure an HTTP client with an authentication filter that can
check the expiration of the access token, and, if necessary, use the
refresh token to acquire a new access token from the Token server before
issuing a request to the DAX services.

Notebook Aspect
---------------

The Portal and the notebook should share some commmon session
information about the user, including refresh tokens, to enable smooth
transitions and interoperability between the two. How this is
implemented is undefined.

Once a user is logged in to the Notebook access, a user in the Notebook
aspect can be viewed as a special case of Third Party access where we
have some access to the user’s local environment, so we may be able to
bootstrap an authentication mechanism on behalf of the user which
ensures any necessary tokens are implicitly available in the user’s
envioronment. For software developed by the LSST project, software which
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

If the DAX service must call another service within the DAC, such as
stashing the results of a long-running TAP query in a user’s workspace
after the expiration of an access token, we will allow the DAX services
to perform that operation by trusting them.

VOSpace/WebDAV/FTS3
~~~~~~~~~~~~~~~~~~~

In general, users must use access tokens to interoperate with these
services.

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
TOPCAT may be a special case, and may be ideally treated similarly to
a mobile application. In that case, we may need Dynamic Client 
Registration supported in CILogon.

.. _astroquerypyvo-1:

Astroquery/PyVO
~~~~~~~~~~~~~~~

In the case of Astroquery, PyVO, or other third party applications, we
expect a user to either explicitly log-in or acquire a token from an
LSST token UI and programmatically configure their clients.
