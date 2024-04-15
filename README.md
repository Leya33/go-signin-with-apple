Go Sign In With Apple
======

![](https://img.shields.io/badge/golang-1.22-blue.svg?style=flat) [![codecov](https://codecov.io/gh/Timothylock/go-signin-with-apple/branch/master/graph/badge.svg)](https://codecov.io/gh/Timothylock/go-signin-with-apple)
 [![Build Status](https://app.travis-ci.com/Timothylock/go-signin-with-apple.svg?branch=master)](https://app.travis-ci.com/github/Timothylock/go-signin-with-apple) [![Codacy Badge](https://api.codacy.com/project/badge/Grade/b54cafe3d1884d9cbe9748839739265e)](https://www.codacy.com/manual/Timothylock/go-signin-with-apple?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Timothylock/go-signin-with-apple&amp;utm_campaign=Badge_Grade)

A library for validating and revoking Apple Sign In tokens generated by either the web or iOS app. 


## Installation
```
go get github.com/Timothylock/go-signin-with-apple
import "github.com/Timothylock/go-signin-with-apple/apple"

```

## Usage

There are several example files based on your particular use case which can be found below:
- [Validating an app token](example/app_validation_example_test.go)
- [Validating a web token](example/web_validation_example_test.go)
- [Validating a refresh token](example/refresh_validation_example_test.go)
- [Revoking an access token](example/revoke_access_token_example_test.go)
- [Revoking a refresh token](example/revoke_refresh_token_example_test.go)

### Example
While it is recommended to look at the specific example file, here is validating an app token:

``` golang
import "github.com/Timothylock/go-signin-with-apple/apple"

...

// Generate the client secret used to authenticate with Apple's validation servers
// Refer to the example files to see where to get secret, teamID, clientID, keyID
secret, _ := apple.GenerateClientSecret(secret, teamID, clientID, keyID)

// Generate a new validation client
client := apple.New()

vReq := apple.AppValidationTokenRequest{
	ClientID:     clientID,
	ClientSecret: secret,
	Code:         "the_authorization_code_to_validate",
}

var resp apple.ValidationResponse

// Do the verification
client.VerifyAppToken(context.Background(), vReq, &resp)

unique, _ := apple.GetUniqueID(resp.IDToken)

// Voila!
fmt.Println(unique)
```

### Generating Client Secret
Apple requires a JWT token along with your validation request to authenticate your request. A token can be generated by 
calling the `GenerateClientSecret` function included. Check [secret.go](apple/secret.go) to see exactly how to obtain the 
parameters required by the function. Note that your account might not have permissions to view/create `service IDs` and 
`keys` required by this function. 

```
import "github.com/Timothylock/go-signin-with-apple/apple"

...

// Your 10-character Team ID
team_id := "XXXXXXXXXX"

// Your Services ID, e.g. com.aaronparecki.services
client_id := "come.change.me"

// Find the 10-char Key ID value from the portal
key_id := "XXXXXXXXXX"

secret := `Your key that starts with -----BEGIN PRIVATE KEY-----`

secret, _ := apples.GenerateClientSecret(secret, team_id, client_id, key_id)
fmt.Println(secret)
```

### Validating Token
To validate a token, you must create a new validation `Client` then call the respective `Verify` function.

```
import "github.com/Timothylock/go-signin-with-apple/apple"

...

// Generate a new validation client
client := apple.New()

vReq := apple.AppValidationTokenRequest{
	ClientID:     clientID,
	ClientSecret: secret,
	Code:         "the_authorization_code_to_validate",
}

var resp apple.ValidationResponse

// Do the verification
client.VerifyAppToken(context.Background(), vReq, &resp)

```

### Obtaining Unique Subject ID
A subject ID is included in the `id_token` field of the response which when decoded, has a subject that can 
uniquely identify the user. A helper function is included to obtain this subject ID: `GetUniqueID`

```
import "github.com/Timothylock/go-signin-with-apple/apple"

... Code to validate token ...

reflect.TypeOf(response)         // ValidationResponse
reflect.TypeOf(response.IdToken) // String


id := apple.GetUniqueID(response.IdToken)
fmt.Println(id)
```

### Obtaining Email
Apple recently added support for the including information about the user in their response. As of right now, you have 
access to the following: 
- email
- email_verified - whether or not the user has validated their email with Apple
- private_email - whether or not the email is a private relay email from Apple

```$xslt
import "github.com/Timothylock/go-signin-with-apple/apple"

... Code to validate token ...

reflect.TypeOf(response)         // ValidationResponse
reflect.TypeOf(response.IdToken) // String


claim, _ := apple.GetClaims(resp.IDToken)

email := (*claim)["email"]
emailVerified := (*claim)["email_verified"]
isPrivateEmail := (*claim)["is_private_email"]
```

### Obtaining Real User Status
Apple determines whether a user is a real person by combining multiple metrics on iOS 14 devices and above. This value is 
assessible by reading `real_user_status` from the claims. More documentation [here](https://developer.apple.com/documentation/authenticationservices/asuserdetectionstatus)

```$xslt
import "github.com/Timothylock/go-signin-with-apple/apple"

... Code to validate token ...

reflect.TypeOf(response)         // ValidationResponse
reflect.TypeOf(response.IdToken) // String


claim, _ := apple.GetClaims(resp.IDToken)

realUserStatus := (*claim)["real_user_status"]
```

## Contributing
Make sure tests pass, submit a PR, and lets get going! 

## License
go-signin-with-apple is licensed under the MIT.
