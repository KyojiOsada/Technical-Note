# 1. Install
## 1-1. RHEL-7 系
### 1-1-1. yum
`$ yum -y install httpd mod_ssl`

### 1-1-2. Fedora Package
http://repos.fedorapeople.org/repos/jkaluza/httpd24/epel-httpd24.repo<br>
↓<br>
http://l-light-note.hatenablog.com/entry/2015/08/10/095602

### 1-1-3. IUS Package
`$ rpm -Uvh "https://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/ius-release-1.0-14.ius.centos6.noarch.rpm"`<br>
↓<br>
http://htnosm.hatenablog.com/entry/2015/08/30/234000

## 1-2. RHEL-6 系


## 1-3. elinks
_for Zabbix httpd monitoring_<br>
`$ yum -y install elinks`

### Check Command
`$ apachectl fullstatus`<br>

### Case: Not Insatlled
`The 'links' package is required for this functionality.`

# 2. Configuration
## 2-1. /etc/httpd/conf/httpd.conf
`$ cd /etc/httpd/conf/`<br>
`$ cp -a httpd.conf httpd.conf.org`<br>
`$ vim httpd.conf`

    ServerAdmin admin
    ServerName example.lan:80
    Require all denied

    # To json like for log analysis.<br>
    LogFormat "\n{\"Request-Time\": \"%{%Y-%m-%d %H:%M:%S}t\", \"Status\": \"%>s\", \"Remote-Addr\": \"%a\", \"Remote-Host\": \"%h\", \"Server-Addr\": \"%A\", \"Server-Host\": \"%v\", \"Protocol\": \"%H\", \"Method\": \"%m\", \"URL\": \"%U\", \"Query\": \"%q\", \"Referer\": \"%{Referer}i\", \"User-Agent\": \"%{User-Agent}i\", \"Byte\": \"%b\", \"Speed\": \"%T\"}," access
    LogFormat "\n{\"Request-Time\": \"%{%Y-%m-%d %H:%M:%S}t\", \"Status\": \"%>s\", \"Remote-Addr\": \"%a\", \"Remote-Host\": \"%h\", \"Server-Addr\": \"%A\", \"Server-Host\": \"%v\", \"Protocol\": \"%H\", \"Method\": \"%m\", \"URL\": \"%U\", \"Query\": \"%q\", \"Referer\": \"%{Referer}i\", \"User-Agent\": \"%{User-Agent}i\", \"Byte\": \"%b\", \"Speed\": \"%T\", \"SSL-Protocol\": \"%{SSL_PROTOCOL}x\", \"SSL-Cipher\": \"%{SSL_CIPHER}x\"}," ssl_access

## 2-2. /etc/httpd/common/
_for common configuration_<br>
`$ cd /etc/httpd`<br>
`$ mkdir common`<br>

## 2-3. /etc/httpd/common/options.conf
`$ cd /etc/httpd/common`<br>
`$ touch options.conf`<br>
`$ vim options.conf`

    Options -Indexes +FollowSymlinks
    AllowOverride All

## 2-4. /etc/httpd/common/basic_auth.conf
`$ cd /etc/httpd/common`<br>
`$ touch basic_auth.conf`<br>
`$ vim basic_auth.conf`

    AuthType Basic
    AuthName "Authentication Required"
    AuthUserFile /etc/httpd/.htpasswd
    Require valid-user

    SetEnvIf User-Agent "Zabbix 2.4.7" grant_ua
    Require env grant_ua

## 2-5. /etc/httpd/common/redirect_https.conf
`$ cd /etc/httpd/common`<br>
`$ touch redirect_https.conf`<br>
`$ vim basic_auth.conf`

    RewriteEngine on
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)?$ https://%{HTTP_HOST}$1 [R=301,L]

## 2-6. /etc/httpd/common/require_lan.conf
`$ cd /etc/httpd/common`<br>
`$ touch require_lan.conf`<br>
`$ vim require_lan.conf`

    # Loopback
    Require ip 127.0.0.0/8
    Require ip ::1
    # Lan
    Require ip 10.0.0.0/8
    Require ip 172.16.0.0/12
    Require ip 192.168.0.0/16
    # Trusted
    Require ip xxx.xxx.xxx.xxx/24

