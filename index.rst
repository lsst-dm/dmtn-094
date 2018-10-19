LSP Authentication
==================

Introduction
============

This document covers core technologies and interactions between
services, APIs, and applications interacting with the LSST Science
Platform. The authentication use cases covered in this cover interactive
users who are primarily interacting with the LSP in the following ways:

-  Logged into the notebook aspect
-  Logged into the portal aspect
-  From a local terminal, exercising the API aspect from third party
   libraries or applications

For the users in question, we make a few assumptions about use of the
LSP in this context:

-  Users are active, continuously or intermittently, over the course of
   an extended work day. Interactions will typically last several hours,
   though the system should be prepared for interactions lasting up to
   24 hours
-  At the start of an interaction with the LSP system, users do not know
   which types of data they will touch.
-  Users include LSST staff, and scientists that are members of science
   collaborations.
-  Users already have an NCSA account through some mechanism. If they
   wish use federated credentials, the assumption is that their
   federated credentials have been associated with their NCSA account.

Due to these assumptions, this document will focus on identity-based
authentication and authorization, though we will cover aspects of
role-based and capabilities-based authorization for the LSP.

SciTokens is mentioned specifically for capabilties-based authorization,
though that is informational.

Identity and Access Management
==============================

For clarity, we further define users and groups within this document, as
well as minimal requirements from the IAM system as needed by the LSP,
and likely the broader Data Management System. It's expected further
definition of the IAM system will be available in the future in a change
controlled document.

User
----

A user is minimally identified by either a UNIX ID number (UID) and a
user name. A service MUST be able to lookup a UID if it has a user name,
or a user name if it has a UID.

Real Accounts
~~~~~~~~~~~~~

A user account identifying a specific person is a *real account*.

Shared Accounts
~~~~~~~~~~~~~~~

A user account may actually be a *shared account*, which does not
identify a specific person. A shared account must have at least one real
account managing the shared account.

A shared account is nominally for the use and organization of shared
resources, such as disk storage and limited compute, if available.

Shared Account Use Cases
^^^^^^^^^^^^^^^^^^^^^^^^

A shared account might be created a number of purposes. This might
include:

-  Projects; such as Stack Club
-  Teams; such as Alerts, DAX, SQuaRE
-  Science Collaborations
-  Data Releases

Groups
------

A user is a member of one or more groups. All groups defined for a given
user are owned exclusively by that user. The IAM system SHOULD disallow
group names that are not representable as UNIX group names within the
Data Management System, including file systems and databases. Users
managing a user that is a `shared account <#shared-accounts>`__ are
implicitly allowed to manage the groups of that user account, including
the creation of them, and membership of them.

A core set of groups do not belong to a specific user. These are defined
and managed by the LSST system administrators.

Group membership MUST be discoverable through at least an LDAP service
provided by the IAM system. Additional services for querying group
membership MAY be implemented.

Clients querying a group membership service SHOULD cache results.
Results SHOULD be cached with a TTL for no less than 30 seconds and no
longer than 1 hour. A 5 minute TTL is recommended.

Users and services should be made aware of the caching TTL as well as
potential latencies due to `user and groups
synchronization <#user-and-groups-synchronization>`__. It may take up to
2 hours for groups to be synchronized.

Some systems may not allow assignment of permissions to users, only
groups. If this is the case, then users MUST be part of a group whose
only membership is the user. This would enable another user to extend
access to a resource by assigning a read permission to the user's
primary group, for example.

User and Groups Synchronization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When necessary, the IAM system SHOULD create users and groups in
underlying systems when necessary, and sychronize membership
accordingly. The sychronization SHOULD finish in under an hour, and MUST
finish within 24 hours.

In databases, groups should be represented as roles.

The assignment of privleges on resources according to users and groups
is out of scope for this document.

Roles
-----

There's currently no concept of roles in the existing IAM system for
NCSA. A system that represents roles must also have permissions
associated with roles. As such, Roles and are generally out of scope for
this document, but they are mentioned for informational purposes.

It's possible that roles may be implemented through some combination of
`shared accounts <#shared-accounts>`__ and group membership. For
example, a ``firefly`` shared account may have the groups
``firefly_usdac_user``, ``firefly_pdac_user``, and ``firefly_admin``
defined. In this example, these groups are effectively roles.

Capabilities-based Authorization
--------------------------------

We expect some form of capabilities-based authorization will be useful
for the Data Management System in the future. This section will outline
what that is and the requirements for such a system to be implemented.

Capabilties-based security system is based on the `object-capability
security
model <https://en.wikipedia.org/wiki/Object-capability_model>`__.

