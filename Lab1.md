# Idm Server + OTP Token parte 1
Este Laboratorio consiste en el despliegue de un servidor Idm (IPA) + OTP Token, basado en una imagen qcow2 desplegada en openstack.

### 1.- Lo primero que realizaremos es descargar la imagen a nuestro equipo.

	host# curl -o /var/lib/libvirt/images/rhel7-guest-official.qcow2 https://access.redhat.com/downloads/content/69/ver=/rhel---7/7.1/x86_64/product-downloads

### 2 .- Checamos la información de la imagen que acabamos de descargar.

	[root@localhost images]# qemu-img info rhel-guest-image-7.2-20160302.0.x86_64.qcow2 
	image: rhel-guest-image-7.2-20160302.0.x86_64.qcow2
	file format: qcow2
	virtual size: 10G (10737418240 bytes)
	disk size: 470M
	cluster_size: 65536
	Format specific information:
    		compat: 0.10
    		refcount bits: 16

Sí investigamos un poco más nos podemos dar cuenta que la imagen solo tiene 6GB de almacenamiento disponible.
	
	[root@localhost images]# virt-filesystems --long -h --all -a rhel-guest-image-7.2-20160302.0.x86_64.qcow2
	Name       Type        VFS  Label  MBR  Size  Parent
	/dev/sda1  filesystem  xfs  -      -    6.0G  -
	/dev/sda1  partition   -    -      83   6.0G  /dev/sda
	/dev/sda   device      -    -      -    10G   -

### 3.- Vamos a tener que cambiar el tamaño de la imagen para tener un poco más de espacio para los laboratorios posteriores. 
La herramienta virt-resize permite cambiar el tamaño de la partición y el sistema de archivos al mismo tiempo, 
pero no funciona sobre la imagen original, por lo que es necesario crear una nueva imagen base:

	[root@localhost images]# qemu-img create -f qcow2 rhel7-guest-40G.qcow2 40G
	Formatting 'rhel7-guest-40G.qcow2', fmt=qcow2 size=42949672960 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16

### 4 .- Con el fin de cambiar automáticamente el tamaño del sistema de ficheros XFS, virt-resize requiere libguestfs-xfs:

	[root@localhost images]# dnf -y install libguestfs-xfs

### 5.- Ahora ejecutamos virt-resize:

	[root@localhost images]# virt-resize --expand /dev/sda1 rhel-guest-image-7.2-20160302.0.x86_64.qcow2 rhel7-guest-40G.qcow2
	[   0.0] Examining rhel-guest-image-7.2-20160302.0.x86_64.qcow2
	**********

	Summary of changes:

	/dev/sda1: This partition will be resized from 6.0G to 40.0G.  The 
	filesystem xfs on /dev/sda1 will be expanded using the 'xfs_growfs' method.

	**********
	[   3.3] Setting up initial partition table on rhel7-guest-40G.qcow2
	[   3.4] Copying /dev/sda1
 	100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
	[  15.8] Expanding /dev/sda1 using the 'xfs_growfs' method

	Resize operation completed with no errors.  Before deleting the old disk, 
	carefully check that the resized disk boots and works correctly.

Verificamos los cambios realizados a nuestro Disco y deberiamos observar que el tamaño es de 40G:

	[root@localhost images]# qemu-img info rhel7-guest-40G.qcow2
	image: rhel7-guest-40G.qcow2
	file format: qcow2
	virtual size: 40G (42949672960 bytes)
	disk size: 1.1G
	cluster_size: 65536
	Format specific information:
    		compat: 1.1
    		lazy refcounts: false
    		refcount bits: 16
    		corrupt: false

Validamos los cambios el el filesystem:

	[root@localhost images]# virt-filesystems --long -h --all -a rhel7-guest-40G.qcow2
	Name       Type        VFS  Label  MBR  Size  Parent
	/dev/sda1  filesystem  xfs  -      -    40G   -
	/dev/sda1  partition   -    -      83   40G   /dev/sda
	/dev/sda   device      -    -      -    40G   -


### 6 .- A continuación, vamos a eliminar la cloud-init desde guest-image; no vamos a necesitar este servicio, 
ya que no estamos usando esta máquina dentro de un entorno que admite un servicio de metadatos, 
además de que también se ralentiza considerablemente el proceso de arranque:

	[root@localhost images]# virt-customize -a rhel7-guest-40G.qcow2 --run-command 'yum remove cloud-init* -y'
	[   0.0] Examining the guest ...
	[  11.8] Setting a random seed
	[  11.8] Running: yum remove cloud-init* -y
	[  15.8] Finishing off

### 7.- Ahora procederemos a cambiar el password default del usuario root:

	[root@localhost images]# virt-customize -a rhel7-guest-40G.qcow2 --root-password password:idmserver
	[   0.0] Examining the guest ...
	[  15.5] Setting a random seed
	[  15.5] Setting passwords
	[  16.9] Finishing off
	
### 8.- Tenemos que asegurarnos de que nuestra máquina Idm tiene una interfaz 2 interfaces de red por defecto,
por lo que crearemos un archivo de configuración para la interfaz para eth1 basado en la eth0:

	[root@localhost images]# virt-customize -a rhel7-guest-40G.qcow2 --run-command 'cp /etc/sysconfig/network-scripts/ifcfg-eth{0,1} && sed -i s/DEVICE=.*/DEVICE=eth1/g /etc/sysconfig/network-scripts/ifcfg-eth1'
	[   0.0] Examining the guest ...
	[  16.4] Setting a random seed
	[  16.4] Running: cp /etc/sysconfig/network-scripts/ifcfg-eth{0,1} && sed -i s/DEVICE=.*/DEVICE=eth1/g /etc/sysconfig/network-scripts/ifcfg-eth1
	[  16.7] Finishing off

### 9.- Procederemos a crear nuestra vm en KVM con la imagen rhel7-guest-40G.qcow2 que acabamos de crear.

* Creación de vm.

![GitHub Logo](/img/Lab-1/Lab-1-a.png)

* Seleccionamos el Disco que creamos.

![GitHub Logo](/img/Lab-1/Lab-1-i.png)

* Asignamos tipo y Versión de S.O.

![GitHub Logo](/img/Lab-1/Lab-1-c.png)

* Asignamos memoria y CPU

![GitHub Logo](/img/Lab-1/Lab-1-d.png)

* Personalizamos la configuración (Agregaremos la segunda interfaz de red)

![GitHub Logo](/img/Lab-1/Lab-1-e.png)

* Asignamos la segunda interfaz de red que configuramos.

![GitHub Logo](/img/Lab-1/Lab-1-f.png)

* Terminamos el proceso de creación, iniciamos la vm, realizamos loggin y validamos interfaces de red.

![GitHub Logo](/img/Lab-1/Lab-1-g.png)


