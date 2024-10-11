# Laravel + Docker Setup no Linux

Este repositório contém as instruções para configurar um ambiente de desenvolvimento Laravel utilizando Docker no Linux. 

O Dockerfile e o docker-compose.yml estão prontos para uso com PHP 8.2, Apache, MySQL e PHPMyAdmin.

## Requisitos

Antes de começar, certifique-se de ter as seguintes ferramentas instaladas no seu sistema:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [PHP](https://www.php.net/manual/pt_BR/install.php)
- [Composer](https://getcomposer.org/)

### 1. Verificar se Docker está instalado:

```bash
docker --version
```

#### 1.1 Se o Docker não estiver instalado, você pode instalá-lo com os seguintes comandos:
```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 2. Verificar se Docker Compose está instalado:

```bash
docker-compose --version
```
#### 2.1 Se o Docker Compose não estiver instalado, você pode instalá-lo com o seguinte comando:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 3. Verificar se PHP está instalado:

```bash
php -v
```
#### 3.1 Se o PHP não estiver instalado, você pode instalá-lo com os seguintes comandos:

```bash
sudo apt update
sudo apt install php php-cli php-mbstring php-xml php-zip
```

### 4. Verificar se o Composer está instalado:

```bash
composer --version
```
#### 4.1 Se o Composer não estiver instalado, você pode instalá-lo com os seguintes comandos:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '48e75d36fe0b1341959a2e15c1c3af08ba7e9e7a11c6dcf13c1e4b2670ec9f1e1b406f450a6820bc474ec20d6d6c0e0f') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```
## Passos para Instalação

### 1. Clonar o repositório

Primeiro, clone o repositório para sua máquina local:

```bash
git clone https://github.com/isabela-tvitek/laravel-docker.git
cd laravel-docker
```

### 2. Estrutura do Projeto
Certifique-se de que seu diretório de projeto contém a seguinte estrutura:
```bash
.
├── .docker
│   └── vhost.conf        # Arquivo de configuração do Apache
├── example-app
│   └── ...               # Código-fonte do Laravel
├── docker-compose.yml    # Configuração do Docker Compose
└── Dockerfile            # Arquivo Docker para Laravel + Apache
```

### 3. Dockerfile
No arquivo Dockerfile, configuramos a imagem base do PHP com Apache e as dependências necessárias para Laravel:
```bash
# Usar a imagem oficial do PHP com Apache
FROM php:8.2-apache

# Definir o diretório de trabalho
WORKDIR /var/www/html

# Copiar o código-fonte e configuração do Apache
COPY . /var/www/html
COPY .docker/vhost.conf /etc/apache2/sites-available/000-default.conf

# Instalar extensões do PHP necessárias
RUN apt-get update && apt-get install -y libpng-dev libjpeg-dev libfreetype6-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd mysqli pdo pdo_mysql

# Configurar permissões corretas para Laravel
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache \
    && chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

# Configurar nome do servidor e habilitar mod_rewrite
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf && a2enmod rewrite

# Expor a porta do Apache
EXPOSE 80
```

### 4. Configuração do Docker Compose
arquivo docker-compose.yml orquestra os serviços:

- Laravel (PHP + Apache)
- MySQL
- PHPMyAdmin

```bash
services:
  laravel:
    build:
      context: ./example-app
    ports:
      - "8081:80"
    networks:
      - example-network
    depends_on:
      - db
    volumes:
      - ./example-app:/var/www/html
      - ./example-app/vendor:/var/www/html/vendor:delegated
    command: |
      sh -c "chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache && chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache && apache2-foreground"

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    ports:
      - "3307:3306"
    networks:
      - example-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    depends_on:
      - db
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: root
    ports:
      - "8082:80"
    networks:
      - example-network

networks:
  example-network:
    driver: bridge
```

### 5. Iniciar os Contêineres
Agora você está pronto para rodar o Docker Compose e iniciar os contêineres:

```bash
docker-compose up --build
```

### 6. Acessar os Serviços
- Laravel: http://localhost:8081
- PHPMyAdmin: http://localhost:8082
  - Usuário MySQL: root
  - Senha MySQL: root

### 7. Configuração Final do Laravel (opcional)
Após a criação dos contêineres, entre no contêiner do Laravel para instalar as dependências:

```bash
docker exec -it seu-repositorio_laravel_1 bash
composer install
```

### 8. Configuração do Banco de Dados
No arquivo .env do Laravel (dentro do contêiner), configure o banco de dados:

```bash
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=user
DB_PASSWORD=userpassword
```

### 9. Finalizar
Agora o Laravel está rodando no Docker, com MySQL e PHPMyAdmin disponíveis. Acesse http://localhost:8081 para ver seu projeto Laravel funcionando.

### Comandos Úteis
- Iniciar os contêineres: docker-compose up --build
- Parar os contêineres: docker-compose down
- Acessar o contêiner do Laravel: docker exec -it seu-repositorio_laravel_1 bash
