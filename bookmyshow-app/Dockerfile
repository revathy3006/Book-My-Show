FROM node:18 AS build

WORKDIR /app

COPY package.json package-lock.json ./

ENV NODE_OPTIONS=--openssl-legacy-provider

RUN npm ci --legacy-peer-deps --no-audit --no-fund

COPY . .

ENV CI=false
ENV PORT=3000

RUN npm run build


FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
