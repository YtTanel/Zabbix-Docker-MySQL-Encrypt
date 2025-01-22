FROM zabbix/zabbix-web-apache-mysql:latest

# Copy CA, key, cert and my.cnf
COPY ./certs/* /var/lib/zabbix/
COPY ./https/* /etc/ssl/apache2/
COPY ./conf/my.cnf /etc/my.cnf
# Assign permissions and ownership

USER root

RUN chmod 444 /var/lib/zabbix/web-cert.pem
RUN chmod 400 /var/lib/zabbix/web-key.pem
RUN chmod 444 /var/lib/zabbix/ca.pem

RUN chown zabbix:zabbix /var/lib/zabbix/web-cert.pem
RUN chown zabbix:zabbix /var/lib/zabbix/web-key.pem
RUN chown zabbix:zabbix /var/lib/zabbix/ca.pem
RUN chown zabbix:zabbix /etc/ssl/apache2/ssl.crt
RUN chown zabbix:zabbix /etc/ssl/apache2/ssl.key

USER zabbix

EXPOSE 8080 8443

ENTRYPOINT ["docker-entrypoint.sh"]