A capabilities-based system, in the context of LSST DM system, would
rely on:

1. A definition of resources across the LSST DM system to which you can
   assign access rights to; such as dataset collections (butler repos),
   database tables, services.
2. A reference to a resource or set of resources; such as a token, which
   the system can validate and enforce access control
3. A definition of operations to be performed on the resource; such as
   ``read``, ``write``, ``delete``, ``execute``, for example.

Together, the reference and operation can be included in a message and
will represent a capability. In order for the system to be secure, the
message MUST be unforgeable. This would typically implemented through a
cryptographic signature.

For the issuance of the capabilities, the following are required:

-  A method of determining the set of those capabilities for a given
   user or use case; and
-  A system which either implements that method, which issues the
   unforgeable message (a token or certificate); or
-  A system that is notified notified by another system implementing the
   method;

Low-level systems, including disk storage (NFS, GPFS, S3/Swift/Ceph) and
databases (Oracle, MySQL), do not have a way of enforcing
capabilities-based authorizations. As such, to implement a
capabilities-based security system, it's required to have a service in
front of those systems which can process the messages.

To process a request with a capabilities message, a service MUST:

-  Agree to the definiton of resources issued in the message, mapping
   them to the system the system (or underlying system) manages
-  Agree to the definition of operations in the message; mapping them to
   the operations the system (or underlying system) implements
-  Examine the request and verify ALL resource and operation pairs a
   request may need are represented in the message.

Authentication
==============

Authentication by a `real user <#real-accounts>`__ is handled by the IAM
system. All authentication for LSP services are handled through the
OAuth 2.0 Protocol by the IAM system. Normally this will be through the
OpenID Connect layer.

Authentication for a `shared account <#shared-accounts>`__ is out of
scope for this document. It is expected that users may be members of
groups that are owned by shared accounts, but they will always
authenticate as themselves.

Authentication using means such as kerberos is out of scope of this
document.

Service Access Authorization
----------------------------

LSP services MAY limit access by users through group membership. In
these cases, a service needs to acquire a list of groups associated with
a user, either as claims in a token, or through a membership query to a
service.

Data Access Authorization
-------------------------

Low-Level systems SHOULD be relied upon to authorize access to data.
This includes:

-  Disk Storage, such as NFS, GPFS;
-  Databases, such as Oracle or Qserv

Technologies
============

This section will cover some technologies used by both the IAM system
and the LSP system to meet the goals of the LSP system.

-  `InCommon <#incommon-federation>`__ and eduPerson to verify
   attributesabout scientists, when possible;
-  `CILogon <#cilogon>`__ to federate those identities and implement
   return identity data about users in the form of *claims*.
-  `OAuth 2.0 <#oauth-2.0>`__ as the generic protocol to interface with
   CILogon. OpenID Connect is layered over the OAuth 2.0 protocol to
   required for an authentication implementation.
-  `JWT <#jwt>`__ as the implementation for identity tokens. This is
   also required as a result of using OpenID Connect.

InCommon Federation
-------------------

InCommon is an identity federation in the United States that provides a
common framework for identity management and trust across member
institutions. The InCommon Federation's identity management is built on
top of eduPerson attributes. The interface used to interact with the
federated institutions is Shibboleth.

.. _oauth-20:

OAuth 2.0
---------

OAuth2 is a framework that enables users to authorize applications to
retrieve information, either in the form of a token or through the use
of a token, about the user from an identity provider. An identity
provider may be Google, Github or an institution. Typically,
institutions themselves do not implement OAuth 2.0 interfaces, but do
implement interfaces with Shibboleth and SAML.

OAuth 2.0 specifies how you may ask for information about a user. It
also specifies a method, through tokens, which a service may use to
request and validate information about the user.

According to the OAuth 2.0 protocol, all tokens are transferred via the
Authorization Header:

   ``Authorization: Bearer [TOKEN]``

OpenID Connect
~~~~~~~~~~~~~~

OpenID Connect is an simple authentication layer on top of OAuth2.
OpenID Connect specifies a small set of information about a user which
may be used to authenticate a user using claims implemented according to
the OAuth2 specification.

OpenID Connect Dynamic Client Registration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Typically, a service, such as Firefly, operates as an OAuth2 client to a
service - the resource server. Due to implementation details of OAuth,
you typically need a client ID and a client secret in order to register
a client to receive information from the OAuth service (CILogon in our
case). This works fine when the client is under full administrative
control, like Firefly, but it falls apart if you want to deploy a client
to a user-controlled device, such as a desktop or mobile phone. In those
scenarios, you should generate per-deployment client IDs and secrets.

