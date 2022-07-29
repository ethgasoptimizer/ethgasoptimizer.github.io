# Dashboard REST API
Basic dashboard CRUD operations with REST API

| Method| Operation|Description|
| ------ |------ |------ |
|POST|https://api.zmok.io/be/user/login| Authenticate with Web3|
|POST|https://api.zmok.io/be/user/signin| Authenticate with Username/Password|
|GET|https://api.zmok.io/be/app/all| List all available Apps|
|POST|https://api.zmok.io/be/app| Create a new App|
|PUT|https://api.zmok.io/be/app/APP_ID| Update existing App with APP_ID|
|DELETE|https://api.zmok.io/be/app/APP_ID| Remove App with APP_ID|

## Authentification

### Authenticate with Web3

**Request headers:**<br/>
- Content-Type: application/json

**Request payload:**<br/>
- **publicAddress** - a checksummed wallet address<br/>
- **signature** - signed message with wallet signer claims to own<br/><br/> To get valid signature hash, go to: https://docs.zmok.io/eth-message-sign/dist/index.html, connect your wallet and sign requested message<br/><br/>
Click Sign. You should see the resulting signature right afterwards.<br/>

<img src="https://raw.githubusercontent.com/zmok-io/eth-message-sign/master/static/preview.png" width="500">

**Create your access token using your public address and given signature:**

```sh
curl https://api.zmok.io/be/user/login \
 -X POST \
 -H "Content-Type: application/json" \
 -d '{"publicAddress":"0x35bd3a5546a0f785fea24ffbc9fe552014852014","signature":"0x3b978ec9f86b2c9e6635615df14a2bc81ed6cf55c7af7e9b65b18da9d1baf2dc719a593e0f0f85e7780a3c96d3ccb0ea6b5d5412b6415070f448e3b22f0b10c81c"}'

 {"accessToken":"eyKhbFciOiJIUzUxMiJ9.eyJzdWIiOiI1ODRjYzg1Mi0zMDgyLTQwYTEtYjNkNy02MTBjZmU2NjliYjkiLCJpYXQiOjE2NTg5MTA1OTgsImV4cCI6MTY1ODkyMDU5OH0.me4XKb3yRa6n8--NThhVSXZ8EhKNMTLfAq1K-jDypcmZP28RhouvH56CPtQmqIvR3GYSZCvH_CyJqRYn4Xt05A","tokenType":"Bearer","userName":"","publicAddress":"0x35bd3a5546a0f785fea24ffbc9fe552014852014","userAvatar":"","userId":"582cc852-2082-40a1-b3d7-610cfe669bb9","userCreated":1614319732381,"userEmail":"","wallet":true}
```

### Authenticate with Username/Password
**Request headers:**<br/>
- Content-Type: application/json

**Request payload:**<br/>
- **username** - username used when user sign-in to dashboard<br/>
- **password** - password in plain text

**Example:**

```sh
curl https://api.zmok.io/be/user/signin \
-X POST \
-H "Content-Type: application/json" \
-d '{"username":"johndoe","pass":"passInPlainText123"}'
```

## List all available Apps
**Request headers:**<br/>
- Content-Type: application/json
- Authorization: Bearer ACCESS_TOKEN

**Example:**
```sh
curl https://api.zmok.io/be/app/all \
-X GET \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyKhbFciOiJIUzUxMiJ9.eyJzdWIiOiI1ODRjYzg1Mi0zMDgyLTQwYTEtYjNkNy02MTBjZmU2NjliYjkiLCJpYXQiOjE2NTg5MTA1OTgsImV4cCI6MTY1ODkyMDU5OH0.me4XKb3yRa6n8--NThhVSXZ8EhKNMTLfAq1K-jDypcmZP28RhouvH56CPtQmqIvR3GYSZCvH_CyJqRYn4Xt05A"
```

## Create a new App
**Request headers:**<br/>
- Content-Type: application/json
- Authorization: Bearer ACCESS_TOKEN

**Example:**
```sh
curl https://api.zmok.io/be/app \
-X POST \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJjN2FlODI4OS03NTlmLTQ4ZTAtYmI3Ni1jMzc0NjRmNTczMTUiLCJpYXQiOjE2NTkwNDEwMjUsImV4cCI6MTY1OTA1MTAyNX0.mkVGcS9jb3ss4zGOh0SQ6yiGGlpcuzIG5xBFOrUj5_Urrafy51aL7OR5dsNSos1jJoYrGJ6njWtUI4K_U59Fbg"
-d '{"key": "","name":"Built using API","userId":"c7ae8289-759f-48e0-bb76-c37464f57315","description":"The API was used","upstream":"eth","reqPerSecLimit":"10","reqPerMonthLimit":"2000","createdAt":-1,"updatedAt":-1}'
```

Upstream is an alias for the Ethereum/ZMOK network, use it as follows:

|Upstream|Network|
|------ |------ |
|eth| Mainnet|
|fr| Mainnet Front-running|
|archive| Mainnet Archive|
|rinkeby| Testnet - Rinkeby|
|ropsten| Testnet - Ropsten|


## Update existing App
**Request headers:**<br/>
- Content-Type: application/json
- Authorization: Bearer ACCESS_TOKEN

**Example:**
```sh
curl https://api.zmok.io/be/app/serzibbxzik0szc8 \
-X PUT \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJjN2FlODI4OS03NTlmLTQ4ZTAtYmI3Ni1jMzc0NjRmNTczMTUiLCJpYXQiOjE2NTkwNDEwMjUsImV4cCI6MTY1OTA1MTAyNX0.mkVGcS9jb3ss4zGOh0SQ6yiGGlpcuzIG5xBFOrUj5_Urrafy51aL7OR5dsNSos1jJoYrGJ6njWtUI4K_U59Fbg" \
-d '{"key": "serzibbxzik0szc8","name":"Test App3","userId":"c7ae8289-759f-48e0-bb76-c37464f57315","description":"app updated","upstream":"fr","reqPerSecLimit":"5","reqPerMonthLimit":"400","createdAt":-1,"updatedAt":-1}'
```

## Remove App
**Request headers:**<br/>
- Content-Type: application/json
- Authorization: Bearer ACCESS_TOKEN

**Example:**
```sh
curl https://api.zmok.io/be/app/fcta8llfbaccsnu4 \
-X DELETE \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyKhbFciOiJIUzUxMiJ9.eyJzdWIiOiI1ODRjYzg1Mi0zMDgyLTQwYTEtYjNkNy02MTBjZmU2NjliYjkiLCJpYXQiOjE2NTg5MTA1OTgsImV4cCI6MTY1ODkyMDU5OH0.me4XKb3yRa6n8--NThhVSXZ8EhKNMTLfAq1K-jDypcmZP28RhouvH56CPtQmqIvR3GYSZCvH_CyJqRYn4Xt05A"
```
