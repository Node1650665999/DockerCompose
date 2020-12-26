ARG MYSQL_IMAGE
FROM ${MYSQL_IMAGE}

#  set timezome
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
