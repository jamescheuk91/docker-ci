# OS base image
FROM centos:centos7
MAINTAINER jccf091

# Use Yum to install NGINX
RUN yum -y update; yum clean all
RUN yum -y install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm; yum -y makecache
RUN yum -y install nginx-1.8.0

# Cleanup some default NGINX configuration files we don’t need:
RUN rm /etc/nginx/conf.d/default.conf
RUN rm /etc/nginx/conf.d/example_ssl.conf

# Go ahead and add our configuration files (we still need to make these):
COPY conf/jenkins.conf /etc/nginx/conf.d/jenkins.conf
COPY conf/nginx.conf /etc/nginx/nginx.conf

# We want NGINX to listen on Port 80 so let’s make sure that port is exposed:
EXPOSE 80
# Finish up by making sure NGINX is started
CMD ["nginx"]
