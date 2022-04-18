# Sequence Diagrams

### Notebook login with Identity Tokens (access token only)

title Authentication to Notebook with CILogon OAuth flow (OpenID Connect)
Browser->Token Proxy: (via Notebook URL) \nRequest to Login
Token Proxy->Browser: Redirect to login service \n(note: Set CSRF Cookie)
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth \nclaims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Token Proxy: Access Code
Token Proxy->CILogon: Present Access code
CILogon->Token Proxy: OpenID Connect Refresh and \nAccess tokens (Identity tokens)
Token Proxy->Token Proxy: (Authorizer) Validate Token, Set-Cookie \nreissue, check groups
Token Proxy->Notebook: (to Notebook URL) Forward reissued tokens
Notebook->Notebook: Initiate new session via \nJupyterHub JWT authenticator,\nSet JupyterHub Cookie
Notebook->Browser: Authorization okay, Notebook starting up (Cookies Set)
Notebook->Notebook: Spinning up container, stash \nreissued token in User Environment
Browser->Notebook: (Data Access Library) Start interacting with python, \naccess token lasts 24 hours
Notebook->Data Access Library: Initialize data access library
Data Access Library->Data Access Library: Initialize token manager, \nconfigured from environment
Data Access Library->Token Proxy: (via TAP URL) Async TAP request, with Access token
Token Proxy->Token Proxy: Reissue new token \nwith appropriate expiration
Token Proxy->TAP: (to TAP URL) Present token and original request
TAP->TAP: Create TAP job, stash \nreissued token in job
TAP->Data Access Library: Async Job Created
TAP->DB: (Executor) Present \ntoken, execute query
DB->TAP: Query Results
TAP->Token Proxy: (via WebDAV URL) \nPresent token, results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Data Access Library->TAP: Request for Results
TAP->Data Access Library: Redirect to WebDAV
Data Access Library->Token Proxy: (via WebDAV URL) Request for Results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Retrieve File
WebDAV->Data Access Library: Respond with Results
Data Access Library->Browser: Interactive table widget

### Portal login with Identity Tokens (access token only)

title Authentication for Portal with data request, using CILogon and OpenID Connect
Browser->Token Proxy: (via Portal URL) Request to Login
Token Proxy->Browser: Redirect to login service \n(note: Set CSRF Cookie)
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth \nclaims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Token Proxy: Access Code
Token Proxy->CILogon: Present Access code
CILogon->Token Proxy: OpenID Connect Refresh and \nAccess tokens (Identity tokens)
Token Proxy->Token Proxy: (Authorizer) Validate Tokens, Set-Cookie \nreissue, check groups
Token Proxy->Portal: (to Portal URL) Present token
Portal->Portal: Initialize session, \nstash token in user session\n(for Token Manager), \nSet Portal Cookies
Portal->Browser: Authorization okay, Portal initializing (Cookies Set)
Browser->Portal: Activate a cone search
Portal->Token Proxy: (via TAP URL) Async TAP request,\nwith Access token
Token Proxy->Token Proxy: Reissue new token \nwith appropriate expiration
Token Proxy->TAP: (to TAP URL) Present token \nand original request
TAP->TAP: Create TAP job, \nstash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, \nexecute query
DB->TAP: Query Results
TAP->Token Proxy: (via WebDAV URL) \nPresent token, results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Portal->TAP: Request for Results
TAP->Portal: Redirect to WebDAV
Portal->Token Proxy: (via WebDAV URL) Request for Results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Retrieve File
WebDAV->Portal: Respond with Results
Portal->Browser: Interactive Table Results Page

### Application with token download flow (access token only)

title Authentication for Application with data request via token download
User->Browser: Follow link to Token Interface
Browser->Token Proxy: (via token interface URL) \nRequest to authorizer
Token Proxy->Browser: Redirect to login service \n(note: Set CSRF cookie)
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth \nclaims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Token Proxy: Access Code
Token Proxy->CILogon: Present Access code
CILogon->Token Proxy: OpenID Connect Refresh \nand Access tokens (Identity tokens)
Token Proxy->Token Proxy: (Authorizer) Validate Token, Set-Cookie \nreissue, check groups
Token Proxy->Token Interface: (to token interface URL) Present token
Token Interface->Token Interface: Initalize Session,\nSet-Cookie
Token Interface->Browser: Present user with Token Capabilities form (Cookies Set)
Browser->Browser: Present user with Token Capabilities form \n(user selects capabilities)
Browser->Token Proxy: (via Form Submit URL) \nVerify session
Token Proxy->Token Interface: (to Form Submit URL) \nProcess Token Capabilities Form, issue tokens and store in backend,\n store new token information in session for next page visit
Token Interface->Browser: Redirect user to token list page
Browser->Token Proxy: (via Token list Page)\n Verify session
Token Proxy->Token Interface: (to Token list Page)\n Read Token Interface session messages
Token Interface->Token Interface: Build Token Inteface page,\n new token message
Token Interface->Browser: Send token list page with new token info
Browser->User: Show user token information, \n(clipboard copy helper)
User->Application: Enter credentials, as appropriate, in application
Application->Application: Initialize Token Manager
Application->User: Authorization okay, Application ready
User->Application: Perform Cone Search
Application->Token Proxy: (via TAP url) Async TAP request, with capability token
Token Proxy->Token Proxy: Reissue new token with appropriate expiration
Token Proxy->TAP: (to TAP URL) Present token and original request
TAP->TAP: Create TAP job, \nstash reissued token in job
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


