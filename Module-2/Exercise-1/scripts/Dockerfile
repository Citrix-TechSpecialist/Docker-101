FROM fnichol/uhttpd
MAINTAINER Mayank Tahilramani and Brian Tannous
COPY ./cpx-blog /www
EXPOSE 80
ENTRYPOINT /usr/sbin/run_uhttpd -f -p 80 -h /www
CMD [""]
