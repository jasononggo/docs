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

## PWA best practice: tips for designing great Progressive Web Apps
Reference:
- https://medium.com/progressivewebapps/pwa-best-practices-tips-for-designing-great-progressive-web-apps-96e92298d2ec
- https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=en to get the analysis report of the web performance.

> Perceived Performance must be Seamless. Show placeholder and shadow first. Make opacity transition.
