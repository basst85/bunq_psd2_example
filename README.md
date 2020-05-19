bunq PSD2 example, get PSP credential via public API  
====

**Only for the API Sandbox environment.  
Use real eIDAS certificate in production**

Create a pseudo PSD2 certificate for the Sandbox:  
```openssl req -x509 -newkey rsa:4096 -keyout psd2_key.pem -out psd2_cert.pem -days 365 -nodes -subj '/CN=My App PISP AISP/C=NL'```

PSD2 private key: See contents of **psd2_key.pem**  
PSD2 public certificate: See contents of **psd2_cert.pem**  
PSD2 public certificate chain: See contents of **psd2_cert.pem**  

Create a client **private key** (2048 bits RSA) for the communication with the API:  
```openssl genrsa -out client_private_key.pem 2048```  
See contents of **client_private_key.pem**  

Create a client **public key**, based on the private key:  
```openssl rsa -in client_private_key.pem -outform PEM -pubout -out client_public_key.pem```  
See contents of **client_public_key.pem** 
 
Send a POST request with the **client public key** to the ```/v1/installation``` endpoint to get an Installation Token.  
Body JSON of the request, this contains the contents **client_public_key.pem** (replace newlines by \n):  
```json
{"client_public_key":"-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsgDXw1qErJ5AVrah9mK3\nFR1TRcW8jN+vx2V2UDoEks+o76ZNaZOJTbKhtxLGTJ4rpxl1z022MczqQgMF6aSi\nita9uJKnw+cxkYeCW5/zUZkmsJr+5gdmyYoNdZTxrBSpOjlaCkKgLEXp9F77IJhX\n+72aGMo/lu/MwOCtCBbn1m1f1JVXY9yO60oJO5T7H1rhZxcXjJ2aU1246VDfbBMa\nulgwLe91crO/36yPvlm6W0VvDBKgPDvoFNKeaW0kbhaFr6NITWYkysjpq5wJMhYR\nT0AgkZbjoRSD3ABQ2xbewpjD7kG/bsN0liGJSdOHiRQ3Ru+ndxDjbO2/PoMHaQjZ\njQIDAQAB\n-----END PUBLIC KEY-----\n"}
```

Received installation token:  
```16055df8b07491c2d901118ba1567bc62f6637eeaa3337cdeedff8e5cac8f95b```

### Create signature for payment-service-provider-credential  
You need to create a signature for the JSON data, to send to the ```/v1/payment-service-provider-credential``` endpoint.  

