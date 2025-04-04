# CAS Proxy

The purpose of this repository is to demo an example of using [mod_auth_cas](https://github.com/apereo/mod_auth_cas) to protect some other web service with single sign-on. In particular, Apache is configured to proxy requests to the backend web service, which can sometimes be useful if that backend web service does not have an easy way to implement single sign-on its self.

Dartmouth College recently came across this situation when we deployed [Open OnDemand]. We needed to use [mod_auth_gssapi](https://github.com/gssapi/mod_auth_gssapi) so that we could access storage with Kerberized ACLs. However, GSSAPI is only protected with username/password so we wanted to layer on single sign-on. Apache does not support using two different authentication modules so our solution was to use two Apaches.

## Run an example service to be proxied

```sh
docker run -d --rm --publish 80:80 nginx
```

## Build and run the CAS Proxy

```sh
docker build -t cas-proxy .

# Works for a Mac on wifi, change as needed
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

Browse to https://localhost:8443/ and accept the invalid certificate. You should be sent through single sign-on and then when you return are presented with the nginx welcome page. This shows that you are using Apache's mod_auth_cas to protect the nginx web service.