## 2-7. /etc/httpd/common/tls_wildcard.examploc.com.conf
`$ cd /etc/httpd/common`<br>
`$ touch tls_wildcard.example.com.conf`<br>
`$ vim tls_wildcard.example.com.conf`

    SSLEngine on
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:!SSLv3:!RC4:+HIGH:!MEDIUM:!LOW
    Header set Strict-Transport-Security "max-age=315360000; includeSubDomains"

    SSLCertificateFile /data/tls/example.com/wildcard/pigment.tokyo.YYYYMMDD.crt
    SSLCertificateKeyFile /data/tls/example.com/wildcard/pigment.tokyo.YYYYMMDD.key
    SSLCertificateChainFile /data/tls/example.com/wildcard/pigment.tokyo.YYYYMMDD.inter.crt

    <Files ~ "\.(cgi|shtml|phtml|php3?)$">
        SSLOptions +StdEnvVars
    </Files>

    <Directory "/data/www/cgi-bin">
        SSLOptions +StdEnvVars
    </Directory>

    SetEnvIf User-Agent ".*MSIE.*" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0

## 2-8. /etc/httpd/common/tls_localhost.conf
`$ cd /etc/httpd/common`<br>
`$ touch tls_localhost.conf`<br>
`$ vim tls_localhost.conf`

    SSLEngine on
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:!SSLv3:!RC4:+HIGH:!MEDIUM:!LOW
    Header set Strict-Transport-Security "max-age=315360000; includeSubDomains"

    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key

    <Files ~ "\.(cgi|shtml|phtml|php3?)$">
        SSLOptions +StdEnvVars
    </Files>

    <Directory "/data/www/cgi-bin">
        SSLOptions +StdEnvVars
    </Directory>

    SetEnvIf User-Agent ".*MSIE.*" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0

## 2-9. /etc/httpd/conf.d/
_Deactivate unnecessary configuration files for performance improvement._<br>
`$ cd /etc/httpd/conf.d/`
`$ mv example.conf example.conf.nouse`

### Example
    autoindex.conf.nouse  php56-php.conf  ssl.conf      status.conf         vhosts.conf
    _.conf                README          ssl.conf.org  userdir.conf.nouse  welcome.conf.nouse

## 2-10. /etc/httpd/conf.d/_.conf
_Configuration file for directives that are no longer defaulted in AAA or all common additional directives._<br>
`$ touch /etc/httpd/conf.d/_.conf`<br>
`$ vim /etc/httpd/conf.d/_.conf`<br>

    # For Security
    ServerTokens ProductOnly
    ServerSignature Off
    TraceEnable Off

    # For Performance
    FileEtag None
    RequestHeader unset If-Modified-Since
    RequestHeader unset If-None-Match
    Header set Cache-Control no-store

    # For Compress
    SetOutputFilter DEFLATE
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/xml
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/javascript
    AddOutputFilterByType DEFLATE application/xhtml+xml
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE application/rss+xml
    AddOutputFilterByType DEFLATE application/atom_xml
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/x-javascript
    AddOutputFilterByType DEFLATE application/x-httpd-php
    AddOutputFilterByType DEFLATE application/json

    # For Process
    ## default 5
    StartServers 8
    ## default 5
    MinSpareServers 8
    ## default 10
    MaxSpareServers 16
    ## default 256
    ServerLimit 256
    ## default 256
    MaxClients 256
    ## default
    #MaxRequestsPerChild

    # For Network
    ## default Off
    HostNameLookups Off
    ## default On
    KeepAlive On
    ## default 5
    KeepAliveTimeout 5
    ## default 100
    MaxKeepAliveRequests 1024
    ## default 60
    Timeout 20

## 2-11. /etc/httpd/conf.d/vhosts.conf
_For Port 80_<br>
`$ touch /etc/httpd/conf.d/vhosts.conf`<br>
`$ vim /etc/httpd/conf.d/vhosts.conf`<br>

