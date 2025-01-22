FROM mysql:9

# Copy CA, key, cert and my.cnf
COPY ./certs/* /etc/mysql/conf.d/certs/
COPY ./conf/my.cnf /etc/my.cnf
# Assign permissions and ownership

RUN chmod 444 /etc/mysql/conf.d/certs/server-cert.pem
RUN chmod 400 /etc/mysql/conf.d/certs/server-key.pem
RUN chmod 444 /etc/mysql/conf.d/certs/ca.pem

RUN chown mysql:mysql /etc/mysql/conf.d/certs/server-cert.pem
RUN chown mysql:mysql /etc/mysql/conf.d/certs/ca.pem
RUN chown mysql:mysql /etc/mysql/conf.d/certs/server-key.pem

# Expose 3306
EXPOSE 3306

# Run mysqld
CMD ["mysqld"]
