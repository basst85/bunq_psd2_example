# bunq PSD2 example, get PSP credential

**Tested in the Sandbox environment.**

PSD2 private key: See contents of **key.pem**  
PSD2 public certificate: See contents of **cert.cert**  
PSD2 public certificate chain: See contents of **cert.cert**  

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
ITii4Kn2W/yJFnsNmC5vlumpq2NB9Nn51u+oowjGBuaY0z7F9B8UCf6c/Jzn75ZroHpv1oAGaNo5q7hrfU3Ev2vUNWKCvIJ6+OZ4JMAj8I67m9+6xKHM2KEQU9KNaJUYowsE2JQ7HwKUvXGMLCCX68Xv+jUTn1SNGai1mLYMMuSxirl0QVxfsQrs/Q0Z9sIJlWI+B0QganmfTYjelTO1+xQvtJQLXIZe3xQfsVIa0Xg6+UbRtyYlQUiQjq52kvVt/GuUcpGEZdwFdHtool0eo/uxPN5sHHYf/m2++2sJmF7D7pt99c0uL/5d1ef4yUTpvHIFNAsxlkBPpAPT1eyWUA==
```

HTTP request data to sign (Create value for **X-Bunq-Client-Signature** header): See contents of **HttpRequest.txt**

Create signature (X-Bunq-Client-Signature header value):  
```openssl dgst -sha256 -sign client_key.pem -out HeaderSignature.txt HttpRequest.txt```  
```openssl enc -base64 -in HeaderSignature.txt -out HeaderSignatureEncoded.txt```  

## Postman screenshot
![screenshot](https://github.com/basst85/bunq_psd2_example/raw/master/image.png)
