# Hugo Runtime Image
# Hardened nginx for serving static Hugo sites

# renovate: datasource=docker depName=nginx
FROM nginx:1.29-alpine@sha256:1d13701a5f9f3fb01aaa88cef2344d65b6b5bf6b7d9fa4cf0dca557a8d7702ba

LABEL maintainer="Simon Lauger <simon@lauger.de>"
LABEL description="Hardened nginx for Hugo static sites"

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Remove default nginx config
RUN rm -f /etc/nginx/conf.d/default.conf

# Create directories and set permissions
RUN mkdir -p /var/cache/nginx/client_temp \
             /var/cache/nginx/proxy_temp \
             /var/cache/nginx/fastcgi_temp \
             /var/cache/nginx/uwsgi_temp \
             /var/cache/nginx/scgi_temp \
             /usr/share/nginx/html \
             /etc/nginx/conf.d && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html && \
    chown -R nginx:nginx /var/log/nginx && \
    chmod -R 755 /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:80/ || exit 1

EXPOSE 80

USER nginx

CMD ["nginx", "-g", "daemon off;"]
