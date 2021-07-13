# Connect service to CLARIAH/CLARIN authentication

This article is mostly useful for CLARIAH application developers who would like to provide a login mechanism that enable members of the CLARIAH/CLARIN community to access their application or service.

## Some context

The CLARIAH community, comprised of mainly acedemic institutes and universities, shares a common Identity Provider (IdP), required for user authentication, via the CLARIN organisation. This way CLARIAH can enable login functionality for CLARIAH services

This CLARIN IdP is connected to the CLARIAH authentication system (SATOSA) and enables CLARIAH services, such as LaMachine and the [Media Suite](https://mediasuite.clariah.nl) to provide a login functionality for (all) members/organisations connected via the CLARIN IdP. (TODO a bit terrible phrasing, fix later)


# How to connect your service

The protocol used to communicate with the CLARIAH authentication is [OpenID Connect](https://openid.net/developers/specs/) or in short: OIDC.

More specifically: currently all communication with the authentication server must follow the [Authorization Code Flow](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth)

## Step 1: Register your service domain name

Before being able to implement the desired OIDC communication flow, it is necessary to register a domain the authentication server can access on the web. It's probably best to setup two domains: one for testing and one for production.

Next to the domain it is also necessary to define a redirect URI (for each domain) that the authentication server can connect with, e.g.

```
https://[YOUR_SERVICE_HOST]/oidc/redirect
```

Contact the [Humanties Cluster](https://huc.knaw.nl/) to obtain the necessary `client ID` and `client secret` for each of these domains.

## Step 2: Implement the Authorization Code Flow

The code below shows how the flow is implemented in the Media Suite (in Python)

### Request a code

```
def request_code(self):
    params = {
        'client_id' : '[OIDC CLIENT ID]',
        'redirect_uri' : 'https://[SERVICE_HOST]/oidc/redirect',
        'scope' : 'openid profile email',
        'response_type' : 'code',
        'claims' : json.dumps(
            {
                'userinfo' : {
                    'edupersontargetedid': None,
                    'schac_home_organisation': None,
                    'nickname': None,
                    'email': None,
                    'eppn': None,
                    'idp' : None
                }
            }
        )
    }
    url = '{}?{}'.format('https://[CLARIAH AUTH HOST]/Saml2/OIDC/authorization', urlencode(params))
    return redirect(url)
```

Note that by providing the `claims` this way, you're asking the auth server to give you this information back when receiving/requesting user information.

### When receiving the code (via redirect_uri)

```
"""
{
    'access_token': '5949566e577445ac9f3cf3ac689971f1',
    'expires_in': 3600,
    'id_token': '[A BIG ENCRYPTED STRING]',
    'token_type': 'Bearer'
}
"""
def redirect_uri(self):
    if request.args:
        if request.args.get('error', None):
            #see https://www.oauth.com/oauth2-servers/authorization/the-authorization-response/
            errorMsg = request.args.get('error_message', None)
            errorDesc = request.args.get('error_description', None)
            self.logger.warning('SATOSA (via OIDC) returned error_message: %s' % errorMsg)
            self.logger.warning('SATOSA (via OIDC) returned error_description: %s' % errorDesc)
            self.logger.warning(request.args.get('error'))
            return redirect(url_for(
                'login_failed',
                error='The login server returned an error preventing a successful login'
            ))
        # grab the code from the request
        code = request.args.get('code')
        self.logger.info('OIDC state %s' % request.args.get('state'))

        # now user the code to fetch an access_token
        token = self.request_token(code)
        access_token = token['access_token'] if token and 'access_token' in token else None

        if access_token:

        	#e.g. put the access token in a session
            session['oidcToken'] = access_token

            # now request the user information, so we can use that in the UI of the service
            userInfo = self.request_user_info(access_token)

            # for the media suite only Dutch institutes are allowed, so check the white list
            if self.get_allowed_user_idp(userInfo) != None:

            	# for the CLARIN IdP the MS only allows specific users
                if self.check_idp_user_whitelist(userInfo):

                	# put the user info in the session
                    session['userInfo'] = userInfo
                    session['oidcIsAuthenticated'] = True

                    # redirect the user to the originally requested URL in your service
                    if 'requestedURL' in session:
                        return redirect(session['requestedURL'])
                    else:
                        self.logger.warning('redirecting to home since no requestedURL was found...')
                        return redirect(url_for('home'))
                else:
                    return self.login_failed('You are not allowed to login via CLARIN please ask for dispensation via the CLARIAH project')
            else:
                return self.login_failed('The organisation you are authenticating from is not authorized access to the CLARIAH Media Suite')
        else:
            return self.login_failed('The server failed to obtain an access token')
    return self.login_failed('The server failed to return any authentication data')
```


### Request an access token with the code

```
    def request_token(self, code):
        self.logger.info('requesting token with code %s' % code)
        client_auth = requests.auth.HTTPBasicAuth(
            '[OICD CLIENT ID]',
            '[OICD CLIENT SECRET]'
        )

        post_data = {
            'grant_type': 'authorization_code',
            'code': code,
            'redirect_uri': 'https://[SERVICE_HOST]/oidc/redirect' # this is not called back...
        }

        resp = requests.post(
            'https://[CLARIAH AUTH HOST]/OIDC/token',
            auth=client_auth,
            data=post_data,
            verify=False
        )
        return self.valid_json_response(resp, 'Request token: no valid response')
```

### Request the user info/attributes

```
"""
    {
        "edupersontargetedid": [
            "[SOME HASH]"
        ],
        "email": "someone.xxx.xxx",
        "idp": [
            "https://secure.xxx.nl/hahah"
        ],
        "nickname": "John Doe",
        "schac_home_organisation": [
            "university-x.nl"
        ],
        "sub": "[ANOTHER HASH]"
    }
    """
    def request_user_info(self, access_token):
        headers = {'Authorization': 'Bearer {}'.format(access_token)}
        params = {'scope' : 'openid profile email'}
        resp = requests.get('{}?{}'.format(
        	'https://[CLARIAH AUTH HOST]/OIDC/userinfo',
        	urlencode(params)
        ), headers=headers)
        return self.valid_json_response(resp, 'Request user info: no valid response')
```