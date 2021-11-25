## Process followed to Setup Accountingwizards On Model Bank (TPP/Open Banking)

# Steps
1.  On Open Banking Environment - Production Sandbox 
2.  Add new Software Statement (softwareId : YYLeCDhNlzUDLpsbG3oDsY) <-----------------A
     with the name Chillibooks live test
3.  Associate OB WAC Organisation level transport certificate 
4.  Create Signing key (key id : ywtfoy-Dw3tjSYJ0K5c2ZRkgRr0) <-----------------B
    Used follwing as suggested in "Add certificates"
    openssl genrsa -out keys-sigkey-YYLeCDhNlzUDLpsbG3oDsY.pem 2048 && openssl rsa -in keys-sigkey-YYLeCDhNlzUDLpsbG3oDsY.pem -pubout -out publickey-sigkey-YYLeCDhNlzUDLpsbG3oDsY.key
    the private key generated with file name: keys-sigkey-YYLeCDhNlzUDLpsbG3oDsY.pem used below <----------------- C

5.  Generate Sogtware statement and copied into <-----------------------D below
6.  Following "Integrating a TPP with Ozone Model Banks Using Postman on Directory Sandbox"
    https://openbanking.atlassian.net/servicedesk/customer/portal/1/article/313918598
    
    1.  "3.1.1 Example Registration Request JWT (Python) I am using python3.9 jwcrypto 1.0
        ----            
        # The software statement ID (software_id) of the software statement created in software statements (MIT).
        SOFTWARE_STATEMENT_ID = "" <-----------------A see above
        #  Value of the kid parameter associated with the signing certificate generated in Generate a
        # transport/signing certificate pair (please note that you need to use the signing certificate kid).
        KID = "" <-----------------B see above
        # Your private signing key. You will use this to sign your JWT.
        PRIVATE_RSA_KEY = """
        -----BEGIN PRIVATE KEY-----
        see above <----------------- C
        -----END PRIVATE KEY-----
        """
        
        SOFTWARE_STATEMENT = "" <-----------------------D see above
        
        def make_registration_jwt(software_statement_id: str, kid: str, software_statement: str) -> str:
            jwt_iat = int(time.time())
            jwt_exp = jwt_iat + 3600
            header = dict(alg='RS256', kid=kid, typ='JWT')
            claims = dict(
                token_endpoint_auth_signing_alg="PS256",
                grant_types=["authorization_code", "client_credentials"],
                subject_type="public",
                application_type="web",
                iss=software_statement_id,
                redirect_uris=["https://app.getpostman.com/oauth2/callback"],
                token_endpoint_auth_method="client_secret_basic",
                aud="0015800001041RHAAY",
                scope= "openid accounts payments",   #accounts for AISP or payments for PISP or both
                request_object_signing_alg="none",
                exp=jwt_exp,
                iat=jwt_iat,
                jti=str(uuid.uuid4()),
                response_types=["code", "code id_token"],
                id_token_signed_response_alg="RS256",
                software_statement=software_statement
            )
        
            token = jwt.JWT(header=header, claims=claims)
            key_obj = jwk.JWK.from_pem(PRIVATE_RSA_KEY.encode('latin-1'))
            token.make_signed_token(key_obj)
            signed_token = token.serialize()
        
            print(signed_token)
            return signed_token
        
        make_registration_jwt(SOFTWARE_STATEMENT_ID, KID, SOFTWARE_STATEMENT)
        <----------------------- Result input below > <----------------- E
    2.  Use result returned from the above request in "3.1.2 Request Registration Endpoint Example (Python): "
        headers = {'Content-Type': 'application/jwt'}
            client = ('./transport.pem', './transports.key')
            response = requests.post("https://ob19-rs1.o3bank.co.uk:4501/dynamic-client-registration/v3.1/register",
                                    registration_request, <----------------- E see above
                                    headers=headers,
                                    verify=False,
                                    cert=client
                                    )
            print(response.content)

               
## Following the above I get the following error when when running the request above
ssl.SSLError: [SSL] PEM lib (_ssl.c:4044)

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "C:\Python39\lib\site-packages\requests\adapters.py", line 439, in send
    resp = conn.urlopen(
  File "C:\Python39\lib\site-packages\urllib3\connectionpool.py", line 755, in urlopen
    retries = retries.increment(
  File "C:\Python39\lib\site-packages\urllib3\util\retry.py", line 574, in increment
    raise MaxRetryError(_pool, url, error or ResponseError(cause))
urllib3.exceptions.MaxRetryError: HTTPSConnectionPool(host='ob19-rs1.o3bank.co.uk', port=4501): Max retries exceeded with url: /dynamic-client-registration/v3.1/register (Caused by SSLError(SSLError(9, '[SSL] PEM lib (_ssl.c:4044)')))

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "C:\Projects\OpenBanking\Ozone\getclientId.py", line 6, in <module>
    response = requests.post("https://ob19-rs1.o3bank.co.uk:4501/dynamic-client-registration/v3.1/register",
  File "C:\Python39\lib\site-packages\requests\api.py", line 117, in post
    return request('post', url, data=data, json=json, **kwargs)
  File "C:\Python39\lib\site-packages\requests\api.py", line 61, in request
    return session.request(method=method, url=url, **kwargs)
  File "C:\Python39\lib\site-packages\requests\sessions.py", line 542, in request
    resp = self.send(prep, **send_kwargs)
  File "C:\Python39\lib\site-packages\requests\sessions.py", line 655, in send
    r = adapter.send(request, **kwargs)
  File "C:\Python39\lib\site-packages\requests\adapters.py", line 514, in send
    raise SSLError(e, request=request)
requests.exceptions.SSLError: HTTPSConnectionPool(host='ob19-rs1.o3bank.co.uk', port=4501): Max retries exceeded with url: /dynamic-client-registration/v3.1/register (Caused by SSLError(SSLError(9, '[SSL] PEM lib (_ssl.c:4044)')))


    


