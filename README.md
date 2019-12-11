# bunq PSD2 example

PSD2 private key: See contents of **key.pem**  
PSD2 public certificate: See contents of **cert.pem**  
PSD2 public certificate chain: See contents of **cert.pem**  

Client public key: See contents of **client_pub_key.pem**  
Client private key: See contents of **client_key.pem**  
 
Installation token (obtained via ```/v1/installation``` endpoint):  
```7aad5c4e145206d1e694e0e819d6ff51db4e900d25dad2a03575ce07cd84669a```

String to sign (client public key + installation token). **StringToSign.txt**:  
```
-----BEGIN PUBLIC KEY-----  
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAmOfPBFF6P9E+V6b35+cl  
Z3PbWfkMu/Eo2kuLA3If12BBNIfmiNSYGpZdw3YanQbZ9y0io6nKG9dxPdRgOvQ2  
zPNOjSI4PSNA5UTVIdGpbcsFnLYvaedJh3ZZ+SUSXUOIVBDfhZY1cDAwIcuMZxpg  
x4Wd9zHX057275XSOU0LNShVtzenXycoSe6IeI0bgBNhE1M6k1Tr6BJZmMR1+EHm  
WTc8PF34hubvEMuBPmuQO955fmjdGSUrOOXcxsj+I1t/H2fnVVVJ1XO8d6vxMKiU  
hhzQ53CS6QHxwnsH7DF07Sb5gUobvvOotqnQDC9jKTVK78lOdqtM7KeqbVhLeOBl  
EQIDAQAB  
-----END PUBLIC KEY-----  
7aad5c4e145206d1e694e0e819d6ff51db4e900d25dad2a03575ce07cd84669a
```

Create signature:  
```openssl dgst -sha256 -sign key.pem -out JsonBodySignature.txt StringToSign.txt```  
```openssl enc -base64 -in JsonBodySignature.txt -out JsonBodySignatureEncoded.txt```

Signature:  
```
zV9vWMZRl8mblmr9nMQrkL31ySk7bhf2ecIhq9Z2wrPnO4QdDX0o4lR7ociJ6vvusrYMAA//RhzZQm3fWi8rYsdh/uMa/4kV2+iPM+GXmbfmLA5TIt+CD2c/D2jvkVhp7tJ7SJ7S2NficZrLE0SniRmjdtZzvrLDFlbSMP3wDjXTvQt+xFJ30fy4G4sK5GtXEBgai69n59O7ip2BY5WzvNCPAJL3KRkPSuvVhe0s4HNZg5J6CS37n67ggd0lGL+EUcnlGuS13uT8D1OGUOQM1ZlzfL1uN7M65LCW6WUtsjp5w3meihgnC+lEKFRw/49RpMlDkOEoqKCZAtVTUp/XpsIK4pxlD0IxRmMXDlkTf19sLkZAVnKDL1I3w8H7sHMshdFdlHgUyhnNBflnGtEbW7LeFQN17f1M2u5R5Xi9SbiP+oedflTG3q8ypYG3C651ryTXGCsI5+zhrrS8u0DrnQj4n+Ty0xhkDIVIOzjDNJS9LMVDpFxSUreXD+VWgQFHCog1hX3B3w6gSlMYFII7L0LZebir6VKatIu/CNs0Drby3zKmnPn9bFWgjGvNbzgyhQ9A1wgo2FG7EI7UKlJSBgiJy5UcoA/M7tK94vW/Y8eQuEgscBp0XsnBGt09UfgFOKiN3moXOuwaFmSiXZ8ZnAiTdqtFnVdb5+yugPnGVeI=
```

HTTP request data to sign (Create value for **X-Bunq-Client-Signature** header): See contents of **HttpRequest.txt**

Create signature (X-Bunq-Client-Signature header value):  
```openssl dgst -sha256 -sign client_key.pem -out HeaderSignature.txt HttpRequest.txt```  
```openssl enc -base64 -in HeaderSignature.txt -out HeaderSignatureEncoded.txt```  
