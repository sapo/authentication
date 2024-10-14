# SAPO Authentication
Help to integrate SAPO authentication into any application.

# Python OAuth 2.0 / OpenID Connect Examples | SAPO ID CONNECT

## Libraries Needed

To implement this SAPO OAuth 2.0 / OpenID Connect example, you'll need the following libraries:

Python Libraries Used:
  - requests
  - oauthlib
  - base64
  - os
  - urllib

Allowed Grant Types:
- **Hybrid Flow**
- **Authorization Code Flow**
- **Implicit Flow**

## 1. Hybrid Flow

The Hybrid flow combines aspects of the Authorization Code flow and Implicit flow.

### Set up client

```python
CONSUMER_KEY = "YOUR_CONSUMER_KEY"
CONSUMER_SECRET = "YOUR_CONSUMER_SECRET"
AUTHORIZATION_URL = "https://id.sapo.pt/oauth/v2/authorize"
ACCESS_TOKEN_URL = "https://id.sapo.pt/oauth/v2/token"
USERINFO_URL = "https://id.sapo.pt/userinfo/"
LOGOUT_URL = "https://id.sapo.pt/oauth/v2/logout"

client = WebApplicationClient(CONSUMER_KEY)
code_verifier = client.create_code_verifier(64)
nonce = base64.urlsafe_b64encode(os.urandom(32)).decode('utf-8').rstrip('=')

auth_request = client.prepare_request_uri(
    AUTHORIZATION_URL,
    redirect_uri="https://example.com",
    scope=["openid", "email", "profile"],
    code_challenge=client.create_code_challenge(code_verifier, "S256"),
    code_challenge_method="S256",
    nonce=nonce
)

auth_request = auth_request.replace("response_type=code", "response_type=code%20id_token%20token")

print(f"Please visit this URL to authorize: {auth_request}")
```

### After user authorization, handle the callback

```python
uri = input("Copy and paste the callback URL from the browser: ")
uri = uri.replace("/#", "/?")
parsed_uri = urlparse(uri)
params = parse_qs(parsed_uri.fragment or parsed_uri.query)

url, headers, body = client.prepare_token_request(
    ACCESS_TOKEN_URL, uri, code_verifier=code_verifier
)
b64string = base64.b64encode(f"{CONSUMER_KEY}:{CONSUMER_SECRET}".encode("utf8")).decode("utf8")
headers["Authorization"] = f"Basic {b64string}"

resp = requests.post(url, data=body.encode("utf8"), headers=headers, timeout=30, verify=False)
json_resp = resp.json()

id_token = json_resp['id_token']
```

### Refresh token

```python
refresh_url, refresh_headers, refresh_body = client.prepare_refresh_token_request(
    ACCESS_TOKEN_URL, json_resp["refresh_token"]
)

refresh_response = requests.post(refresh_url, headers=refresh_headers, data=refresh_body.encode("utf8"), timeout=30, verify=False)

print(f"\nRefresh Token Response:\n{refresh_response.json()}")
```

### Logout

```python
params = {
    'id_token_hint': id_token,
    'client_id': CONSUMER_KEY,
    'post_logout_redirect_uri': "https://example.com",
}

logout_url_with_params = f"{LOGOUT_URL}?{urlencode(params)}"

response = requests.post(logout_url_with_params, headers={'Content-Type': 'application/x-www-form-urlencoded'}, timeout=30, verify=False)

if response.status_code == 200:
    print("Logout successful.")
else:
    print(f"Failed to log out. Status code: {response.status_code}")
```

## 2. Authorization Code Flow

The Authorization Code flow is used to obtain an access token and refresh token.

### Set up client

```python
CONSUMER_KEY = "YOUR_CONSUMER_KEY"
CONSUMER_SECRET = "YOUR_CONSUMER_SECRET"
AUTHORIZATION_URL = "https://id.sapo.pt/oauth/v2/authorize"
ACCESS_TOKEN_URL = "https://id.sapo.pt/oauth/v2/token"
LOGOUT_URL = "https://id.sapo.pt/oauth/v2/logout"
USERINFO_URL = "https://id.sapo.pt/userinfo/"

client = WebApplicationClient(CONSUMER_KEY)
code_verifier = client.create_code_verifier(64)
nonce = base64.urlsafe_b64encode(os.urandom(32)).decode('utf-8').rstrip('=')

auth_request = client.prepare_request_uri(
    AUTHORIZATION_URL,
    redirect_uri="https://example.com",
    scope=["openid", "email", "profile"],
    code_challenge=client.create_code_challenge(code_verifier, "S256"),
    code_challenge_method="S256",
    nonce=nonce
)

print(f"Please visit this URL to authorize: {auth_request}")
```

