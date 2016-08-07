# Idm Server + OTP Token parte 2
## Deploy Idm.

### Configurando Red.
Lo primero que realizaremos será colocar ip's estáticas en nuestro servidor.

	[root@localhost network-scripts]# cat ifcfg-eth0 
		DEVICE=eth0 
		BOOTPROTO=static
		ONBOOT=yes
		NM_CONTROLLED=no
		NETWORK=192.168.122.0 
		NETMASK=255.255.255.0 
		IPADDR=192.168.122.10
		GATEWAY=192.168.122.1
		DNS1=192.168.122.10
		DNS2=192.168.122.1 
		USERCTL=no
		TYPE=Ethernet

	[root@localhost network-scripts]# cat ifcfg-eth1 
		DEVICE=eth1
		BOOTPROTO=static
		ONBOOT=yes
		NM_CONTROLLED=no
		NETWORK=10.10.0.10 
		NETMASK=255.255.252.0 
		IPADDR=10.10.0.100
		USERCTL=no
		TYPE=Ethernet
		
	[root@localhost network-scripts]# systemctl disable NetworkManager
	[root@localhost network-scripts]# systemctl stop NetworkManager
	[root@localhost network-scripts]# chkconfig network on
	[root@localhost network-scripts]# service network start


### Asignaremos nombre al equipo
Hostname:

	[root@localhost network-scripts]# hostnamectl set-hostname idm.poc.redhat.com
	[root@localhost network-scripts]# hostnamectl set-hostname "Mike's lab for demos" --pretty
	[root@localhost network-scripts]# hostnamectl set-hostname --transient ipa.pisa.com

Validamos hostname:

	[root@localhost network-scripts]# hostnamectl status
	   Static hostname: idm.poc.redhat.com
	   Pretty hostname: Mike's lab for demos
	         Icon name: computer-vm
	           Chassis: vm
	        Machine ID: 02f1ddb1415c4feba9880b2b8c4c5925
	           Boot ID: b130ac7c1b514dfdbd308f378dd7ae61
	    Virtualization: kvm
	  Operating System: Red Hat Enterprise Linux Server 7.2 (Maipo)
	       CPE OS Name: cpe:/o:redhat:enterprise_linux:7.2:GA:server
	            Kernel: Linux 3.10.0-327.10.1.el7.x86_64
	      Architecture: x86-64
	
	[root@localhost network-scripts]# hostname
	idm.poc.redhat.com


### Registramos S.O. y Actualizamos S.O. desde repositorios oficiales.

Registrando:
	
	[root@idm ~]#   subscription-manager register --username rhn-user --password password --auto-attach
	Registering to: subscription.rhn.redhat.com:443/subscription
	The system has been registered with ID: 05bb900a-5b64-4497-9895-b80ead952014
	
Validando nuestro tipo de soporte:
	
	[root@idm ~]# subscription-manager service-level --list
	+-------------------------------------------+
	Available Service Levels
	+-------------------------------------------+
	Premium-Support
	
Asignando Canales necesarios:
	
	subscription-manager repos --disable=*
   	subscription-manager repos --enable=rhel-7-server-rpms
   	subscription-manager repos --enable=rhel-7-server-optional-rpms
   	subscription-manager repos --enable=rhel-7-server-supplementary-rpms

Actualizando:
	
	[root@idm ~]# yum install deltarpm ; yum update -y
	(...)
	Resumen de la transacción
	================================================================
	Instalar     1 Paquete  (+1 Paquete dependiente)
	Actualizar  65 Paquetes
	================================================================
	Tamaño total de la descarga: 111 M
	Downloading packages:
		(...)
	¡Listo!
	