String to sign (**client public key** and Installation token combined). Contents of file **StringToSign.txt**:  
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsgDXw1qErJ5AVrah9mK3
FR1TRcW8jN+vx2V2UDoEks+o76ZNaZOJTbKhtxLGTJ4rpxl1z022MczqQgMF6aSi
ita9uJKnw+cxkYeCW5/zUZkmsJr+5gdmyYoNdZTxrBSpOjlaCkKgLEXp9F77IJhX
+72aGMo/lu/MwOCtCBbn1m1f1JVXY9yO60oJO5T7H1rhZxcXjJ2aU1246VDfbBMa
ulgwLe91crO/36yPvlm6W0VvDBKgPDvoFNKeaW0kbhaFr6NITWYkysjpq5wJMhYR
T0AgkZbjoRSD3ABQ2xbewpjD7kG/bsN0liGJSdOHiRQ3Ru+ndxDjbO2/PoMHaQjZ
jQIDAQAB
-----END PUBLIC KEY-----
16055df8b07491c2d901118ba1567bc62f6637eeaa3337cdeedff8e5cac8f95b
```  

Create signature for **StringToSign.txt** contents, signed with the private PSD2 key:  
```openssl dgst -sha256 -sign psd2_key.pem -out JsonBodySignature.txt StringToSign.txt```  
Encode signature as Base64:  
```openssl enc -base64 -in JsonBodySignature.txt -out JsonBodySignatureEncoded.txt -A```  

Base64 encoded signature inside **JsonBodySignatureEncoded.txt**:  
```
Jjqma7OYlECo6OsrpJ0ObtfWYnf3pTyAyyTQLRmQZtpqKPa8T4tTmQpFk+s5Od8HEfTgGvMQ98ps/HYYLCA/p3j5IORUNFhgTDzitKxegyp1G260M/QEHDepucNx514SdALVtR7FgnBubrZ0IIGMH59Sn59pRTr+k86qsEZWdbhfjEXTKDswaOIcJNz8UC7S6MJitn2hAzjc6nhaZ2KuizVXYKFajCW11Zs3TkgaIF37yhI3GDjkpyhXhBF3Vf8FHqe1eD+j12pDu6k8cjzOam6jPmffcagupgFBecZ5oPyfQAydLIUXgFo+MOu/Sx110u9jOQv6cOqYx5rS077eigQkcyyX+t1zcXYfFi4HXX9v7M88m87ija8qnkr5yz2aO1kWUoBp1i7dtlxulWjaReuCmCJmMQioEtLAcHBxbfmb6mWMf2jv+WggZ0Op2fM4feFJ205h33ZStybJwBH70fYxBYJ3elg5Ureln/BYJoOnpYgreNvbMaS9upt2lkl8T8U7iPnKBGZeXKLJ1758OuM8nao5tvEOHFbp/dfdDAAUOrnlpafU+fxb0wCwRiprA3ZjfmJzNWpJnr0pZVg+mHtlRm+4RathQWn55eO2wyWIdtCduenmhziLTfLmqX9aXPsyiL8Ww+PvrcZyJigqeAcdT4uYH24nf0QBKMqPJJY=
```

### Send PSD2 certificate and signature to payment-service-provider-credential  
The JSON data to send to the endpoint ```/v1/payment-service-provider-credential```:  
```json
{
	"client_payment_service_provider_certificate": "-----BEGIN CERTIFICATE-----\nMIIFIzCCAwugAwIBAgIJAKT0fvWrqcsNMA0GCSqGSIb3DQEBCwUAMCgxGTAXBgNV\nBAMMEE15IEFwcCBQSVNQIEFJU1AxCzAJBgNVBAYTAk5MMB4XDTIwMDUxOTExNTYy\nNFoXDTIxMDUxOTExNTYyNFowKDEZMBcGA1UEAwwQTXkgQXBwIFBJU1AgQUlTUDEL\nMAkGA1UEBhMCTkwwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQCgryG8\nIACUzS/uoARtNPasWCAPDetE2OLd1bi/nDkxBXBJqQh0S76GZKnl4FxnXYwR1kY7\n0qWn9GhNyjQL6Td2D18EBLzojp5BTjWbXfuMu4M3WreGqKxONbuW/r7lTFUCPn7h\nVBHMEEeaGStWVh36lhJ46CTho04/xhRjUS0xtl8k0rTUkz74h1grpv0oRaDGxH4a\nFVrkUlhPtRGNSC6q6iEWq+EhLw047fcNi4iRR5cUluwwplAKQ5wuNq1Ru8A6JSkW\neWnZm+KStYaczB1Hx2hW7NaMEVAr857+mSSAQ+LAJIFc8njZLG7arhHso1n7Oux8\nzTfRjdWC1yWYQR9nA2qBtGh308VjreIYtiJFUi/pm7cJ3uyYhsW9cVdZd3RE3eXw\nGSW6faTH2QCYh/4TF5IgI1Ap1/L4+zQt2zNGxe7nrd4xp271vXjx6mY6kGL1sacA\nFxhLyw323qm7Uln99AsjG18v9irr9ZrWKw0s7TG/0WPk38zfIfjtPrGUuekAeURh\nQUPsA7RzIprSbp5wHxAjKxvVutJWsvnJdGa+NM7yccQfd6yq0BmHDLNbq31E32Zd\n/OfN7Io5cKpP7gBbwNNIkjARMDUc/qMaaT8A44f/79I1VQDjw2AzwVNHQ9eK03FV\nBvAi9PXz0AOhX0RMYGJl4GOkayiqO3Zt1YbA6QIDAQABo1AwTjAdBgNVHQ4EFgQU\nthox26m3n7NrYC6U6sbqi4eIAy4wHwYDVR0jBBgwFoAUthox26m3n7NrYC6U6sbq\ni4eIAy4wDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAcG4qKBJGmQCH\niMjXWylvM+fvFC9vko/nlY/B321yHlkeeZqQJ5Tgk3e+Ez/vmbsvE8N0V3N+i+j2\n1bb4tzwukAVPgXD9t/W9hVIxYy4hWf+HKoCPkAXu6FqqtJhYRhrhRNC/W9QTHWi+\ntXDuheU1xNNGFS8nKrzAskKxUUgolBkw53X2g/c06K0WdliKUe0yOCPy+3iRkLXS\n7ic+TPcXFBa8EW5yUsiaBcu8+cjuOK3kFtixJa2ogeEygF4dD+SxUa47b2iOWR2R\n7LmPWal4DkMOF7JKJaRmLdIVhbUqliutEqjc9aztPqsnZ0pagRJ3R6oLY3W9c97O\nq1gi7cQmTZlqpTGAyUq8itgkJgvJC+LK3gNwBtcreecrK9SZI9ZU5n0B/Flnq7Uj\naJn5W5/hCiaiuI7+GfuuwvtcxAD9JljF+Wb45TJpPHIM/RCA+Fmaiq7sW6qk4vNQ\nmeBP9JDR9TNRd+g+zIe94MeJXoI7QGC4H5s9hbC10xXoHOI8FSdwEwvlim1OLyXr\ne+hF0/NlwEAfpV1LpEqiZWEARZlEa+pMGwzR4VFVB6snIo17x/u+mgx+U/esZ9tL\nPFko7V15BudbskW+VYKDdiKACCtwhvnzTDnWGXLAmtF5ThTRavHYvlnCT5zZoa5t\nlL6SwVaoW+Jk7Oem8KzMqjd0gqzK77g=\n-----END CERTIFICATE-----\n",
	"client_payment_service_provider_certificate_chain": "-----BEGIN CERTIFICATE-----\nMIIFIzCCAwugAwIBAgIJAKT0fvWrqcsNMA0GCSqGSIb3DQEBCwUAMCgxGTAXBgNV\nBAMMEE15IEFwcCBQSVNQIEFJU1AxCzAJBgNVBAYTAk5MMB4XDTIwMDUxOTExNTYy\nNFoXDTIxMDUxOTExNTYyNFowKDEZMBcGA1UEAwwQTXkgQXBwIFBJU1AgQUlTUDEL\nMAkGA1UEBhMCTkwwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQCgryG8\nIACUzS/uoARtNPasWCAPDetE2OLd1bi/nDkxBXBJqQh0S76GZKnl4FxnXYwR1kY7\n0qWn9GhNyjQL6Td2D18EBLzojp5BTjWbXfuMu4M3WreGqKxONbuW/r7lTFUCPn7h\nVBHMEEeaGStWVh36lhJ46CTho04/xhRjUS0xtl8k0rTUkz74h1grpv0oRaDGxH4a\nFVrkUlhPtRGNSC6q6iEWq+EhLw047fcNi4iRR5cUluwwplAKQ5wuNq1Ru8A6JSkW\neWnZm+KStYaczB1Hx2hW7NaMEVAr857+mSSAQ+LAJIFc8njZLG7arhHso1n7Oux8\nzTfRjdWC1yWYQR9nA2qBtGh308VjreIYtiJFUi/pm7cJ3uyYhsW9cVdZd3RE3eXw\nGSW6faTH2QCYh/4TF5IgI1Ap1/L4+zQt2zNGxe7nrd4xp271vXjx6mY6kGL1sacA\nFxhLyw323qm7Uln99AsjG18v9irr9ZrWKw0s7TG/0WPk38zfIfjtPrGUuekAeURh\nQUPsA7RzIprSbp5wHxAjKxvVutJWsvnJdGa+NM7yccQfd6yq0BmHDLNbq31E32Zd\n/OfN7Io5cKpP7gBbwNNIkjARMDUc/qMaaT8A44f/79I1VQDjw2AzwVNHQ9eK03FV\nBvAi9PXz0AOhX0RMYGJl4GOkayiqO3Zt1YbA6QIDAQABo1AwTjAdBgNVHQ4EFgQU\nthox26m3n7NrYC6U6sbqi4eIAy4wHwYDVR0jBBgwFoAUthox26m3n7NrYC6U6sbq\ni4eIAy4wDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAcG4qKBJGmQCH\niMjXWylvM+fvFC9vko/nlY/B321yHlkeeZqQJ5Tgk3e+Ez/vmbsvE8N0V3N+i+j2\n1bb4tzwukAVPgXD9t/W9hVIxYy4hWf+HKoCPkAXu6FqqtJhYRhrhRNC/W9QTHWi+\ntXDuheU1xNNGFS8nKrzAskKxUUgolBkw53X2g/c06K0WdliKUe0yOCPy+3iRkLXS\n7ic+TPcXFBa8EW5yUsiaBcu8+cjuOK3kFtixJa2ogeEygF4dD+SxUa47b2iOWR2R\n7LmPWal4DkMOF7JKJaRmLdIVhbUqliutEqjc9aztPqsnZ0pagRJ3R6oLY3W9c97O\nq1gi7cQmTZlqpTGAyUq8itgkJgvJC+LK3gNwBtcreecrK9SZI9ZU5n0B/Flnq7Uj\naJn5W5/hCiaiuI7+GfuuwvtcxAD9JljF+Wb45TJpPHIM/RCA+Fmaiq7sW6qk4vNQ\nmeBP9JDR9TNRd+g+zIe94MeJXoI7QGC4H5s9hbC10xXoHOI8FSdwEwvlim1OLyXr\ne+hF0/NlwEAfpV1LpEqiZWEARZlEa+pMGwzR4VFVB6snIo17x/u+mgx+U/esZ9tL\nPFko7V15BudbskW+VYKDdiKACCtwhvnzTDnWGXLAmtF5ThTRavHYvlnCT5zZoa5t\nlL6SwVaoW+Jk7Oem8KzMqjd0gqzK77g=\n-----END CERTIFICATE-----\n",
	"client_public_key_signature": "Jjqma7OYlECo6OsrpJ0ObtfWYnf3pTyAyyTQLRmQZtpqKPa8T4tTmQpFk+s5Od8HEfTgGvMQ98ps/HYYLCA/p3j5IORUNFhgTDzitKxegyp1G260M/QEHDepucNx514SdALVtR7FgnBubrZ0IIGMH59Sn59pRTr+k86qsEZWdbhfjEXTKDswaOIcJNz8UC7S6MJitn2hAzjc6nhaZ2KuizVXYKFajCW11Zs3TkgaIF37yhI3GDjkpyhXhBF3Vf8FHqe1eD+j12pDu6k8cjzOam6jPmffcagupgFBecZ5oPyfQAydLIUXgFo+MOu/Sx110u9jOQv6cOqYx5rS077eigQkcyyX+t1zcXYfFi4HXX9v7M88m87ija8qnkr5yz2aO1kWUoBp1i7dtlxulWjaReuCmCJmMQioEtLAcHBxbfmb6mWMf2jv+WggZ0Op2fM4feFJ205h33ZStybJwBH70fYxBYJ3elg5Ureln/BYJoOnpYgreNvbMaS9upt2lkl8T8U7iPnKBGZeXKLJ1758OuM8nao5tvEOHFbp/dfdDAAUOrnlpafU+fxb0wCwRiprA3ZjfmJzNWpJnr0pZVg+mHtlRm+4RathQWn55eO2wyWIdtCduenmhziLTfLmqX9aXPsyiL8Ww+PvrcZyJigqeAcdT4uYH24nf0QBKMqPJJY="
}
```
This JSON data contains the content of **psd2_cert.pem** (geplace newlines by \n) and the content of **JsonBodySignatureEncoded.txt** (Base64 encoded signature)  

Use the Installation token as header value for ```X-Bunq-Client-Authentication```  

### Postman screenshot
![screenshot](https://github.com/basst85/bunq_psd2_example/raw/master/image.png)
