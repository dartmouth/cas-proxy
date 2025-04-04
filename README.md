# CAS Proxy

The purpose of this repository is to demo an example using [mod_auth_cas](https://github.com/apereo/mod_auth_cas) to protect another web service with CAS single sign-on. In particular, Apache is configured to proxy requests to the backend web service, which can sometimes be useful if that backend web service does not have an easy way to implement single sign-on itself.

Dartmouth College recently came across this situation when we deployed [Open OnDemand](https://openondemand.org/). We needed to use [mod_auth_gssapi](https://github.com/gssapi/mod_auth_gssapi) so that we could access storage with Kerberized ACLs. However, GSSAPI is only username/password protection so we wanted to layer on our instituational single sign-on. Apache does not support using two different authentication modules so our solution was to use two Apaches.

The example below use nginx as the example service to make it more clear what is being proxied.

## Run an example service to be proxied

```sh
docker run -d --rm --publish 80:80 nginx
```

## Build

```sh
docker build -t cas-proxy .
```

## Run the CAS Proxy

```sh
# Get the local system's IP address
# This work on wifi connected Mac, change as needed
LOCAL_IP=$(ipconfig getifaddr en0)

docker run -d \
--name cas-proxy \
-p 8443:443 \
-e EMAIL_ADDRESS=research.computing@dartmouth.edu \
-e WEB_ADDRESS=https://localhost:8443 \
-e PROXY_ADDRESS=http://$LOCAL_IP:80 \
-e CAS_ADDRESS=https://login.dartmouth.edu \
-e SERVER_NAME=localhost:8443 \
cas-proxy
```

## Test

Browse to https://localhost:8443/ and accept the invalid certificate. You should be sent through single sign-on and then are presented with the nginx welcome page. This shows that you are using Apache's mod_auth_cas to protect and proxy the nginx web service.
