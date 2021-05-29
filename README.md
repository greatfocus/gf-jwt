# jwt
This is a minimal implementation of JWT designed with simplicity in mind.

# What is JWT?
Jwt is a signed JSON object used for claims based authentication. A good resource on what JWT Tokens are is [jwt.io](https://jwt.io/), and in addition you can always read the [RFC](https://tools.ietf.org/html/rfc7519).

This implementation doesn't fully follow the specs in that it ignores the algorithm claim on the header. It does this due to the security vulnerability in the JWT specs. Details on the vulnerability can be found [here](https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/)

## What algorithms does it support?
* HS256
* HS384
* HS512

## How does it work?

### How to create a token?
Creating a token is actually pretty easy.

The first step is to pick a signing method. For demonstration purposes we will choose HSMAC256.

    algorithm :=  jwt.HmacSha256("ThisIsTheSecret")
   
Now we need to the claims, and edit some values

    claims := jwt.NewClaim()
    claims.Set("Role", "Admin")
    
Then we will need to sign it!

    token, err := algorithm.Encode(claims)
    if err != nil {
        panic(err)    
    }
    
### How to authenticate a token?
Authenticating a token is quite simple. All we need to do is...

    if algorithm.Validate(token) == nil {
        //authenticated
    } 
    
### How do we get the claims?
The claims are stored in a `Claims` struct. To get the claims from a token simply...
    
    claims, err := algorithm.Decode(token)
    if err != nil {
        panic(err)
    }
    _, role := claims.Get("Role")
    if strings.Compare(role, "Admin") {
        //user is an admin    
    }
    
### Example

    package main

    import (
        "fmt"
        "strings"
        "time"

        "github.com/robbert229/jwt"
    )

    func main() {
        secret := "ThisIsMySuperSecret"
        algorithm := jwt.HmacSha256(secret)

        claims := jwt.NewClaim()
        claims.Set("Role", "Admin")
        claims.SetTime("exp", time.Now().Add(time.Minute))

        token, err := algorithm.Encode(claims)
        if err != nil {
            panic(err)
        }

        fmt.Printf("Token: %s\n", token)

        if algorithm.Validate(token) != nil {
            panic(err)
        }

        loadedClaims, err := algorithm.Decode(token)
        if err != nil {
            panic(err)
        }

        role, err := loadedClaims.Get("Role")
        if err != nil {
            panic(err)
        }

        roleString, ok := role.(string)
        if !ok {
            panic(err)
        }

        if strings.Compare(roleString, "Admin") == 0 {
            //user is an admin
            fmt.Println("User is an admin")
        }
    }
    
### How is it different from golang.org/x/oauth2/jwt?
This package contains just the logic for `jwt` encoding, decoding, and verification. The `golang.org/x/oauth2/jwt` package does not implement the `jwt` specifications. Instead it is specifically tied to `oauth2`, requiring a `TokenURL`, `email` and can only use a specific algorithm (RSA).
