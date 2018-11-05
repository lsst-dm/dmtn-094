# Sequence Diagrams
## Phase 1 - Identity Tokens
### Notebook with Identity Tokens
title Authentication to Notebook with CILogon OAuth flow (OpenID Connect)
Browser->Notebook: Request to Login
Notebook->Browser: Redirect to login service
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth claims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Notebook: Access Code
Notebook->CILogon: Present Access code
CILogon->Notebook: OpenID Connect Refresh and Access tokens (Identity tokens)
Notebook->Notebook: Authorize group access
Notebook->Browser: Authorization okay, Notebook starting up
Notebook->Notebook: Spinning up container, stash OpenID Connect \nRefresh and Access token in User Environment
Browser->Notebook: (Data Access Library) Start interacting with python, \naccess token lasts 24 hours
Notebook->Data Access Library: Initialize data access library
Data Access Library->Data Access Library: Initialize supported \ntoken manager with configured tokens from environment
Data Access Library->TAP: Async TAP request, with Access token
TAP->TAP: (Authenticator) Reissue new token with appropriate expiration
TAP->Token Proxy: (Authenticator) Reissue new token with appropriate expiration
Token Proxy->TAP: Reissued token
TAP->TAP: Create TAP job, stash reissued token in job
TAP->Data Access Library: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Query Results
TAP->WebDAV: Present token, results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Data Access Library->TAP: Request for Results
TAP->Data Access Library: Redirect to WebDAV
Data Access Library->WebDAV: Request for Results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Retrieve File
WebDAV->Data Access Library: Respond with Results
Data Access Library->Browser: Interactive table widget

### Portal with Identity Tokens
title Authentication for Portal with data request, using CILogon and OpenID Connect
Browser->Portal: Request to Login
Portal->Browser: Redirect to login service
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth claims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Portal: Access Code
Portal->CILogon: Present Access code
CILogon->Portal: OpenID Connect Refresh token (Identity token)
Portal->Portal: Authorize group access
Portal->Portal: Stash Identity refresh token in Token Manager for user session
Portal->Browser: Authorization okay, Portal initializing
Browser->Portal: Activate a cone search
Portal->Portal: Verify access token exists and is valid
Portal->CILogon: (Token Manager) Acquire Identity token (Access token)
CILogon->Portal: (Token Manager) Return Identity Token
Portal->TAP: Async TAP request, with Access token
TAP->TAP: (Authenticator) Reissue new token with appropriate expiration
TAP->Token Proxy: (Authenticator) Reissue new token with appropriate expiration
Token Proxy->TAP: Reissued token
TAP->TAP: Create TAP job, stash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Query Results
TAP->WebDAV: Present token, results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Portal->TAP: Request for Results
TAP->Portal: Redirect to WebDAV
Portal->WebDAV: Request for Results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Retrieve File
WebDAV->Portal: Respond with Results
Portal->Browser: Interactive Table Results Page

### Application with Identity Tokens
title Authentication for Application with data request, using CILogon and OpenID Connect
User->Application: Initialize Application
Application->Browser: Redirect to login service
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth claims for User (with email)
CILogon->Browser: Redirect to localhost with access code
Browser->User: Copy URL parameters in URL
User->Application: Access Code
Application->CILogon: Present Access code
CILogon->Application: OpenID Connect Refresh and Access tokens (Identity tokens)
Application->Token Issuer: Request for Capabilities Refresh \n(with only TAP capabilities)
Token Issuer->Application: Capabilities Refresh Token
Application->Application: Initialize Token Manager
Application->User: Authorization okay, Application initialized
User->Application: Perform Cone Search
Application->TAP: Async TAP request, with capability token
TAP->TAP: (Authenticator) Reissue new token with appropriate expiration
TAP->Token Proxy: (Authenticator) Reissue new token with appropriate expiration
Token Proxy->TAP: Reissued token
TAP->TAP: Create TAP job, stash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Return Query Results
TAP->WebDAV: Present token, results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Application->TAP: Request for Results
TAP->Application: Redirect to WebDAV
Application->WebDAV: Request for Results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Retrieve File
WebDAV->Application: Respond with Results
Application->User: Show table results

## Phase 2 - Capability Tokens