### Instalando y configurando Idm 

	[root@localhost network-scripts]# yum install ipa-server bind bind-dyndb-ldap ipa-server-dns
	(...)
	[root@localhost network-scripts]# echo "192.168.122.10 idm.poc.redhat.com idm" >>/etc/hosts
	
	Antes de continuar crearemos un snapshot de nuestra vm.
	
	* Creando:
	
	[root@localhost ~]# virsh snapshot-create-as IDM-Server IDM-Server-snap-Lab2
	Ha sido creada la captura instantánea IDM-Server-snap-Lab2 del dominio

	* Validando snapshot:
	
	[root@localhost ~]# virsh snapshot-list IDM-Server
	 Nombre               Hora de creación         Estado
	------------------------------------------------------------
	 IDM-Server-snap-Lab2 2016-05-21 23:11:05 -0500 shutoff
	
	* Reiniciando vm:
	
	[root@localhost ~]# virsh start IDM-Server
	Se ha iniciado el dominio IDM-Server

	* Validando status vm:
	
	[root@localhost ~]# virsh list --all
	 Id    Nombre                         Estado
	----------------------------------------------------
	 18    IDM-Server                     ejecutando

	* Configurando Idm.
	
	[root@idm ~]# ipa-server-install

		The log file for this installation can be found in /var/log/ipaserver-install.log
		==============================================================================
		This program will set up the IPA Server.
		
		This includes:
		  * Configure a stand-alone CA (dogtag) for certificate management
		  * Configure the Network Time Daemon (ntpd)
		  * Create and configure an instance of Directory Server
		  * Create and configure a Kerberos Key Distribution Center (KDC)
		  * Configure Apache (httpd)
		
		To accept the default shown in brackets, press the Enter key.
				(...)
		Do you want to configure integrated DNS (BIND)? [no]: yes
		(...)
		Server host name [idm.poc.redhat.com]: 
		(...)
		Please confirm the domain name [poc.redhat.com]: 
		(...)
		Directory Manager password: password!
		Password (confirm): password!
		(...)
		IPA admin password: password!
		Password (confirm): password!
		Existing BIND configuration detected, overwrite? [no]: yes
		Do you want to configure DNS forwarders? [yes]: 
		Enter an IP address for a DNS forwarder, or press Enter to skip: 192.168.122.1
		(...)
		Do you want to configure the reverse zone? [yes]: 
		Please specify the reverse zone name [122.168.192.in-addr.arpa.]: 
		Using reverse zone(s) 122.168.192.in-addr.arpa.
		
		The IPA Master Server will be configured with:
		Hostname:       idm.poc.redhat.com
		IP address(es): 192.168.122.10
		Domain name:    poc.redhat.com
		Realm name:     POC.REDHAT.COM
		
		BIND DNS server will be configured to serve IPA domain with:
		Forwarders:    192.168.122.1
		Reverse zone(s):  122.168.192.in-addr.arpa.
		
		Continue to configure the system with these values? [no]: yes
		(...)
		(...)
		(...)
		==============================================================
		Setup complete
		
		Next steps:
			1. You must make sure these network ports are open:
				TCP Ports:
				  * 80, 443: HTTP/HTTPS
				  * 389, 636: LDAP/LDAPS
				  * 88, 464: kerberos
				  * 53: bind
				UDP Ports:
				  * 88, 464: kerberos
				  * 53: bind
				  * 123: ntp
		
			2. You can now obtain a kerberos ticket using the command: 'kinit admin'
			   This ticket will allow you to use the IPA tools (e.g., ipa user-add)
			   and the web user interface.
		
		Be sure to back up the CA certificates stored in /root/cacert.p12
		These files are required to create replicas. The password for these
		files is the Directory Manager password
	
	* Agregamos los puertos al firewall 
	
	[root@idm ~]# yum install firewalld
	(...)

	Dependencias resueltas

		======================================================================================================
		 Package          Arquitectura          Versión          Repositorio          Tamaño
		======================================================================================================
		Instalando:
		 firewalld           noarch             0.3.9-14.el7     rhel-7-server-aus-rpms    476 k
		Instalando para las dependencias:
		 ebtables            x86_64             2.0.10-13.el7    rhel-7-server-aus-rpms    122 k
		 python-slip         noarch             0.4.0-2.el7      rhel-7-server-aus-rpms    30 k
		 python-slip-dbus    noarch             0.4.0-2.el7      rhel-7-server-aus-rpms    31 k
		Resumen de la transacción
		=======================================================================================================
		Instalar  1 Paquete (+3 Paquetes dependientes)
		----------------------------------------------------------------------------------------------------
		Total                                                                                                                                                                              192 kB/s | 660 kB  00:00:03     
		Running transaction check
		Running transaction test
		Transaction test succeeded
		Running transaction
		(...)
		¡Listo!
		
		* Iniciamos firewall

		[root@idm ~]# systemctl start firewalld.service

		* Habilitamos firewall

		[root@idm ~]# systemctl enable firewalld.service

		*Validamos status firewall.

		[root@idm ~]# systemctl status firewalld.service
			● firewalld.service - firewalld - dynamic firewall daemon
			   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
			   Active: active (running) since dom 2016-05-22 01:04:39 EDT; 8s ago
			 Main PID: 5980 (firewalld)
			   CGroup: /system.slice/firewalld.service
			           └─5980 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
			
			may 22 01:04:36 idm.poc.redhat.com systemd[1]: Starting firewalld - dynamic firewall daemon...
			may 22 01:04:39 idm.poc.redhat.com systemd[1]: Started firewalld - dynamic firewall daemon.
			
		* Agregamos puertos firewall		

		[root@idm ~]# firewall-cmd --permanent --zone=public --add-port=80/tcp --add-port=443/tcp --add-port=389/tcp --add-port=636/tcp --add-port=88/tcp --add-port=464/tcp --add-port=88/udp --add-port=464/udp --add-port=123/udp
		success
		
		*Reiniciamos firewall.
		[root@idm ~]# systemctl restart firewalld.service
		
