# Use the official Apache HTTPD base image
FROM httpd:latest

# Copy the custom index.html to the web server's document root
COPY ./index.html /usr/local/apache2/htdocs/

# Copy the custom httpd.conf file with reverse proxy configuration
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf