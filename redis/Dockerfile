ARG REDIS_IMAGE
FROM ${REDIS_IMAGE:-'redis:6.0'}

ARG TZ
#  set timezome
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