OAuth has a provision for that called Dynamic Client Registration.

In the cases of TOPCAT, Astroquery, or PyVO - it may be beneficial to
model either an application (TOPCAT) or the device itself as a client,
should CILogon to support Dynamic Client Registration. This would allow
TOPCAT or any application on the system to work similarly to a Mobile
Application on a phone.

CILogon
-------

CILogon is a generic authentication proxy/clearing house for
authentication providers from multiple services or institutions,
especially institutions federated into the InCommon federation, as well
as other services such as Github and Google. CILogon serves as a common
endpoint for these various identity providers and translates their
authentication mechanisms (OAuth 2.0, Shibboleth, OpenID Connect)
mechanisms to a common authentication mechanism, often while also
translating claims, when possible.

CILogon translates authentication information and user claims into
OpenID Connect claims, layered on the OAuth 2.0 protocol. Using this, we
typically know what institution a user is from, their email address, and
whether or not they are faculty, staff, or a student. We may use this
information to also map them to an NCSA user, provided that information
has been previously captured, and potentially retrieve additional claims
about that user, such as the `groups <#groups>`__ they are a member of.
Should we want additional claims beyond the subject of a token - claims
such as group membership or capabilities, we will need to deploy a
server which we can present a refresh token to that will provide us with
those additional claims. We do not expect this implementation-specific
needs to be included in CILogon.

JWT
---

A JSON Web Token (JWT) is a way of representing claims to as JSON, as
well as information for validating those claims through the use of
signatures (JWS) in the token, and a meants of validating those
signature (JWE/JWK) - all in the same token. Included in the JWT
specification is also a way of encoding a token using Base64 in a way
that's friendly for the web.

For all LSST Applications, we will use RS256, an asymmetric algorithm,
to sign the tokens.

We will be relying primarily on tokens generated by CILogon. In certain
cases, the services MAY issue tokens that should be honored by other
services. The primary use case of this is to ensure a request is
completed by the system.

A whitelist of token issuers we trust MUST be maintained, and services
that validate tokens MUST be configurable with that whitelist. Public
keys used to validate tokens must be available on all token issuers,
follwing to the JWK specification. Applications should cache the JWK for
a given token issuer for at least 5 minutes and not more than 1 hour.

All Access Tokens will be based on JWT. Some access tokens may also
include claims implemented according to the SciTokens specification.

See also: https://tools.ietf.org/html/rfc7519

SciTokens
---------

This section is informative, though as mentioned in the introduction, we
are focusing on tokens representing user identity.