## Acedemos desde el navegador a nuestro Idm.

* Ingresaremos con nuestro usuario y password que configuramos durante el proceso de instalación.
	
![GitHub Logo](/img/Lab-2/Lab-2-a.png) 

* En la configuración post-instalación podemos observar que solo se encuentra el usuario admin.

![GitHub Logo](/img/Lab-2/Lab-2-b.png)

* Podemos ver las zonas de DNS que se generarón.

![GitHub Logo](/img/Lab-2/Lab-2-c.png)

* Y los registros que se ingresarón default.
	
![GitHub Logo](/img/Lab-2/Lab-2-d.png) 

##**En este laboratorio realizaremos la asignación de token mediante 2 procedimientos**
###_Escenario 1_
El procedimiento se basa en los siguiente pasos:

1. Como admin
	+ Crear usuario
	+ logout admin
2. Como usuario.
	+ First login
	+ Cambiar password default
	+ Agregar OTP Token (TOTP) (HOTP)
	+ Configurar OTP QR
	+ Logout usuario
3. Como admin.
	+ Editar usuario
	+ Habilitar autenticacion de 2 factores

_En este punto el usuario es capas de autenticar con su **Password** + **Token OTP**_

###_Escenario 2_
El procedimiento se basa en los siguiente pasos:

1. Como admin
	+ Crear usuario
	+ Editar usuario
	+ Habilitar autenticacion de 2 factores
	+ logout admin
2. Como usuario.
	+ First login
	+ Cambiar password default
	+ Agregar OTP Token (TOTP) (HOTP)
	+ Configurar OTP QR
	+ Logout user


# Escenario I
	
* Ahora dentro de la pestaña Users crearemos al usuario Demo
	
![GitHub Logo](/img/Lab-2/Lab-2-e.png) 

* Como siguiente paso realizaremos logout.

![GitHub Logo](/img/Lab-2/Lab-2-f.png) 

* Para continuar con la configuración, haremos login como usuario Demo

![GitHub Logo](/img/Lab-2/Lab-2-g.png) 

* El Idm por default nos pedirá que cambiemos el password default por uno nuevo.

![GitHub Logo](/img/Lab-2/Lab-2-h.png)

* Posteriormente en la pestaña Actions, seleccionaremos Add OTP Token.

![GitHub Logo](/img/Lab-2/Lab-2-i.png)

* Seleccionaremos Time-based (TOTP).

![GitHub Logo](/img/Lab-2/Lab-2-k.png) 

* Agregamos el codigo QR.

![GitHub Logo](/img/Lab-2/Lab-2-l.png) 

* Abrimos la URL que nos muestra.

![GitHub Logo](/img/Lab-2/Lab-2-m.png) 

* Seleccionamos nuestra app a utlizar (Goole Authenticator, Red Hat FreeOTP).

![GitHub Logo](/img/Lab-2/Lab-2-n.png) 

* Nuestro token ahora esta disponible para ser usado.

![GitHub Logo](/img/Lab-2/Lab-2-o.png) 

* Como admin editamos las propiedades del usuario.

![GitHub Logo](/img/Lab-2/Lab-2-p.png) 

* Como admin habilitamos la autenticación de 2 factores.

![GitHub Logo](/img/Lab-2/Lab-2-q.png) 

* Como usuario ahora ya podemos autenticarnos con nuestro Password + Token.

![GitHub Logo](/img/Lab-2/Lab-2-r.png) 

# Escenario II

* Ahora dentro de la pestaña Users crearemos al usuario Red Hat y seleccionamos la opción Editar

![GitHub Logo](/img/Lab-2/Lab-2-s.png) 

* Como admin habilitamos la autenticación de 2 factores y guardamos cambios.

![GitHub Logo](/img/Lab-2/Lab-2-t.png) 

* Ahora haremos login como Red Hat

![GitHub Logo](/img/Lab-2/Lab-2-u.png) 

* Cambiamos el password por default por uno personalizado.

![GitHub Logo](/img/Lab-2/Lab-2-v.png) 

* Posteriormente en la pestaña Actions, seleccionaremos Add OTP Token.

![GitHub Logo](/img/Lab-2/Lab-2-w.png) 

* Seleccionaremos Time-based (HOTP).

![GitHub Logo](/img/Lab-2/Lab-2-x.png)

* Aparecera el codigo QR.

![GitHub Logo](/img/Lab-2/Lab-2-y.png)

* Desde Red Hat FreeOTP leeremos el QR.

![GitHub Logo](/img/Lab-2/Lab-2-z.png) 

* Ahora ya tenemos nuestro token asignado a la cuenta Red Hat.

![GitHub Logo](/img/Lab-2/Lab-2-aa.png)

* Realizamos Login con el usuarios Red Hat / password+token.

![GitHub Logo](/img/Lab-2/Lab-2-ab.png) 


	






