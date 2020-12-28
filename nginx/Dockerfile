ARG NGINX_IMAGE
FROM ${NGINX_IMAGE:-'nginx:1.19'}

#  set timezome
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
EXPOSE 80
EXPOSE 443
CMD ["debug", "-g", "daemon off;"]