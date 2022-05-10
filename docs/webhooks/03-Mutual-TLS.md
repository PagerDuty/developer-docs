---
tags: [webhooks]
---

# Mutual TLS


## Why use Mutual TLS with webhooks?

Mutual TLS builds upon normal TLS by adding client authentication in addition to server authentication to let you verify that webhooks you receive actually came from PagerDuty.

## How it works

If you specify an HTTPS endpoint for your webhook, PagerDuty will verify your server's certificate to ensure that we are communicating with your server. (See details below)

To take advantage of mutual TLS, you can configure your server to verify PagerDuty's client certificate. (See steps below)

## Steps to verify PagerDuty's client certificate

These steps assume you already have server authentication setup.

In general, there are five steps needed to turn on client authentication for your server:

1. Download the PEM version of the [DigiCert Global Root CA](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) certificate.
2. Turn on client certificate verification. 
3. Specify the CA certificate from step 1 as trusted.
4. Set the verification depth to 2 since our PagerDuty certificate is actually signed by the [DigiCert SHA2 Secure Server CA](https://dl.cacerts.digicert.com/DigiCertSHA2SecureServerCA.crt) which is an intermediate CA under DigiCert Global Root CA.
5. Verify the client certificate is actually from PagerDuty by inspecting its Subject Domain Name.

Now we will go over sample server configurations for NGINX and Apache.

### NGINX Sample Config

```
server {

    listen 443 ssl default_server;
    # ... existing SSL configuration for server authentication ...

    ssl_verify_client on;
    ssl_client_certificate /path/to/DigiCert_Global_Root_CA.pem;
    ssl_verify_depth 2;

    location / {
        if ($ssl_client_s_dn !~ "CN=webhooks.pagerduty.com") {
            return 403;
        }

        # ... existing location configuration ...
    }
}
```

For more info, see http://nginx.org/en/docs/http/ngx_http_ssl_module.html.

### Apache Sample Config

```
Listen 443
<VirtualHost *:443>
    # ... existing SSL configuration for server authentication ...

    SSLVerifyClient require
    SSLCACertificateFile "/path/to/DigiCert_Global_Root_CA.pem"
    SSLVerifyDepth 2
</VirtualHost>

<Directory /var/www/>
    Require expr "%{SSL_CLIENT_S_DN_CN} == 'webhooks.pagerduty.com'"

    # ... existing directory configuration ...
</Directory>
```

For more info, see https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html#accesscontrol and https://httpd.apache.org/docs/2.4/mod/mod_ssl.html.

## PagerDuty's Server Verification

### Trusted Root Certificates

PagerDuty supports [Mozilla’s list of included CA Certificates](https://wiki.mozilla.org/CA/Included_Certificates) (HTML or CSV list available). We will drop webhooks if your certificate is self-signed or if your certificate uses a CA outside of this list.

Note: At this time, PagerDuty can only properly verify server certificate chains which are presented in order. Out of order chains will be rejected and result in dropped webhooks. You can test yours using the OpenSSL command below and looking at the numbers / order in the certificate chain in the output.

### Supported TLS Versions

PagerDuty’s webhook delivery system supports only TLS v1.2.

### Supported Cipher Suites

```
TLS_EMPTY_RENEGOTIATION_INFO_SCSV
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384
TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
TLS_DHE_DSS_WITH_AES_256_GCM_SHA384
TLS_DHE_RSA_WITH_AES_256_CBC_SHA256
TLS_DHE_DSS_WITH_AES_256_CBC_SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256
TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
TLS_DHE_DSS_WITH_AES_128_GCM_SHA256
TLS_DHE_RSA_WITH_AES_128_CBC_SHA256
TLS_DHE_DSS_WITH_AES_128_CBC_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
TLS_DHE_RSA_WITH_AES_256_CBC_SHA
TLS_DHE_DSS_WITH_AES_256_CBC_SHA
TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA
TLS_ECDH_RSA_WITH_AES_256_CBC_SHA
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
TLS_DHE_RSA_WITH_AES_128_CBC_SHA
TLS_DHE_DSS_WITH_AES_128_CBC_SHA
TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA
TLS_ECDH_RSA_WITH_AES_128_CBC_SHA
```

## Testing your connection

Use a tool like [SSL Labs](https://www.ssllabs.com/ssltest/) or OpenSSL to verify that your certificate is valid and working.

From [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/online/ch-testing-with-openssl.html):
```
openssl s_client -connect www.pagerduty.com:443
```
