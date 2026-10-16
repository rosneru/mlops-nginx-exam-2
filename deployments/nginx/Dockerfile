# Instead of ':latest' using ':1.29' here to have a reproducible build / 
# not possibly changing the Nginx version on every build
FROM nginx:1.29

COPY nginx.conf /etc/nginx/nginx.conf
COPY certs /etc/nginx/certs
COPY .htpasswd /etc/nginx/.htpasswd

EXPOSE 80
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
