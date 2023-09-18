FROM httpd:alpine

# These variables are inherited from the httpd:alpine image:
# ENV HTTPD_PREFIX /usr/local/apache2
# WORKDIR "$HTTPD_PREFIX"

# Copy in our configuration files.
COPY 2.4/conf/ conf/

RUN set -ex; \
    # Create empty default DocumentRoot.
    mkdir -p "/var/www/html"; \
    # Create directories for Dav data and lock database.
    mkdir -p "/var/lib/dav/data"; \
    touch "/var/lib/dav/DavLock"; \
    chown -R www-data:www-data "/var/lib/dav"; \
    \
    # Enable DAV modules.
    for i in dav dav_fs; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "conf/httpd.conf"; \
    done; \
    \
    # Make sure authentication modules are enabled.
    for i in authn_core authn_file authz_core authz_user auth_basic auth_digest; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "conf/httpd.conf"; \
    done; \
    \
    # Make sure other modules are enabled.
    for i in alias headers mime setenvif; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "conf/httpd.conf"; \
    done; \
    \
    # Run httpd as "www-data" (instead of "daemon").
    for i in User Group; do \
        sed -i -e "s|^$i .*|$i www-data|" "conf/httpd.conf"; \
    done; \
    \
    # Include enabled configs and sites.
    printf '%s\n' "Include conf/conf-enabled/*.conf" \
        >> "conf/httpd.conf"; \
    printf '%s\n' "Include conf/sites-enabled/*.conf" \
        >> "conf/httpd.conf"; \
    \
    # Enable dav and default site.
    mkdir -p "conf/conf-enabled"; \
    mkdir -p "conf/sites-enabled"; \
    ln -s ../conf-available/dav.conf "conf/conf-enabled"; \
    ln -s ../sites-available/default.conf "conf/sites-enabled"; \
    # Install openssl if we need to generate a self-signed certificate.
    apk add --no-cache openssl; \
    \
    # Commands from docker-entrypoint.sh
    sed -e "s|AuthType .*|AuthType Digest|" -i "/usr/local/apache2/conf/conf-available/dav.conf"; \
    touch "/user.passwd"; \
    HASH="`printf '%s' "hpco:webdav:ILoveHPCI" | md5sum | awk '{print $1}'`"; \
    printf '%s\n' "hpco:webdav:$HASH" > /user.passwd; \
    mkdir /usr/local/apache2/var/; \
    printf '%s\n' "hpco:webdav:$HASH" >  /usr/local/apache2/var/user.passwd; \
    mkdir -p "/var/lib/dav/data"; \
    touch "/var/lib/dav/DavLock"; \
    chown -R www-data:www-data "/var/lib/dav"; \
    \
    # Change port
    sed -E -i "s/Listen (.*)80/Listen \18080/" /usr/local/apache2/conf/httpd.conf; \
    \
    # Allow logs
    chmod -R 777 /usr/local/apache2/logs

#COPY 2.4/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
EXPOSE 8080/tcp 8443/tcp
#ENTRYPOINT [ "docker-entrypoint.sh" ]
CMD [ "httpd-foreground" ]
