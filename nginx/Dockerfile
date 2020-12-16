FROM nginx:1.12

#  set timezome
ENV TZ=Asia/Shanghai
ENV NAME=TCL
ENV SEX=MAN
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
