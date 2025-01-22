FROM zabbix/zabbix-server-mysql:latest

# Copy CA, key, cert
COPY ./ca/ca.pem /var/lib/zabbix/ssl/ssl_ca/
COPY ./keys/client-key.pem /var/lib/zabbix/ssl/keys/
COPY ./certs/client-cert.pem /var/lib/zabbix/ssl/certs/
COPY ./conf/my.cnf /etc/my.cnf
USER root

# Assign permissions and ownership
RUN chmod 444 /var/lib/zabbix/ssl/ssl_ca/ca.pem
RUN chmod 400 /var/lib/zabbix/ssl/keys/client-key.pem
RUN chmod 444 /var/lib/zabbix/ssl/certs/client-cert.pem

RUN chown zabbix:zabbix /var/lib/zabbix/ssl/ssl_ca/ca.pem
RUN chown zabbix:zabbix /var/lib/zabbix/ssl/keys/client-key.pem
RUN chown zabbix:zabbix /var/lib/zabbix/ssl/certs/client-cert.pem

USER zabbix

# Expose 10051
EXPOSE 10051
