# SELinux
Source : https://blog.microlinux.fr/selinux/

Changer de mode
```Bash
# getenforce
Enforcing
# setenforce 0
# getenforce
Permissive
# setenforce 1
# getenforce
Enforcing
```

Changer de mode ou type depuis le fichier de configuration
```Bash
# /etc/selinux/config
SELINUX=enforcing
SELINUXTYPE=targeted
```

Lorsque SELinux est activé – autrement dit, lorsqu’on passe du mode disabled à permissive ou enforcing, il faut impérativement songer à réétiqueter l’ensemble des fichiers du système. Pour ce faire, il suffit de créer un fichier vide .autorelabel à la racine du système de fichiers avant de redémarrer :

```Bash
# touch /.autorelabel
# reboot
```

L’option -Z me permet d’afficher le contexte de sécurité :

```Bash
# ls -Z
unconfined_u:object_r:httpd_sys_content_t:s0 index.html
```

Error logs Apache

```Bash
# cat /var/log/httpd/error_log
```

Errors liés à SELinux
```Bash
# Installer auparavant le package setroubleshoot-server
sealert -a /var/log/audit/audit.log | less
...appliquer les consignes de type: restorecon -Rv /var/www/html/....
```

La commande matchpathcon peux nous renseigner sur l’étiquette utilisée en fonction du répertoire :

```Bash
$ matchpathcon /var/www/html/
/var/www/html   system_u:object_r:httpd_sys_content_t:s0
$ matchpathcon /var/log/httpd/
/var/log/httpd  system_u:object_r:httpd_log_t:s0
$ matchpathcon /etc/httpd/conf/
```

Pour modifier le contexte par défaut d'une arborescence:

```Bash
# semanage fcontext -a -t httpd_sys_content_t '/srv/web(/.*)?'
# restorecon -Rv /srv/web/
```

L’option fcontext permet de gérer les contextes de sécurité SELinux.
L’option -a (--add) permet d’ajouter une entrée.
L’option -t (--type) spécifie le type, en l’occurrence httpd_sys_content_t.
L’utilisation de l’expression régulière '/srv/web(/.*)?' applique la directive récursivement sur toute l’arborescence.
Une fois que le contexte par défaut est défini, il faut l’appliquer avec restorecon.

Booleans

```Bash
*****  Plugin catchall_boolean (32.5 confidence) suggests   *****
If you want to allow httpd to enable homedirs
Then you must tell SELinux about this by enabling the 
'httpd_enable_homedirs' boolean. Do
setsebool -P httpd_enable_homedirs 1


# getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off

# semanage boolean -l | grep httpd | sort | less
httpd_anon_write        (off, off) Allow httpd to anon write
httpd_builtin_scripting (on, on)   Allow httpd to builtin scripting
httpd_can_check_spam    (off, off) Allow httpd to can check spam
httpd_can_connect_ftp   (off, off) Allow httpd to can connect ftp
httpd_can_connect_ldap  (off, off) Allow httpd to can connect ldap

# setsebool -P httpd_enable_homedirs on
Notez qu’ici l’option -P rend la directive permanente, c’est-à-dire qu’elle est conservée après un redémarrage du système.

Et pour afficher l’ensemble des booléens personnalisés, vous pourrez utiliser la commande suivante :

# semanage boolean -lC 
SELinux boolean                State  Default Description
httpd_enable_homedirs          (on   ,   on)  Allow httpd to enable homedirs
```

Afficher les contexts locaux
```semanage fcontext -C -l```

Le fichier contenant les fcontexts

```/etc/selinux/targeted/contexts/files```
file_contexts - default contexts and updated by semanage
file_contexts.local - newly created and not found in file_contexts

