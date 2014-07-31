# Creacion de maquina virtual desde consola

- Revisar si la maquina aguanta virtualiacion

	`egrep -c '(vmx|svm)' /proc/cpuinfo`

	__NOTA__: Si no estrega un 0 significa que no acepta virtualizacion de no ser asi continuar con la guia

- Instalar paquetes necesario para la creacion de la maquina virtual

	`apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils`
	
- Agregar nuestro usuario al grupo de virtualizacion

	`adduser root libvirtd`
	
- verificar instalacion

		root@storage-hq:/var/lib/libvirt/images# virsh list	
	 	Id Name                 State
		----------------------------------
		
- crear maquina virtual
	- crear archivo de configuracion de las particiones
		
		`cd /var/lib/libvirt/images/`
	- crear contenedor
	
		`mkdir nombremserver/: cd nombresever/`
		
	- Crear archivo de particiones
		
		`vim vmbuilder.partition`

			root 4000
			swap 2000
			---
			/var/log 4000
			---
			/opt 4000

__Nota__ : los espacios deben ser establecidos con anterioridad, en el ejemplo estamos estableciendo __4GB para /__, __2GB para swap__, __4GB para /var/log__, y __4GB para /opt__

- configurar tarjeta de red virtual

	- editar el archvio `/etc/network/interfaces` y dejar la tarjeta por defecto de la siguiente forma
	
			auto eth0
			iface eth0 inet manual
			auto br0
			iface br0 inet static
			address 10.0.1.2
			netmask 255.255.255.0
			gateway 10.0.1.201
			dns-nameservers 10.0.1.201
			bridge_ports eth0
			bridge_fd 9
			bridge_hello 2
			bridge_maxage 12
			bridge_stp off
			
- Instalar sistema operativo en maquina virtual

``vmbuilder kvm ubuntu --suite precise --flavour virtual --arch amd64 -o --libvirt qemu:///system --ip 10.0.1.249 --mask 255.255.255.0 --gw 10.0.1.201 --dns 10.0.1.201  --bridge br0 --hostname desarrollo.mediastre.am --part vmbuilder.partition --mem 1024 --cpus 2 --addpkg htop --addpkg vim --addpkg aptitude --addpkg openssh-server  --mirror http://cl.archive.ubuntu.com/ubuntu/ --user usuario --pass changeme`

en las lineas de vmbuilder le estamos indicando lo siguiente

	Kvm = modulo que usara
	suite = sistema operativo a instalar en esta caso se instalara ubuntu precise (ubuntu 12.04)
	arch = arquitectura
	ip = asignamos la ip para la maquina virtual
	mask = mascara
	gw = gateway
	dns = dns
	brigge = tarjeta base en la cual debe crear la red virtual
	hostname = nombre de la maquina
	part = archivo de particiones
	mem = cantidad de memoria
	cpus = cpus disponibles
	addpkg = paquetes que debe instalar
	mirror = desde donde debe tomar la imagen y sus paquetes
	user = usuario
	pass = password


__NOTA__ : Es posible experimentar problemas al crear una máquina virtual. Cambie el grupo del dispositivo para kvm/libvirtd dejandolo de la siguiente forma:

`chown root:kvm /dev/kvm`