### After user authorization, handle the callback

```python
uri = input("Copy and paste the callback URL from the browser: ")
parsed_uri = urlparse(uri)
params = parse_qs(parsed_uri.fragment or parsed_uri.query)
code = params.get('code', [None])[0]

url, headers, body = client.prepare_token_request(
    ACCESS_TOKEN_URL, uri, code_verifier=code_verifier
)
b64string = base64.b64encode(f"{CONSUMER_KEY}:{CONSUMER_SECRET}".encode("utf8")).decode("utf8")
headers["Authorization"] = f"Basic {b64string}"

resp = requests.post(url, data=body.encode("utf8"), headers=headers, timeout=30, verify=False)
json_resp = resp.json()

id_token = json_resp['id_token']
access_token = json_resp['access_token']
```

### Get user info

```python
userinfo_response = requests.get(
    USERINFO_URL,
    headers={"Authorization": f"Bearer {access_token}"},
    timeout=30,
    verify=False
)

print(f"\nUser Info:\n{userinfo_response.json()}")
```

### Refresh token

```python
refresh_url, refresh_headers, refresh_body = client.prepare_refresh_token_request(
    ACCESS_TOKEN_URL, json_resp["refresh_token"]
)

refresh_response = requests.post(refresh_url, headers=refresh_headers, data=refresh_body.encode("utf8"), timeout=30, verify=False)

print(f"\nRefresh Token Response:\n{refresh_response.json()}")
```

### Logout

```python
params = {
    'id_token_hint': id_token,
    'client_id': CONSUMER_KEY,
    'post_logout_redirect_uri': "https://example.com",
}

logout_url_with_params = f"{LOGOUT_URL}?{urlencode(params)}"

response = requests.post(logout_url_with_params, headers={'Content-Type': 'application/x-www-form-urlencoded'}, timeout=30, verify=False)

if response.status_code == 200:
    print("Logout successful.")
else:
    print(f"Failed to log out. Status code: {response.status_code}")
```

## 3. Implicit Flow

The Implicit flow is used to obtain an access token directly.

### Set up client

```python
CONSUMER_KEY = "YOUR_CONSUMER_KEY"
CONSUMER_SECRET = "YOUR_CONSUMER_SECRET"
AUTHORIZATION_URL = "https://id.sapo.pt/oauth/v2/authorize"
ACCESS_TOKEN_URL = "https://id.sapo.pt/oauth/v2/token"
LOGOUT_URL = "https://id.sapo.pt/oauth/v2/logout"
USERINFO_URL = "https://id.sapo.pt/userinfo/"

client = WebApplicationClient(CONSUMER_KEY)
code_verifier = client.create_code_verifier(64)
nonce = base64.urlsafe_b64encode(os.urandom(32)).decode('utf-8').rstrip('=')

auth_request = client.prepare_request_uri(
    AUTHORIZATION_URL,
    redirect_uri="https://example.com",
    scope=["openid", "email", "profile"],
    code_challenge=client.create_code_challenge(code_verifier, "S256"),
    code_challenge_method="S256",
    nonce=nonce
)

auth_request = auth_request.replace("response_type=code", "response_type=id_token%20token")

print(f"Please visit this URL to authorize: {auth_request}")
```

### After user authorization, handle the callback

```python
uri = input("Copy and paste the callback URL from the browser: ")
#uri = uri.replace("/#", "/?")

parsed_uri = urlparse(uri)
params = parse_qs(parsed_uri.fragment or parsed_uri.query)

id_token = params.get('id_token', [None])[0]
access_token = params.get('access_token', [None])[0]
```

### Get user info

```python
userinfo_response = requests.get(
    USERINFO_URL,
    headers={"Authorization": f"Bearer {access_token}"},
    timeout=30,
    verify=False
)

print(f"\nUser Info:\n{userinfo_response.json()}")
```

### Logout

```python
params = {
    'id_token_hint': id_token,
    'client_id': CONSUMER_KEY,
    'post_logout_redirect_uri': "https://example.com",
}

logout_url_with_params = f"{LOGOUT_URL}?{urlencode(params)}"

response = requests.post(logout_url_with_params, headers={'Content-Type': 'application/x-www-form-urlencoded'}, timeout=30, verify=False)

if response.status_code == 200:
    print("Logout successful.")
else:
    print(f"Failed to log out. Status code: {response.status_code}")
```
```
