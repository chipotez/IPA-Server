# Laboratorio III implementación de FreeRadius + IPA + Token

* Actualizamos Sistema.
[root@radius ~]# yum install deltarpm -y  ; yum update -y
(...)

Dependencias resueltas

================================================================================
 Package        Arquitectura Versión         Repositorio                  Tamaño
================================================================================
Instalando:
 deltarpm       x86_64       3.6-3.el7       rhel-7-server-aus-rpms        82 k

Resumen de la transacción
================================================================================
Instalar  1 Paquete
(...)
(...)
Dependencias resueltas

================================================================================
 Package             Arquitectura
                            Versión                Repositorio            Tamaño
================================================================================
Instalando:
 kernel              x86_64 3.10.0-327.18.2.el7    rhel-7-server-aus-rpms  33 M
(...)
(...)

* Instalamos FreeRadius.
[root@radius ~]# yum install freeradius freeradius-ldap freeradius-utils wpa_supplicant --enablerepo=rhel-7-server-optional-rpms
(...)
(...)
Dependencias resueltas

================================================================================
 Package                 Arquitectura
                                Versión          Repositorio              Tamaño
================================================================================
Instalando:
 freeradius              x86_64 3.0.4-6.el7      rhel-7-server-aus-rpms   985 k
 freeradius-ldap         x86_64 3.0.4-6.el7      rhel-7-server-optional-rpms
                                                                           95 k
 freeradius-utils        x86_64 3.0.4-6.el7      rhel-7-server-optional-rpms
                                                                          188 k
Instalando para las dependencias:
(...)
(...)



[root@radius raddb]# cd /etc/raddb/

[root@radius raddb]# cp clients.conf  cp clients.conf.original






