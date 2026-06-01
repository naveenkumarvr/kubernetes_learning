# TLS Enabling and Termination at Gateway API

## Step:1 Generate TLS Certificate
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=echo.local"
```