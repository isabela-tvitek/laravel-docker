# Usar a imagem oficial do PHP com Apache
FROM php:8.2-apache

# Definir o diretório de trabalho
WORKDIR /var/www/html

# Copiar o código-fonte do Laravel e o arquivo de configuração do Apache
COPY . /var/www/html
COPY .docker/vhost.conf /etc/apache2/sites-available/000-default.conf

# Instalar extensões do PHP necessárias e dependências
RUN apt-get update && apt-get install -y \
    libpng-dev libjpeg-dev libfreetype6-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd \
    && docker-php-ext-install mysqli pdo pdo_mysql

# Configurar permissões corretas para Laravel
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache \
    && chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

# Configurar o nome do servidor para evitar aviso do Apache
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Habilitar mod_rewrite para Laravel
RUN a2enmod rewrite

# Expor a porta padrão do Apache
EXPOSE 80