### Application with PKCE flow

title Authentication for Application with data request with PKCE flow
User->Application: Initialize Application
Application->Application: Generate a code verifier and challenge
Application->Browser: Redirect to /authorize with developer \nclient_id, code verifier+challenge,\nand callback url
Browser->Token Proxy: (via /authorize URL) \nRequest to authorizer
Token Proxy->Browser: Redirect to login service \n(note: Set CSRF cookie)
Browser->CILogon: Get Login Page (List of IdPs)
CILogon->Browser: Response with Redirect to IdP
Browser->IdP: Login with Credentials (InCommon, Github, etc...)
IdP->Browser: Redirect to CILogon with code/token from IdP
Browser->CILogon: Request with code/token from IdP
CILogon->IdP: Validate code/token
IdP->CILogon: SAML assertions/OAuth \nclaims for User (with email)
CILogon->Browser: Redirect with access code
Browser->Token Proxy: Access Code
Token Proxy->CILogon: Present Access code
CILogon->Token Proxy: OpenID Connect Refresh and \nAccess tokens (Identity tokens)
Token Proxy->Token Proxy: (Authorizer) Validate Token, Set-Cookie \nreissue, check groups
Token Proxy->OAuth2 Authorize: (to /authorize URL) Present token
OAuth2 Authorize->OAuth2 Authorize: Initalize Session,\nSet-Cookie
OAuth2 Authorize->OAuth2 Authorize: Verify client_id+callback url, verifier+challenge
OAuth2 Authorize->Browser: Return Token Capabilities form \n(Cookies Set)
Browser->Browser: Present user with Token Capabilities form \n(user selects capabilities)
Browser->Token Proxy: (via Form Submit URL) \nVerify session
Token Proxy->OAuth2 Authorize: (to Form Submit URL)\nProcess Token Capabilities Form
OAuth2 Authorize->Token Issuer: Issue new tokens with \ncapabilites (with refresh)
Token Issuer->OAuth2 Authorize: Issue new tokens, register in backend, \ncreate access code, return access code
OAuth2 Authorize->Browser: Redirect to application URL with access code
Browser->Application: (Via OS) Follow Application URL
Application->Application: Prepare token request
Application->OAuth2 Authorize: (via /token URL) Request to token endpoint\n(note: /token is not 
protected by token proxy)
OAuth2 Authorize->OAuth2 Authorize: Process access code, \nupdate backend, get tokens
OAuth2 Authorize->Application: Return tokens (refresh and capabilities access token)
Application->Application: Initialize Token Manager
Application->User: Authorization okay, \nApplication initialized
User->Application: Perform Cone Search
Application->Token Proxy: (via TAP url) Async TAP request, with capability token
Token Proxy->Token Proxy: Reissue new token with appropriate expiration
Token Proxy->TAP: (to TAP URL) Present token and original request
TAP->TAP: Create TAP job, \nstash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Return Query Results
TAP->Token Proxy: (via WebDAV URL) \nPresent token, results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Application->TAP: Request for Results
TAP->Application: Redirect to WebDAV
Application->Token Proxy: (via WebDAV URL) Request for Results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Retrieve File
WebDAV->Application: Respond with Results
Application->User: Show table results

### Application with expired access token and valid refresh token

title Authentication for Application with expired access token and valid refresh token
User->Application: Initialize Application
Application->Application: Initialize Token Manager from environment
Application->User: Authorization okay, Application initialized
User->Application: Perform Cone Search
Application->Application: (Token Manager) Access token is expired
Application->OAuth2 Authorize: (via /token URL) \nRequest to token endpoint with \nrefresh token (note: /token is not protected) 
OAuth2 Authorize->OAuth2 Authorize: Verify refresh token is valid
OAuth2 Authorize->OAuth2 Authorize: Generate new Access Token
OAuth2 Authorize->Application: Return refreshed tokens
Application->Token Proxy: (via TAP url) Async TAP request, with capability token
Token Proxy->Token Proxy: Reissue new token with \nappropriate expiration
Token Proxy->TAP: (to TAP URL) Present token and \noriginal request
TAP->TAP: Create TAP job, \nstash reissued token in job
TAP->Portal: Async Job Created
TAP->DB: (Executor) Present token, execute query
DB->TAP: Return Query Results
TAP->Token Proxy: (via WebDAV URL) \nPresent token, results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Store results
WebDAV->TAP: Results Stored
Application->TAP: Request for Results
TAP->Application: Redirect to WebDAV
Application->Token Proxy: (via WebDAV URL) Request for Results
Token Proxy->Token Proxy: Validate Token, check groups
Token Proxy->WebDAV: (to WebDAV URL) Present token, original request
WebDAV->WebDAV: (WebDAV Authenticator) \nImpersonate User
WebDAV->File System: Retrieve File
WebDAV->Application: Respond with Results
Application->User: Show table results
