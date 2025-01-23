# Zabbix MySQL encryption 
Zabbix MySQL encryption using Dockerfiles and Docker Compose. This requires CA, keys and certs. Since I don't know what the names of your CA, keys and certs are going to be, you may have to edit the Dockerfiles and my.cnfs to specify the correct names and locations you want to put them in the containers.

## Docker Compose, .env and init.sql files
.env
```
# Zabbix Frontend
DB_SERVER_HOST="mysql-server IP or DNS"
MYSQL_USER="zabbix"
MYSQL_PASSWORD="password"
ZBX_SERVER_HOST="zabbix-server IP or DNS"
PHP_TZ="Time/Zone"
ZBX_DB_KEY_FILE="/var/lib/zabbix/web-key.pem"
ZBX_DB_CERT_FILE="/var/lib/zabbix/web-cert.pem"
ZBX_DB_CA_FILE="/var/lib/zabbix/ca.pem"

# Zabbix Server
ZBX_DBTLSCERTFILE="/var/lib/zabbix/ssl/certs/client-cert.pem"
ZBX_DBTLSKEYFILE="/var/lib/zabbix/ssl/keys/client-key.pem"
ZBX_DBTLSCAFILE="/var/lib/zabbix/ssl/ssl_ca/ca.pem"
DBTLSCIPHER13="TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256"
# ZBX_ALLOWUNSUPPORTEDDBVERSIONS=1 <- Remove comment if using unsupported MySQL version

# MySQL
MYSQL_ROOT_PASSWORD="password"
MYSQL_DATABASE="zabbix"
MYSQL_USER="zabbix"
MYSQL_PASSWORD="password"
```

docker-compose.yaml
```yaml
services:
  zabbix-frontend:
    image: <user>/zabbix-web-apache-mysql
    build:
      context: ./zabbix-web/
    depends_on:
      - zabbix-server
    container_name: zabbix-frontend
    restart: unless-stopped
    ports:
      - "80:8080"
      - "443:8443"
    environment:
      - DB_SERVER_HOST=${DB_SERVER_HOST}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - ZBX_SERVER_HOST=${ZBX_SERVER_HOST}
      - PHP_TZ=${PHP_TZ}
      - ZBX_DB_ENCRYPTION=true
      - ZBX_DB_KEY_FILE=${ZBX_DB_KEY_FILE}
      - ZBX_DB_CERT_FILE=${ZBX_DB_CERT_FILE}
      - ZBX_DB_CA_FILE=${ZBX_DB_CA_FILE}
      - ZBX_DB_VERIFY_HOST=true
    networks:
      zabbix-net:
        ipv4_address: <IP>

  zabbix-server:
    image: <user>/zabbix-server-mysql
    build:
      context: ./zabbix-server-mysql/
    depends_on:
      - mysql-server
    container_name: zabbix-server
    restart: unless-stopped
    environment:
      - DB_SERVER_HOST=${DB_SERVER_HOST}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - ZBX_DBTLSCONNECT=verify_full
      - ZBX_DBTLSCAFILE=${ZBX_DBTLSCAFILE}
      - ZBX_DBTLSCERTFILE=${ZBX_DBTLSCERTFILE}
      - ZBX_DBTLSKEYFILE=${ZBX_DBTLSKEYFILE}
      - DBTLSCIPHER13=${DBTLSCIPHER13}
      - ZBX_ALLOWUNSUPPORTEDDBVERSIONS=${ZBX_ALLOWUNSUPPORTEDDBVERSIONS}
    # networks:
      # zabbix-net:
        # ipv4_address: <IP>

  mysql-server:
    image: <user>/mysql-server
    build:
      context: ./mysql-server/
    container_name: mysql-server
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    command: --log-bin-trust-function-creators=ON
    # networks:
      # zabbix-net:
        # ipv4_address: <IP>
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

# networks:
  # zabbix-net:

volumes:
  mysql-data:
```
init.sql
```sql
DROP DATABASE zabbix;
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER USER 'zabbix'@'%' REQUIRE X509;
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'%';
FLUSH PRIVILEGES;
```
