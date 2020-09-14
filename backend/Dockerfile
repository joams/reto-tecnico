# Se descarga la imagen debian stable-slim
# Este ambiente es más ligero que otras opciones linux, por lo que el resultado en cuanto a container resultará menos pesado en memoria.
FROM debian:stable-slim

# actualiza ubuntu
RUN apt update

# define las variables de entorno (ENV) que voy a usar durante la ejecución de esta imagen
ENV php_conf /etc/php/7.3/fpm/php.ini
ENV nginx_conf /etc/nginx/nginx.conf
ENV supervisor_conf /etc/supervisor/supervisord.conf

# Desabilita el Prompt durante la instalacion
ARG DEBIAN_FRONTEND=noninteractive

# Instala nginx, php-fpm y supervisord desde el repositorio de debian
RUN apt install -y nginx php-fpm supervisor composer curl && \
    rm -rf /var/lib/apt/lists/* && \
    apt clean
    

#Luego de la instalación de lo que necesito, proceso a configurar nginx y php
# Se habilita PHP-fpm en nginx 
COPY config/default /etc/nginx/sites-available/default

RUN sed -i -e 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' ${php_conf} && \
    echo "\ndaemon off;" >> ${nginx_conf}
    
#crea carpeta /run/php y le asigno un owner www-data a dicha carpeta y a /var/www/html
RUN mkdir -p /run/php && \
    chown -R www-data:www-data /var/www/html && \
    chown -R www-data:www-data /run/php

#copio los archivos del bacjend a la carpeta de var/www/html  de la imagen
COPY code /var/www/html

#ejecuta el localhost e instala composer para poder ejecutar el slim
RUN cd /var/www/html && composer install

# Se copia la configuracion de supervisor a la variable de entorno
COPY config/supervisord.conf ${supervisor_conf}

# Configuracion de los Volume
VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]

# asigna los puertos indicados para la aplicacion
EXPOSE 80 443


# Copia tarts.sh en la imagen y define los comandos por defecto en el contenedor
COPY start.sh /start.sh
CMD ["./start.sh"]

# Configura healthcheck para validad que todo este corriendo
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/