# Idm Server + OTP Token parte 2
## Deploy Idm.

### Configurando Red.
Lo primero que realizaremos será colocal ip estaticas en nuestro servidor.

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


### Asignaremos nombre al equipo
Hostname:

	[root@localhost network-scripts]# hostnamectl set-hostname idm.poc.redhat.com
	[root@localhost network-scripts]# hostnamectl set-hostname "Mike's lab for demos" --pretty 

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
	
	Actualizando:
	
	[root@idm ~]# yum update -y
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
	
	Validando nuestro tipo de soporte:
	
	[root@idm ~]# subscription-manager service-level --list
	+-------------------------------------------+
               Available Service Levels
	+-------------------------------------------+
	Premium-Support