_A NameVirtualHost Directive is abolished._<br>

    # default
    <VirtualHost *:80>
        ServerName _default_
        ErrorLog "| /usr/sbin/rotatelogs /data/log/default/httpd_error_%Y%m%d.log 86400 540"
        LogLevel error

        <Directory "/data/www/default/">
	    Include common/options.conf
            Include common/require_lan.conf
        </Directory>
    </VirtualHost>

    # www.example.com
    <VirtualHost *:80>
        ServerName www.example.com
        ServerAlias example.com
        ErrorLog "| /usr/sbin/rotatelogs /data/log/www.example.com/httpd_error_%Y%m%d.log 86400 540"
        LogLevel error

        Include common/redirect_https.conf
    </VirtualHost>

    # img.example.tokyo
    <VirtualHost *:80>
        ServerName img.example.com
        ErrorLog "| /usr/sbin/rotatelogs /data/log/img.example.com/httpd_error_%Y%m%d.log 86400 540"
        LogLevel error

        ProxyRequests Off
        ProxyPass /img/ http://s3-ap-northeast-1.amazonaws.com/example.com/
        ProxyPassReverse /img/ http://s3-ap-northeast-1.amazonaws.com/example.com/

        <Proxy *>
            Require all granted
        </Proxy>
    </VirtualHost>

    # localhost for Zabbix
    <VirtualHost *:80>
        ServerName localhost
        ServerAlias 127.0.0.1
        DocumentRoot "/data/www/localhost/"
        ErrorLog "| /usr/sbin/rotatelogs /data/log/localhost/httpd_error_%Y%m%d.log 86400 540"
        CustomLog "| /usr/sbin/rotatelogs /data/log/localhost/httpd_access_%Y%m%d.log 86400 540" access
        CustomLog "| /usr/sbin/rotatelogs /data/log/localhost/httpd_combined_%Y%m%d.log 86400 540" combined
        LogLevel warn

        <Directory "/data/www/localhost/">
            Include common/options.conf
            Include common/require_lan.conf
        </Directory>
    </VirtualHost>

## 2-12. /etc/httpd/conf.d/ssl.conf
_For Port 443_<br>
`$ vim /etc/httpd/conf.d/ssl.conf`<br>

    ##
    ##  SSL Global Context
    ##

    Listen *:443 https
    SSLPassPhraseDialog exec:/usr/libexec/httpd-ssl-pass-dialog
    SSLSessionCache         shmcb:/run/httpd/sslcache(512000)
    SSLSessionCacheTimeout  300
    SSLRandomSeed startup file:/dev/urandom  256
    SSLRandomSeed connect builtin
    #SSLRandomSeed startup file:/dev/random  512
    #SSLRandomSeed connect file:/dev/random  512
    #SSLRandomSeed connect file:/dev/urandom 512
    SSLCryptoDevice builtin
    #SSLCryptoDevice ubsec

    # added
    SSLHonorCipherOrder On
    SSLStaplingCache shmcb:/tmp/stapling_cache(128000)
    SSLStrictSNIVHostCheck Off

    ##
    ## SSL Virtual Host Context
    ##

    # default
    <VirtualHost *:443>
        ServerName _default_:443
        DocumentRoot "/data/www/default/"
        ErrorLog "| /usr/sbin/rotatelogs /data/log/default/httpd_error_%Y%m%d.log 86400 540"
        CustomLog "| /usr/sbin/rotatelogs /data/log/default/httpd_access_%Y%m%d.log 86400 540" ssl_access
        CustomLog "| /usr/sbin/rotatelogs /data/log/default/httpd_combined_%Y%m%d.log 86400 540" combined
        LogLevel warn

        Include common/tls_localhost.conf

        <Directory "/data/www/default/">
            Include common/options.conf
            Include common/require_lan.conf
        </Directory>
    </VirtualHost>

    # www.examplo.com
    <VirtualHost *:443>
        ServerName www.example.com:443
        DocumentRoot "/data/www/www.example.com/app/puglic/"
        ErrorLog "| /usr/sbin/rotatelogs /data/log/www.example.com/httpd_error_%Y%m%d.log 86400 540"
        CustomLog "| /usr/sbin/rotatelogs /data/log/www.example.com/httpd_access_%Y%m%d.log 86400 540" ssl_access
        CustomLog "| /usr/sbin/rotatelogs /data/log/www.example.com/httpd_combined_%Y%m%d.log 86400 540" combined
        LogLevel warn

        Include common/tls_wildcard.example.com.conf

        <Directory "/data/www/www.example.com/">
            Include common/options.conf
            Require all granted
        </Directory>
    </VirtualHost>

    # img.example.com
    <VirtualHost *:443>
        ServerName img.example.com:443
        ErrorLog "| /usr/sbin/rotatelogs /data/log/img.example.com/httpd_error_%Y%m%d.log 86400 540"
        LogLevel error

        Include inc/tls_wildcard.example.com.conf

        SSLProxyEngine On
        ProxyRequests Off
        ProxyPass /img/ https://s3-ap-northeast-1.amazonaws.com/example.com/
        ProxyPassReverse /img/ https://s3-ap-northeast-1.amazonaws.com/example.com/
	
        <Proxy *>
            Require all granted
        </Proxy>
    </VirtualHost>

    # localhost for Zabbix
    <VirtualHost *:443>
        ServerName localhost:443
        ServerAlias 127.0.0.1
        DocumentRoot "/data/www/localhost/"
        ErrorLog "| /usr/sbin/rotatelogs /data/log/localhost/httpd_error_%Y%m%d.log 86400 540"
        CustomLog "| /usr/sbin/rotatelogs /data/log/localhost/httpd_access_%Y%m%d.log 86400 540" access
        CustomLog "| /usr/sbin/rotatelogs /data/log/localhost/httpd_combined_%Y%m%d.log 86400 540" combined
        LogLevel warn

        Include inc/tls_localhost.conf

        <Directory "/data/www/localhost/">
            Include common/options.conf
            Include common/require_lan.conf
        </Directory>
    </VirtualHost>

