FROM node:22-alpine3.21 AS deps

# Update system packages to patch security vulnerabilities (zlib CVE-2026-22184 and others)
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

FROM deps AS dev

COPY . .
COPY docker-entrypoint.sh /usr/local/bin/gestionev-frontend-entrypoint
RUN chmod +x /usr/local/bin/gestionev-frontend-entrypoint

EXPOSE 5173

ENTRYPOINT ["gestionev-frontend-entrypoint"]
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5173"]

FROM deps AS build

ARG VITE_API_URL=/api
ENV VITE_API_URL=${VITE_API_URL}

COPY . .
RUN npm run build

FROM nginx:1.27-alpine3.21 AS production

# Update system packages to patch security vulnerabilities (zlib CVE-2026-22184 and others)
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
