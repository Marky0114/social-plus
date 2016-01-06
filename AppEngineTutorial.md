# Google+ Domains API - Quick Start (App Engine + Python + Service Accounts) #

## introduction ##

Google+ Domains API are meant to interact with your G+ accounts in the domain. With these APIs you can manage circles, read and write posts, shares, and  comments etc.. more informations  here https://developers.google.com/+/domains/


This tutorial is for creating an application that uses the **Domains API**, running on Google **App Engine** with **python**.

### create GAE (Google App Engine) app ###

Create a new app in GAE as usual, but when you have to specify the access to the app, select `Restricted to the following Google Apps domain` specifying your domain.

### setup API access ###

Only after having created the GAE app, go to the [api console](https://code.google.com/apis/console) and select you GAE app from the dropdown menu on the left.

Activate the Google+ Domains API and the other APIs that you need under _services_.

In _API Access_ create the secrets, selecting _Service Account_. Download and store with care the private key.

### authorize service account

From the [domain console](http://admin.google.com), authorize your new service account going under `security > advanced settings > Manage third party OAuth Client access` and add your client ID (you can find it in the API console, _Client ID_ **not** _email_) and the scopes that you need.

A few useful scopes for G+ Domains are:

  * https://www.googleapis.com/auth/plus
  * https://www.googleapis.com/auth/plus.circles.read
  * https://www.googleapis.com/auth/plus.circles.write
  * https://www.googleapis.com/auth/plus.me
  * https://www.googleapis.com/auth/plus.profiles.read
  * https://www.googleapis.com/auth/plus.stream.read
  * https://www.googleapis.com/auth/plus.stream.write

**important:** at the moment (August 2013) the scope `plus.login` is not supported and should not be used.

### convert you certificate ###

You have downloaded a `.p12` certificate (PKCS12 key), which must be converted to a PEM key in order to work on GAE with python. Use these commands (should work on both OSX and Linux):
```
    >> openssl pkcs12 -passin pass:notasecret -in privatekey.p12 -nocerts -passout pass:notasecret -out key.pem
    
    >> openssl pkcs8 -nocrypt -in key.pem -passin pass:notasecret -topk8 -out privatekey.pem
    
    >> rm key.pem
```
Check that your new `.pem` key starts with a line similar to `-----BEGIN PRIVATE KEY-----`, after which the key begins. If at the beginning of the file you find `bag attributes` or similar, remove that part (should not happen with the commands provided).

### create the service in python on GAE ###

follow the following script to build the G+ Domains service with python on GAE. Before running your application, remember to update you `app.yaml` file to include the _pycrypto_ library:
```
    libraries:
    - name: pycrypto
      version: "2.6"
```
and check that you're using python 2.7:
```
    runtime: python27
```
this is an example file to create the G+ Domains service:

```
import json
import httplib2
import sys
import os

from apiclient.discovery import build
from oauth2client.client import SignedJwtAssertionCredentials


SERVICE_ACCOUNT_EMAIL = '<your-id>@developer.gserviceaccount.com'
SERVICE_ACCOUNT_PEM_FILE_PATH = 'privatekey.pem'
DOMAIN_NAME = "your.domain.com"

def create_plus_service(user_email):

    """ read credentials from file """
    f = file(SERVICE_ACCOUNT_PEM_FILE_PATH, 'rb')
    key = f.read()
    f.close()

    """ authorize http object with delegation (as if you were user_email) """
    http = httplib2.Http()
    credentials = SignedJwtAssertionCredentials(SERVICE_ACCOUNT_EMAIL, key, scope=[
        "https://www.googleapis.com/auth/plus.circles.read",
        "https://www.googleapis.com/auth/plus.circles.write",
        "https://www.googleapis.com/auth/plus.profiles.read",
        "https://www.googleapis.com/auth/plus.stream.read",
        "https://www.googleapis.com/auth/plus.stream.write",
    ], sub=user_email)
    http = credentials.authorize(http)

    
    """ build and return authorized service """
    service = build("plus", "v1domains", http=http)
    return service
```