## 2-13. /etc/httpd/conf.modules.d/
_Deactivate unnecessary configuration files for performance improvement._<br>
`$ cd /etc/httpd/conf.modules.d/`
`$ mv example.conf example.conf.nouse`<br>

### Example
    00-base.conf      00-dav.conf.nouse  00-mpm.conf    00-ssl.conf      01-cgi.conf.nouse
    00-base.conf.org  00-lua.conf.nouse  00-proxy.conf  00-systemd.conf  10-php56-php.conf

## 2-14. /etc/httpd/conf.modules.d/00-base.conf
`$ cd /etc/httpd/conf.modules.d/`
`$ cp -a 00-base.conf 00-base.conf.org`
`$ vim 00-base.conf`

### Basic Auth Module
    LoadModule auth_basic_module modules/mod_auth_basic.so
    LoadModule authn_file_module modules/mod_authn_file.so
    LoadModule authn_core_module modules/mod_authn_core.so
    LoadModule authz_user_module modules/mod_authz_user.so
    LoadModule authz_groupfile_module modules/mod_authz_groupfile.so

### Example
    LoadModule access_compat_module modules/mod_access_compat.so
    LoadModule actions_module modules/mod_actions.so
    LoadModule alias_module modules/mod_alias.so
    LoadModule allowmethods_module modules/mod_allowmethods.so
    LoadModule auth_basic_module modules/mod_auth_basic.so
    #LoadModule auth_digest_module modules/mod_auth_digest.so
    #LoadModule authn_anon_module modules/mod_authn_anon.so
    LoadModule authn_core_module modules/mod_authn_core.so
    #LoadModule authn_dbd_module modules/mod_authn_dbd.so
    #LoadModule authn_dbm_module modules/mod_authn_dbm.so
    LoadModule authn_file_module modules/mod_authn_file.so
    #LoadModule authn_socache_module modules/mod_authn_socache.so
    LoadModule authz_core_module modules/mod_authz_core.so
    LoadModule authz_dbd_module modules/mod_authz_dbd.so
    LoadModule authz_dbm_module modules/mod_authz_dbm.so
    LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
    LoadModule authz_host_module modules/mod_authz_host.so
    LoadModule authz_owner_module modules/mod_authz_owner.so
    LoadModule authz_user_module modules/mod_authz_user.so
    #LoadModule autoindex_module modules/mod_autoindex.so
    #LoadModule cache_module modules/mod_cache.so
    #LoadModule cache_disk_module modules/mod_cache_disk.so
    #LoadModule data_module modules/mod_data.so
    #LoadModule dbd_module modules/mod_dbd.so
    LoadModule deflate_module modules/mod_deflate.so
    LoadModule dir_module modules/mod_dir.so
    #LoadModule dumpio_module modules/mod_dumpio.so
    #LoadModule echo_module modules/mod_echo.so
    LoadModule env_module modules/mod_env.so
    LoadModule expires_module modules/mod_expires.so
    LoadModule ext_filter_module modules/mod_ext_filter.so
    LoadModule filter_module modules/mod_filter.so
    LoadModule headers_module modules/mod_headers.so
    #LoadModule include_module modules/mod_include.so
    LoadModule info_module modules/mod_info.so
    LoadModule log_config_module modules/mod_log_config.so
    LoadModule logio_module modules/mod_logio.so
    LoadModule mime_magic_module modules/mod_mime_magic.so
    LoadModule mime_module modules/mod_mime.so
    #LoadModule negotiation_module modules/mod_negotiation.so
    LoadModule remoteip_module modules/mod_remoteip.so
    LoadModule reqtimeout_module modules/mod_reqtimeout.so
    LoadModule rewrite_module modules/mod_rewrite.so
    LoadModule setenvif_module modules/mod_setenvif.so
    LoadModule slotmem_plain_module modules/mod_slotmem_plain.so
    LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
    #LoadModule socache_dbm_module modules/mod_socache_dbm.so
    #LoadModule socache_memcache_module modules/mod_socache_memcache.so
    LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
    LoadModule status_module modules/mod_status.so
    #LoadModule substitute_module modules/mod_substitute.so
    #LoadModule suexec_module modules/mod_suexec.so
    LoadModule unique_id_module modules/mod_unique_id.so
    LoadModule unixd_module modules/mod_unixd.so
    #LoadModule userdir_module modules/mod_userdir.so
    LoadModule version_module modules/mod_version.so
    LoadModule vhost_alias_module modules/mod_vhost_alias.so

    #LoadModule buffer_module modules/mod_buffer.so
    #LoadModule watchdog_module modules/mod_watchdog.so
    #LoadModule heartbeat_module modules/mod_heartbeat.so
    #LoadModule heartmonitor_module modules/mod_heartmonitor.so
    #LoadModule usertrack_module modules/mod_usertrack.so
    #LoadModule dialup_module modules/mod_dialup.so
    #LoadModule charset_lite_module modules/mod_charset_lite.so
    #LoadModule log_debug_module modules/mod_log_debug.so
    #LoadModule ratelimit_module modules/mod_ratelimit.so
    #LoadModule reflector_module modules/mod_reflector.so
    #LoadModule request_module modules/mod_request.so
    #LoadModule sed_module modules/mod_sed.so
    #LoadModule speling_module modules/mod_speling.so

# 3. Basic Auth
## 3-1. Initialize basic auth
`$ htpasswd -c /etc/httpd/.htpasswd <USER-NAME>`

## 3-2. Add a user or modify a user
`$ htpasswd /etc/httpd/.htpasswd <USER-NAME>`

## 3-3. Delete a user
`$ htpasswd -D /etc/httpd/.htpasswd <USER-NAME>`

## 3-4. Configure basic auth
    <Directory "/hoge">
        AuthType Basic
        AuthName "Authentication required"
        AuthUserFile /etc/httpd/.htpasswd
        Require valid-user
    </Directory>

# 4. Location
## 4-1. When the entity directory does not exist
### 4-1-1. Apply only subdirectories
    <Directory "/data/www/www.example.com">
        Include common/options.conf
        Require all granted
    </Directory>
    <LocationMatch "/example/*">
        Include common/basic_auth.conf
    </LocationMatch>

### 4-1-2. Remove only subdirectories
    <Directory "/data/www/www.example.com">
        Include common/options.conf
        Require all granted
    </Directory>
    <LocationMatch "/example/*">
        Satisfy Any
    </LocationMatch>

