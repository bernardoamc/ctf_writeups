# JWT bruteforcing 
#pentest/tryhackme

Recall that `JWT HS256` is calculated using a secret.The exact format of the calculation is 

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

Therefore, it stands to reason that, since we have the full jwt token, and the header and payload, the secret can be brute forced to obtain the full JWT token. If the secret can be brute forced then the attacker could sign his own JWT tokens.

## Tool
[GitHub - lmammino/jwt-cracker: Simple HS256 JWT token brute force cracker](https://github.com/lmammino/jwt-cracker)

**Explanation of Parameters:**

Token
	The HS256 JWT token
Alphabet
	The alphabet that the cracker will use to check passwords(default: "abcdefghijklmnopqrstuvwxyz")
max-length
	The max expected length of the secret(12 by default)

