FROM nginx:1.17
RUN usermod -u 1001 nginx
RUN mkdir /app
COPY default.conf /etc/nginx/conf.d/
EXPOSE 8000
