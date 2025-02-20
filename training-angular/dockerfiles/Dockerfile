#Node v18
FROM node:18-alpine as build-step
RUN mkdir /training-angular
WORKDIR /training-angular
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:1.17.1-alpine
COPY --from=build-step /training-angular/dist/training-angular/browser /usr/share/nginx/html
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf
