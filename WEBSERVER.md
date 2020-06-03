## NGINX Best Practices
Reference: https://docs.sucuri.net/warnings/hardening/

- Disable Server Banners by adding this code to the `/etc/nginx/nginx.conf` file

  ```
  http {
           server_tokens off;
  }
  ```

- (optional) Set HTTP header session_id cookie with HttpOnly and Secure flag by adding this code to the `/etc/nginx/nginx.conf`file

  ```
  proxy_cookie_path / "/; HttpOnly; Secure"; # enable at your own risk, may break certain apps
  ```

## SSL and TLS Deployment Best Practices

Reference:
- https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices
- https://ssllabs.com/ssltest to get the analysis report of the TLS configuration.

**Mitigate the Logjam Attack**

Reference: https://weakdh.org/

There is **known security risk** of pre-computed Diffie-Hellman (DH) key exchange with the 1024-bit prime or below. This repository use 4096-bit prime.

If you think your web is a target of an attacker with enough computational power (let's say NSA).

```
$ cd volumes/nginx
$ openssl dhparam -out dhparams.pem <"2048"-bit prime or above>
```

**SSL Configuration**

Reference:
- https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices
- https://www.freecodecamp.org/news/docker-nginx-letsencrypt-easy-secure-reverse-proxy-40165ba3aee2/

This repository priority are (in order) **security**, **speed** and the **handshake simulation**.
- We support TLSv1.2 as the minimum security standard and consideration for the handshake compatibility.
- We support TLSv1.3 for faster speed.
- We dropped support for the browsers and devices that only support [CBC Cipher Suites](https://github.com/tempatkerja/docker-odoo-https#cbc-cipher-suites)

### CBC Cipher Suites

Reference: https://blog.qualys.com/technology/2019/04/22/zombie-poodle-and-goldendoodle-vulnerabilities

It is reported for websites that use CBC (Cipher Block Chaining) block ciphers modes **may be** vulnerable.
These vulnerabilities are applicable only if the server uses TLS 1.2 or TLS 1.1 or TLS 1.0 with CBC cipher modes.

The CBC block cipher we drop is TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 or `OpenSSL cipher name: ECDHE-RSA-AES256-SHA384`.
This CBC block cipher is **only reported as weak**, it is not yet encouraged (by Qualys, Inc) to move away from CBC block cipher modes.

> We are only encouraging to move away from CBC based cipher suits after 4 new CBC based vulnerabilities. As of now, there is no grade change for CBC and servers can continue to use. - Yash K.S

- We drop the handshake simulation support for:
  - Safari 6 / iOS 6.0.1
  - Safari 7 / iOS 7.1
  - Safari 7 / OS X 10.9
  - Safari 8 / iOS 8.4
  - Safari 8 / OS X 10.10

## PWA best practice: tips for designing great Progressive Web Apps

Reference:
- https://medium.com/progressivewebapps/pwa-best-practices-tips-for-designing-great-progressive-web-apps-96e92298d2ec
- https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=en to get the analysis report of the web performance.

> Perceived Performance must be Seamless. Show placeholder and shadow first. Make opacity transition.

## Page auditor

**Qualys SSL Labs**

[Qualys SSL Labs](https://ssllabs.com/ssltest) audits the page by simulating a TLS handshake. The tests determine the page's security and compatibility with the list of browsers and devices. Analyzing the report will help you understand the page's security loophole and compatibility.

> Take a look at these key words: OCSP stapling, Cipher Suites, Handshake Simulation, Protocol Details.

```
$ openssl ciphers -convert [SSL Cipher Suites from https://ssllabs.com/ssltest]
```

**Lighthouse**

[Lighthouse](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=en) audits a page performance. Although designed for web apps, analyzing the report will help you understand the page's performance and help you focus with a list of applicable improvements.

> Take a look at these key words: Opportunities, Additional items to manually checks.
