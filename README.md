## Introduction
[OpenLDAP Software](https://www.openldap.org/) is an open source implementation of the Lightweight Directory Access Protocol.

This demo uses [bitnami OpenLDAP](https://github.com/bitnami/bitnami-docker-openldap) to speed up the test, including:

1. Add a new demo entry with customized objectClass and customized attributes

2. Configurate `memberof overlay` and `ldap replication`

## Installation & Run
- install openldap and run it
```sh
docker pull bitnami/openldap

docker run -p 127.0.0.1:1389:1389/tcp --name openldap --env LDAP_ORGANISATION=redditcoll.com --env LDAP_ROOT=dc=redditcoll,dc=com --env LDAP_ADMIN_USERNAME=admin --env LDAP_ADMIN_PASSWORD=adminpassword --env LDAP_CONFIG_ADMIN_ENABLED=yes --env LDAP_CONFIG_ADMIN_USERNAME=config --env LDAP_CONFIG_ADMIN_PASSWORD=configpassword bitnami/openldap
``` 

- Log in container by root
```sh 
# check container ID
docker container ls

# open container terminal
docker exec -it -u 0 {container_id} bash

# create folder
mkdir /opt/bitnami/openldap/etc/redditcoll-file
mkdir /tmp/ldap_config
``` 

- Connection info
```
localhost:1389

# as admin
cn=admin,dc=redditcoll,dc=com
adminpassword

# as config admin
cn=config,cn=config
configpassword

# as user
dc=redditcoll,dc=com
cn=user01,ou=users,dc=a4,dc=com
bitnami1
```

## Configuration
- Copy config files into container and execute them
```sh
# copy config files
docker cp config.ldif {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/config.ldif
docker cp memberof_modify.ldif {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/memberof_modify.ldif
docker cp memberof_add.ldif {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/memberof_add.ldif
docker cp replication_test_01.ldif {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/replication_test_01.ldif

# execute by config admin
ldapadd -x -D "cn=config,cn=config" -w configpassword -H "ldapi:///" -f "/opt/bitnami/openldap/etc/redditcoll-file/config.ldif"
ldapmodify -x -D "cn=config,cn=config" -w configpassword -H "ldapi:///" -f "/opt/bitnami/openldap/etc/redditcoll-file/memberof_modify.ldif"
ldapadd -x -D "cn=config,cn=config" -w configpassword -H "ldapi:///" -f "/opt/bitnami/openldap/etc/redditcoll-file/memberof_add.ldif"
ldapadd -x -D "cn=config,cn=config" -w configpassword -H "ldapi:///" -f "/opt/bitnami/openldap/etc/redditcoll-file/replication_test_01.ldif"

# test memberof
docker cp memberof_test.ldif {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/memberof_test.ldif
ldapadd -x -D "cn=admin,dc=redditcoll,dc=com" -w adminpassword -H "ldap://127.0.0.1:1389" -f "/opt/bitnami/openldap/etc/redditcoll-file/memberof_test.ldif"
ldapsearch -x -D "cn=admin,dc=redditcoll,dc=com" -w adminpassword -H "ldap://127.0.0.1:1389" -b "dc=redditcoll,dc=com" memberOf 
```

- Copy customized schema files
```sh
# copy demo schema files
docker cp demo.schema {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/demo.schema
docker cp demo.ldif {container_id}:/opt/bitnami/openldap/etc/redditcoll-file/demo.ldif

# copy conf file
docker cp ldap.conf {container_id}:/opt/bitnami/openldap/etc/ldap.conf
```

- Install the schema
```sh
# install schema
slaptest -f /opt/bitnami/openldap/etc/ldap.conf -F /tmp/ldap_config

# valid check
ls /tmp/ldap_config/cn\=config
ls /tmp/ldap_config/cn\=config/cn\=schema

# copy ldif to slapd.d
cp /tmp/ldap_config/cn\=config/cn\=schema/cn={4}demo.ldif /opt/bitnami/openldap/etc/slapd.d/cn\=config/cn\=schema/
chmod 777 /opt/bitnami/openldap/etc/slapd.d/cn\=config/cn\=schema/cn={4}demo.ldif

# Restart container

# test add entry
ldapadd -x -D "cn=admin,dc=redditcoll,dc=com" -w adminpassword -H "ldap://127.0.0.1:1389" -f "/opt/bitnami/openldap/etc/redditcoll-file/demo.ldif"
```

- Test OpenLDAP Replication
```sh
# find container IP
docker ps
docker network ls
docker inspect {container_id} # e.g. "IPAddress": "172.17.0.2"

# setup same schema and with different replication configuration (e.g. replication_test_xx.ldif) in another two containers (e.g. 172.17.0.3 & 172.17.0.4).
# try to add some entries to see if the data is synchronized.
```

## Maintenance by Web
- Use `phpLDAPadmin` to view and edit ldap data, [installation example](./Install_phpAdmin.MD)