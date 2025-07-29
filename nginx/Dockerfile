FROM nginx:alpine

# Install envsubst for template processing
RUN apk add --no-cache gettext

# Create templates directory and nginx configuration template
RUN mkdir -p /etc/nginx/templates && \
    cat > /etc/nginx/templates/default.conf.template << 'EOF'
server {
    listen 80;
    server_name _;

    location /__clerk/ {
        rewrite          ^/__clerk/(.*)$ /$1 break;
        proxy_pass       ${CLERK_API_URL};
        proxy_set_header Clerk-Proxy-Url ${CLERK_PROXY_URL};
        proxy_set_header Clerk-Secret-Key ${CLERK_SECRET_KEY};
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_redirect   off;

        # Add connection and timeout settings to prevent upstream errors
        proxy_connect_timeout       60s;
        proxy_send_timeout          60s;
        proxy_read_timeout          60s;
        proxy_buffering             off;
        proxy_http_version          1.1;
        proxy_set_header Connection "";
    }
}
EOF

# Create startup script that processes template and starts nginx
RUN cat > /docker-entrypoint.sh << 'EOF'
#!/bin/sh
envsubst '${CLERK_PROXY_URL} ${CLERK_SECRET_KEY} ${CLERK_API_URL}' < /etc/nginx/templates/default.conf.template > /etc/nginx/conf.d/default.conf
exec nginx -g "daemon off;"
EOF

RUN chmod +x /docker-entrypoint.sh

EXPOSE 80

CMD ["/docker-entrypoint.sh"]