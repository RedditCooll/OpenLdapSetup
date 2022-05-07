## Install phpLDAPadmin
Use [osixia phpLDAPadmin](https://github.com/osixia/docker-phpLDAPadmin) to view and edit ldap data by web. 

- install phpLDAPadmin
```sh
docker pull osixia/phpldapadmin
```

- run phpLDAPadmin
```sh
# run OpenLDAP and then run phpLDAPadmin
docker run -p 6443:80 \
        --name phpLDAPadmin \
        --link openldap \
        --env PHPLDAPADMIN_HTTPS=false \
        --env PHPLDAPADMIN_LDAP_HOSTS=ldap://172.17.0.2:1389 \
        --detach osixia/phpldapadmin:latest
```

- open `http://localhost:6443` in your browser, and log in to check the data inside.
```
# as admin
cn=admin,dc=redditcoll,dc=com
adminpassword
```