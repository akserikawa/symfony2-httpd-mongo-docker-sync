FROM httpd:2.4

RUN apt-get update && apt-get upgrade -y

# Copy apache vhost file to proxy php requests to php-fpm container
COPY demo.apache.conf /usr/local/apache2/conf/demo.apache.conf
RUN echo "Include /usr/local/apache2/conf/demo.apache.conf" \
    >> /usr/local/apache2/conf/httpd.conf

# RUN apt-get install -y curl wget

# RUN mkdir -p /app/var/cache /app/vendor