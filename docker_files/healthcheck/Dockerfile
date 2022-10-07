FROM centos
RUN yum install -y curl httpd 
HEALTHCHECK --interval=2m --timeout=3s CMD curl 127.0.0.1 || exit 1
ENTRYPOINT /usr/sbin/httpd