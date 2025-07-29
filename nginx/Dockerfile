FROM nginx:alpine

# Install envsubst for template processing
RUN apk add --no-cache gettext

# Create nginx configuration template and startup script
RUN cat > /etc/nginx/templates/default.conf.template << 'EOF'
server {
    listen 80;
    server_name _;

    location /__clerk/ {
        rewrite          ^/__clerk/(.*)$ /$1 break;
        proxy_pass       ${CLERK_API_URL};
        proxy_set_header Clerk-Proxy-Url ${CLERK_PROXY_URL};
        proxy_set_header Clerk-Secret-Key ${CLERK_SECRET_KEY};
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_redirect   off;
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