### Notebook with Capability Token
title Authentication to Notebook with CILogon OAuth flow and Capability token
Browser->Notebook: Request to Login
Notebook->Browser: Redirect to login service
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth claims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Notebook: Access Code
Notebook->CILogon: Present Access code
CILogon->Notebook: OpenID Connect Refresh and Access tokens (Identity tokens)
Notebook->Notebook: Authorize group access
Notebook->Token Issuer: Request for Capabilities Refresh \n(with only necessary capabilities)
Token Issuer->Notebook: Capabilities Refresh Token
Notebook->Browser: Authorization okay, Notebook starting up
Notebook->Notebook: Spinning up container, stash Capabilties \nRefresh and Access token in User Environment
Browser->Notebook: Start interacting with python, access token lasts 24 hours
Notebook->Data Access Library: Initialize data access library
Data Access Library->Data Access Library: Initialize supported \ntoken manager with configured tokens from environment
Data Access Library->TAP: Async TAP request, with Access token
TAP->TAP: (Authenticator) Check capability
TAP->TAP: (Authenticator) Reissue new token with appropriate expiration
TAP->Token Proxy: (Authenticator) Reissue new token with appropriate expiration
Token Proxy->TAP: Reissued token
TAP->TAP: Create TAP job, stash reissued token in job
TAP->Data Access Library: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Query Results
TAP->WebDAV: Present token, results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Data Access Library->TAP: Request for Results
TAP->Data Access Library: Redirect to WebDAV
Data Access Library->WebDAV: Request for Results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Retrieve File
WebDAV->Data Access Library: Respond with Results
Data Access Library->Browser: Interactive table widget

### Portal with Capability Token
title Authentication to Portal with data request, using capability token
Browser->Portal: Request to Login
Portal->Browser: Redirect to login service
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth claims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Portal: Access Code
Portal->CILogon: Present Access code
CILogon->Portal: OpenID Connect Refresh token (Identity token)
Portal->Portal: Authorize group access
Portal->Token Issuer: Request for Capabilities Refresh \n(with only necessary capabilities)
Token Issuer->Portal: Capabilities Refresh Token
Portal->Portal: Stash Capabilities refresh token in Token Manager for user session
Portal->Browser: Authorization okay, Portal initializing
Browser->Portal: Activate a cone search
Portal->Portal: Verify access token exists and is valid
Portal->CILogon: (Token Manager) Acquire capability token (Access token)
CILogon->Portal: (Token Manager) Return capability token
Portal->TAP: Async TAP request, with capability token
TAP->TAP: (Authenticator) Check capability
TAP->TAP: (Authenticator) Reissue new token with appropriate expiration
TAP->Token Proxy: (Authenticator) Reissue new token with appropriate expiration
Token Proxy->TAP: Reissued token
TAP->TAP: Create TAP job, stash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Return Query Results
TAP->WebDAV: Present token, results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Portal->TAP: Request for Results
TAP->Portal: Redirect to WebDAV
Portal->WebDAV: Request for Results
WebDAV->WebDAV: (Authenticator) Authorize Capabilities
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Retrieve File
WebDAV->Portal: Respond with Results
Portal->Browser: Interactive Table Results Page

### Application with Capability Token
> TODO: Not clear how a user, in conjunction with the application,
> specifies the capabilties they need.

title Authentication for Application with data request, using capability token
User->Application: Initialize Application
Application->Browser: Redirect to login service
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth claims for User (with email)
CILogon->Browser: Redirect to localhost with access code
Browser->User: Copy URL parameters in URL
User->Application: Access Code
Application->CILogon: Present Access code
CILogon->Application: OpenID Connect Refresh and Access tokens (Identity tokens)
Application->Token Issuer: Request for Capabilities Refresh \n(with only necessary capabilities)
Token Issuer->Application: Capabilities Refresh Token
Application->Application: Initialize Token Manager
Application->User: Authorization okay, Application initialized
User->Application: Perform Cone Search
Application->TAP: Async TAP request, with capability token
TAP->TAP: (Authenticator) Check capability
TAP->TAP: (Authenticator) Reissue new token with appropriate expiration
TAP->Token Proxy: (Authenticator) Reissue new token with appropriate expiration
Token Proxy->TAP: Reissued token
TAP->TAP: Create TAP job, stash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Return Query Results
TAP->WebDAV: Present token, results
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Application->TAP: Request for Results
TAP->Application: Redirect to WebDAV
Application->WebDAV: Request for Results
WebDAV->WebDAV: (Authenticator) Authorize Capabilities
WebDAV->WebDAV: (Authenticator) Impersonate User
WebDAV->File System: Retrieve File
WebDAV->Application: Respond with Results
Application->User: Show table results