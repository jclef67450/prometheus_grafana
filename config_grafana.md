# Configuration de Grafana  
Si vous faites comme moi, vous avez grafana en natif sur linux, le répertoire et donc `/etc/grafana`.  
il y a 2 fichiers à configurer:
1. grafana.ini
2. ldap.toml

## grafana.ini  ##
Chercher la section "Server", puis indiquez les parametres suivants :  
```
#################################### Server ####################################
[server]
# Protocol (http, https, h2, socket)
# DE BASE, LE PARAMETRE, EST HTTP
protocol = https 

# The ip address to bind to, empty will bind to all interfaces
;http_addr =

# The http port  to use
http_port = 3000

# The public facing domain name used to access grafana from a browser
# INDIQUEZ VOTRE DOMAINE
domain = XXXXXX.XXXX.FR

# Redirect to correct domain if host header does not match domain
# Prevents DNS rebinding attacks
;enforce_domain = false

# The full public facing url you use in browser, used for redirects and emails
# If you use reverse proxy and sub path specify full url (with sub path)
;root_url = %(protocol)s://%(domain)s:%(http_port)s/

# Serve Grafana from subpath specified in `root_url` setting. By default it is set to `false` for compatibility reasons.
;serve_from_sub_path = false

# Log web requests
;router_logging = false

# the path relative working path
;static_root_path = public

# enable gzip
;enable_gzip = false

# https certs & key file
# INDIQUEZ ICI VOS CHEMINS DE CERTIFICAT
cert_file = /etc/certificat/grafana/grafana.crt
cert_key = /etc/certificat/grafana/grafana.key
```  
Ensuite chercher la section "Database" et indiquez les éléments suivants pour synchroniser votre BDD:  
Je ne vous apprends rien, remplacer mes éléments par les vôtres.  
```
#################################### Database ####################################
[database]
# You can configure the database connection by specifying type, host, name, user and password
# as separate properties or as on string using the url properties.

# Either "mysql", "postgres" or "sqlite3", it's your choice
type = mysql
host = 127.0.0.1:3306
name = grafana
user = grafana
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password = MonMotDePasse
```  
Chercher la section "Auth LDAP" et compléter là:  
```
#################################### Auth LDAP ##########################
[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
allow_sign_up = true

# LDAP background sync (Enterprise only)
# Indiquez le temp en version crontab
;sync_cron = "0 2 * * *"
;active_sync_enabled = true
```
## ldap.toml  ##  
Ce fichier va initialiser votre serveur LDAP, j'ai mis par défaut les utilisateurs en viewer, vous pouvez bien sûr le remplacer par n'importe quel rôle.  
```
# To troubleshoot and get more log info enable ldap debug logging in grafana.ini
# [log]
# filters = ldap:debug
[[servers]]
# Ldap server host (specify multiple hosts space separated)
host = "MONSERVEURLDAP.FR"
#  Default port is 389 or 636 if use_ssl = true
port = 389
# Set to true if ldap server supports TLS
use_ssl = false
# Set to true if connect ldap server with STARTTLS pattern (create connection in insecure, then upgrade$
start_tls = false
# set to true if you want to skip ssl cert validation
ssl_skip_verify = true
# set to the path to your root CA certificate or leave unset to use system defaults
# root_ca_cert = "/path/to/certificate.crt"
# Authentication against LDAP servers requiring client certificates
# client_cert = "/path/to/client.crt"
# client_key = "/path/to/client.key"

# Search user bind dn
bind_dn = ""
# Search user bind password
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
bind_password = ''

# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"
#search_filter = "(&(uid=%{user})(objectClass=posixAccount))"

search_filter = "(uid=%s)"
# An array of base dns to search through
search_base_dns = ["ou=XX,ou=XX,dc=XX,dc=XX,dc=XX"]

## For Posix or LDAP setups that does not support member_of attribute you can define the below settings
## Please check grafana LDAP docs for examples
# group_search_filter = "(&(objectClass=posixGroup)(memberUid=%s))"
# group_search_base_dns = ["ou=groups,dc=grafana,dc=org"]
# group_search_filter_user_attribute = "uid"

group_search_filter = "(&(objectClass=posixGroup)(|(gidNumber=%{gid})(memberUID=%{user})))"
group_search_base_dns = ["ou=XX,ou=XX,dc=XX,dc=XX,dc=XX"]
group_search_filter_user_attribute = "cn"

# Specify names of the ldap attributes your ldap uses
[servers.attributes]
#name = "givenName"
surname = "sn"
username = "uid"
#member_of = "memberOf"
email =  "email"

# Map ldap groups to grafana org roles
#[[servers.group_mappings]]
#group_dn = "cn=admins,dc=grafana,dc=org"
#org_role = "Admin"
# To make user an instance admin  (Grafana Admin) uncomment line below
# grafana_admin = true
# The Grafana organization database id, optional, if left out the default org (id 1) will be used
# org_id = 1

#[[servers.group_mappings]]
#group_dn = "cn=users,dc=grafana,dc=org"
#org_role = "Editor"

[[servers.group_mappings]]
# If you want to match all (or no ldap groups) then you can use wildcard
group_dn = "*"
org_role = "Viewer"
```
