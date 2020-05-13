# bunq PSD2 example, get PSP credential

**Tested in the Sandbox environment.**

PSD2 private key: See contents of **key.pem**  
PSD2 public certificate: See contents of **cert.cert**  
PSD2 public certificate chain: See contents of **cert.cert**  

Client public key: See contents of **client_pub_key.pem**  
Client private key: See contents of **client_key.pem**  
 
Installation token (obtained via ```/v1/installation``` endpoint):  
```ff97433160e538aef186b5a543b121ec7c7c6d7e2116137a081ffe0fa8c82aa8```

String to sign (client public key + installation token). **StringToSign.txt**:  
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsS0F3O//51rkOEO49he/
gyCIQm96fFQMVfYWesHRoUhX8uVgKivjNF/XIjJrE13n0f+fSuMdguUrO0SPM1aA
1Yg7xylLCXqhutRrt1vks656eWBgrBPM+j1vUpmFgkNnl+wacZxb1E5JcDmOYJeU
vwI9B2iIn3EihlrzJSxrzQNpnpy+n4UC3l0IQ8sLu1Ue4KcKaWwXEMfmAYOLSume
s2accllGkWLbTQFkPdXsmft7fDtEsSa3I18F7JNlSDlGD8joj+IWJAZ/ujv47omY
1+nmmJoNq+OxG96pdhMDHLBSPqUoRIidxgXt1k+klmWyfTH5RtzbELl8L0RNyDIL
CwIDAQAB
-----END PUBLIC KEY-----
ff97433160e538aef186b5a543b121ec7c7c6d7e2116137a081ffe0fa8c82aa8
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
