FROM nginx:mainline-alpine

# 資安：每次建置強制把所有 OS 套件升級到最新修補版
# （修復 curl / openssl / expat / util-linux 等 CVE；--available 強制拉最新可用版）
# 改 REBUILD_DATE 可強制重新執行此層，確保拉到當下最新修補
ARG REBUILD_DATE=2026-09-17
RUN apk update && \
    apk upgrade --no-cache --available && \
    apk add --no-cache --upgrade ca-certificates openssl && \
    rm -rf /var/cache/apk/*

RUN printf 'server {\n\
    listen 8080;\n\
    server_tokens off;\n\
\n\
    # FME 帳號驗證代理\n\
    location /auth {\n\
        proxy_pass https://eip.fme.com.tw/FMEIP/AasApi/CheckUserId;\n\
        proxy_ssl_server_name on;\n\
        proxy_set_header Host eip.fme.com.tw;\n\
        proxy_set_header Content-Type "application/json";\n\
        proxy_pass_request_headers on;\n\
    }\n\
\n\
    # 前端代理到 Cloudflare Pages（永遠保持最新版）\n\
    location / {\n\
        proxy_pass https://music-inspection.pages.dev;\n\
        proxy_ssl_server_name on;\n\
        proxy_set_header Host music-inspection.pages.dev;\n\
        proxy_set_header X-Real-IP $remote_addr;\n\
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;\n\
        proxy_hide_header X-Frame-Options;\n\
        add_header X-Frame-Options "SAMEORIGIN" always;\n\
        add_header X-Content-Type-Options "nosniff" always;\n\
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;\n\
        proxy_redirect https://music-inspection.pages.dev/ /;\n\
    }\n\
}\n' > /etc/nginx/conf.d/default.conf

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
