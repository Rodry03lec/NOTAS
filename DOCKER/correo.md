# La conexión  para las pruebas con correos
```javascript
services:
  mailpit:
    image: axllent/mailpit:latest
    container_name: local_mailpit
    restart: unless-stopped
    volumes:
      - mailpit_data:/data
    environment:
      - MP_MAX_MESSAGES=500
      - TZ=America/La_Paz
    ports:
      - "1025:1025" # SMTP
      - "8025:8025" # Web UI

volumes:
  mailpit_data:
```

```javascript
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_SECURE=false
MAIL_USER=
MAIL_PASS=
MAIL_FROM="SISCAP <no-reply@siscap.local>"
```