SciTokens is an implementation of `capabilities-based
authorizations <#capabilities-based-authorization>`__ built as specific
claims inside a JWT token. Those claims are modeled as ``read`` and
``write`` capabilities, on named resources. A named resource may be a
file path (e.g. ``read:/datasets/catalogs``) or a a more general
resource (e.g. ``read:mysql://server:3806/schema``)

SciTokens recommend NOT using the subject (``sub`` claim) for identity
purposes. This implies that SciTokens SHOULD NOT be used for
authorizations based on identity.

It's expected that we will use some form of login flow from CILogon to
eventually translate claims a client may have to a SciTokens token. It's
expected that there will be a service where a user would present tokens
acquired from CILogon to the SciTokens service to acquire a refresh
token and an access token, with a very limited lifetime, from a
SciTokens service. The refresh token may be used to acquire new access
tokens to present to services which will accept them. Services or
applications which frequently may need to operate on behalf of a user
for longer than the lifetime of an access token must acquire a refresh
token, which they can use to acquire access tokens.

When a SciToken is passed to a service, it is always passed according to
the OAuth standard for passing Bearer tokens. This means it's passed in
the Authorization header, in the form of:

::

   Authorization: Bearer [TOKEN]

Token lifetimes
===============

Access token lifetimes are expected to be short, typically on the order
of several hours or less, but may last as long as 24 hours, depending on
the issuer and use case. An exact number is not available.

Refresh tokens, which are used to acquire access tokens in the OAuth 2.0
protocol, can last longer. It's expected a refresh token will last at
least 24 hours and may last as long as a week. In some limited use
cases, they may last longer.

DAX services intend to guarantee all requests received that a DAX
services recieved will succeed. To work with shorter access token
lifetimes, the succeed. In order to guarantee this, the DAX services
MUST issue a new token with the same claims which ONLY other DAX
services will be configured to honor. The lifetime of this token is not
specified, but it should the upper bound for the limit of time it takes
to service a request, around 24 hours.

DAX services SHOULD NOT issue new tokens from requests with DAX-issued
tokens.

Token Claims
============

Access tokens used for identity-based authorizations, issued from the
appropriate token issuers, MUST have the UID, user name, or fully
qualified user name (email address) in the ``sub`` claim of an access
token. This will allow a service to identify the `user <#user>`__.

SciTokens Token Claims
----------------------

A SciToken MUST come with a ``scope`` claim. The ``scope`` claim is a
space-sperated list of capabilities.

In accordance with the principle of least-privilege, a SciTokens issuer
SHOULD also allow a user to attenuate those claims with successive calls
to the SciTokens issuer, trading an existing token for attenuated one.
This may be especially useful with Grid computing, for example. It's
important to consider the lifetime of a token in these scenarios to
determine what token may be required.

Interactions
============

All of the following interactions have an assumption that a user is
registered and is already a member of requisite LDAP groups for
accessing LSST resources within NCSA.

All of the following interactions als assume that a user using federated
authentication has also associated their account from a third parting
Identity Provider to their account at NCSA, and that CILogon is able to
perform that association and return information about who the user is at
NCSA.

It is also assumed that the Portal and Notebook applications have
registered as OpenID Connect clients to CILogon.

Portal Aspect (Firefly)
-----------------------

When a user first logs into the portal, they will be redirected to the
token issuer. They may select either NCSA as their Identity Provider or
their home institution. CILogon executes the login, ultimately returning
information about who the user is at NCSA to the portal aspect through
CILogon's OpenID Connect interface and the token's ``sub`` claim. This
provides the Portal aspect with an access token and a refresh token.

Firefly is an OAuth 2.0 client and SHOULD use the refresh token to
generate new access tokens. When calls are made to DAX, the access token
is passed as an OAuth 2.0 Bearer token in the HTTP ``Authorization``
header, according to the OAuth 2.0 Specification:

   ``Authorization: Bearer [TOKEN]``

Portal to DAX
-------------

The Portal will send the access token to a DAX service. The Portal
SHOULD configure an HTTP client with an authentication filter that can
check the expiration of the access token, and, if necessary, use the
refresh token to acquire a new access token from the token issuer before
issuing a request to the DAX services.

Notebook Aspect
---------------

The Portal and the notebook should share some common session information
about the user, including refresh tokens, to enable smooth transitions
and interoperability between the two. How this is implemented is
undefined.

Once a user is logged in to the Notebook access, a user in the Notebook
aspect can be viewed as a special case of Third Party access where we
have some access to the user's local environment, so we may be able to
bootstrap an authentication mechanism on behalf of the user which
ensures any necessary tokens are implicitly available in the user's
environment. For software developed by the LSST that may utilize the DAX
services, such as the Butler, we will ensure those applications can be
automatically configured based on some form of information in the user's
Notebook environment. Other third party software MAY be automatically
configured, or they should be configurable in the same way as if a user
was running on their local machine and not in an LSP instance.

Astroquery/PyVO
~~~~~~~~~~~~~~~

We are targeting Astroquery an PyVO as primary libraries to be used
within the Notebook environment.

DAX
---

Authentication to DAX services is performed by using access tokens only.
Applications calling into DAX services are responsible for ensuring an
access token is valid and hasn't expired before calling into a DAX
service.

A DAX service SHOULD reissue a new token if the service needs to issue a
request to another DAX service, as laid out in the `token
lifetimes <#token-lifetimes>`__ section. If Qserv can accept tokens
directly, the token SHOULD be passed as the username.

We expect a PAM module to be a product of the IAM system which can
accept an access token and login a user.

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
third party authentication. It's not clear if a user of third-party
applications will share a common token (e.g. Refresh Token) or if a user
will need to explicitly retrieve tokens for all third party services. At
least in the case of TOPCAT, we are incentivized to make the process as
easy as possible, and we will work with the TOPCAT developer closely to
develop an optimal solution.

In the case of X.509 certifications, for applications such as GSI-SSH, a
certificate is typically written out to a well-defined location in the
system's temporary disk space (e.g. ``/tmp/x509up_u${UID}``),
``.globus`` for Windows users) for reuse by all GSI-enabled
applications. A similar convention would need to be constructed to allow
multiple third party applications to share a common set of credential,
or we can stash a token in an environment variable.

TOPCAT
~~~~~~

We will work closely with TOPCAT developers to find an optimal solution.
It's possible that TOPCAT is ideally modeled as mobile application which
acquires a refresh token and manages the access token locally.

.. _astroquerypyvo-1:

Astroquery/PyVO
~~~~~~~~~~~~~~~

In the case of Astroquery, PyVO, or other third party applications, we
expect a user to either explicitly log-in or acquire a token from an
LSST token UI and programmatically configure their clients.
