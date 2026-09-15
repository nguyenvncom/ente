# STAGE 1
FROM node:24 AS builder

RUN apt-get update && \
    apt-get install -y curl build-essential && \
    rm -rf /var/lib/apt/lists/* && \
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable --target wasm32-unknown-unknown

ENV PATH="/root/.cargo/bin:${PATH}"
WORKDIR /build
COPY web/package.json web/package-lock.json ./web/
WORKDIR /build/web
RUN npm ci
WORKDIR /build
COPY web/ ./web
COPY rust/ ./rust
WORKDIR /build/web
ENV NEXT_PUBLIC_ENTE_ENDPOINT=ENTE_API_ORIGIN_PLACEHOLDER
RUN npm run build:photos
RUN npm run build:albums
RUN npm run build:accounts
RUN npm run build:auth
RUN npm run build:cast
RUN npm run build:share
RUN npm run build:embed
RUN npm run build:paste
RUN npm run build:locker
RUN npm run build:memories

# STAGE 2
FROM nginx
WORKDIR /out
COPY --from=builder /build/web/apps/photos/out /out/photos
COPY --from=builder /build/web/apps/albums/out /out/albums
COPY --from=builder /build/web/apps/accounts/out /out/accounts
COPY --from=builder /build/web/apps/auth/out /out/auth
COPY --from=builder /build/web/apps/cast/out /out/cast
COPY --from=builder /build/web/apps/share/out /out/share
COPY --from=builder /build/web/apps/embed/out /out/embed
COPY --from=builder /build/web/apps/paste/out /out/paste
COPY --from=builder /build/web/apps/locker/out /out/locker
COPY --from=builder /build/web/apps/memories/out /out/memories

COPY <<'EOF' /etc/nginx/conf.d/default.conf
server {
    listen 3000;
    root /out/photos;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3001;
    root /out/accounts;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3002;
    root /out/albums;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3003;
    root /out/auth;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3004;
    root /out/cast;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3005;
    root /out/share;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3006;
    root /out/embed;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3008;
    root /out/paste;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3009;
    root /out/locker;
    location / { try_files $uri $uri.html /index.html; }
}
server {
    listen 3010;
    root /out/memories;
    location / { try_files $uri $uri.html /index.html; }
}
EOF

EXPOSE 3000 3001 3002 3003 3004 3005 3006 3008 3009 3010
ENV ENTE_API_ORIGIN=http://localhost:8080

COPY <<EOF /docker-entrypoint.d/90-replace-ente-env.sh
find /out -name '*.js' |
    xargs sed -i'' "s#ENTE_API_ORIGIN_PLACEHOLDER#\$ENTE_API_ORIGIN#g"
EOF

RUN chmod +x /docker-entrypoint.d/90-replace-ente-env.sh

COPY <<'EOF' /docker-entrypoint.d/40-enable-ipv6.sh
#!/bin/sh
if [ -f /proc/net/if_inet6 ]; then
    sed -i 's/#listen \[::\]:/listen [::]:/g' /etc/nginx/conf.d/default.conf
fi
EOF

RUN chmod +x /docker-entrypoint.d/40-enable-ipv6.sh
