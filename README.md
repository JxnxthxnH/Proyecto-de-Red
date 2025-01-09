## PROYECTO DE RED EMPRESARIAL

Topología de red hibrida de una empresa de tres niveles, con divisiones por VLANs en cada departamento. Esta comprendida por 2 IPS, 2 Routers, 2 Multilayers, 6 Switches, 6 PCs, 5 Printers, 5 Access Point, 5 Laptops, 5 Smartphones y 3 Servers, creando un sistema de redundancia sólido.

![image alt](https://github.com/JxnxthxnH/Proyecto-de-Red/blob/main/images/topologia.png?raw=true)
![image alt](https://github.com/JxnxthxnH/Proyecto-de-Red/blob/main/images/topologia2.png?raw=true)

### Configuración básica de acceso

- Se configuró cada Switch, Multilayer y Router con hostname y contraseña, en este caso es una predeterminada (cisco).

> En el siguiente ejemplo se muestra cómo se configuró en el router CORE-R1:

```
hostname CORE-R1
line console 0
password cisco
login
exit
enable password cisco
no ip domain lookup
banner motd #NO Unauthorised Access!!!#
service password-encryption
do wr
```

- A su vez se configuró el protocolo SSH en los Routers, Multilayers y Switches para establecer conexión de manera remota:

```
ip domain name cisco.net
username admin password cisco
crypto key generate rsa
1024
line vty 0 15
login local
transport input ssh
exit
ip ssh version 2
do wr
```

### Rango de IP que se utilizó

- Red base: 172.16.1.0

> Primer piso

```
Ventas | 172.16.1.0 | 255.255.255.128/25 | 172.16.1.1 to 172.16.1.126 | 172.16.1.127
RRHH | 172.16.1.128 | 255.255.255.128/25 | 172.16.1.129 to 172.16.1.254 | 172.16.1.255
```

> Segundo piso

```
Contabilidad | 172.16.2.0 | 255.255.255.128/25 | 172.16.2.1 to 172.16.2.126 | 172.16.2.127
Administracion | 172.16.2.128 | 255.255.255.128/25 | 172.16.2.129 to 172.16.2.254 | 172.16.2.255
```

> Tercer piso

```
TI | 172.16.3.0 | 255.255.255.128/25 | 172.16.3.1 to 172.16.3.126 | 172.16.3.127
Servidores | 172.16.3.128 | 255.255.255.240/28 | 172.16.3.129 to 172.16.3.142 | 172.16.3.143
```

> Entre los Routers y Multilayers

```
R1-MLSW1 | 172.16.3.144 | 255.255.255.252 | 172.16.3.145 a 172.16.3.146 | 172.16.3.147
R2-MLSW2 | 172.16.3.148 | 255.255.255.252 | 172.16.3.149 a 172.16.3.150 | 172.16.3.151
R1-MLSW1 | 172.16.3.152 | 255.255.255.252 | 172.16.3.153 a 172.16.3.154 | 172.16.3.155
R2-MLSW2 | 172.16.3.156 | 255.255.255.252 | 172.16.3.157 a 172.16.3.158 | 172.16.3.159
```

> Entre Routers e IPS

```
Direcciones de IP públicas: 195.136.17.0/30, 195.136.17.4/30, 195.136.17.8/30 y 195.136.17.12/30
```


### Enrutamiento

- Se configuró VLANs para cada departamento.

> En este caso se muestra de ejemplo la VLAN 10 del departamento de Ventas:

```
int range fa0/1-2
switchport mode trunk
exit
vlan 10
name Ventas
vlan 99 BlackHole
exit
int range fa0/3-24
switchport mode access
switchport access vlan 10
exit
int range gig0/1-2
switchport mode access
switchport access vlan 99
exit
do wr
```
- Configuración de puertos trunks en los Multilayers centrales.

```
int range gig1/0/3-8
switchport mode trunk
vlan 10
name Ventas
vlan 20
name HHRR
vlan 30
name Contabilidad
vlan 40
name Administracion
vlan 50
name TI
vlan 60
name Servidores
```

- Configuracion de OSPF en los Multilayers, Routers e IPS

> Configuración en los multilayers:

```
ip routing
router ospf 10
router-id 2.2.2.2
network 172.16.1.0 0.0.0.127 area 0
network 172.16.1.128 0.0.0.127 area 0
network 172.16.2.0 0.0.0.127 area 0
network 172.16.2.128 0.0.0.127 area 0
network 172.16.3.0 0.0.0.127 area 0
network 172.16.3.128 0.0.0.15 area 0
network 172.16.3.144 0.0.0.3 area 0
network 172.16.3.148 0.0.0.3 area 0
exit
do wr
```

> Configuración en los routers:

```
router ospf 10
router-id 3.3.3.3
network 172.16.3.144 0.0.0.3 area 0
network 172.16.3.152 0.0.0.3 area 0
network 195.136.17.0 0.0.0.3 area 0
network 195.136.17.4 0.0.0.3 area 0
exit
do wr
```

> Configuración en los IPS:

```
router ospf 10
router-id 5.5.5.5
network 195.136.17.0 0.0.0.3 area 0
network 195.136.17.8 0.0.0.3 area 0
exit
do wr
```

### Puerto de seguridad (Dpto. de Contabilidad)

> Configuración al switcher del Departamento de Contabilidad:

```
int range fa0/3-24
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
exit
do wr
```

### Configuración del área de servidores con IP estática

> DHCP Server:

```
IP: 172.16.3.130
Mascara de Subred: 255.255.255.240
Default Gateway: 172.16.3.129
```

> DNS Server:

```
IP: 172.16.3.131
Mascara de Subred: 255.255.255.240
Default Gateway: 172.16.3.129
```

> SysAdmin-PC:

```
IP: 172.16.3.132
Mascara de Subred: 255.255.255.240
Default Gateway: 172.16.3.129
```

> WEB Server:

```
IP: 172.16.3.133
Mascara de Subred: 255.255.255.240
Default Gateway: 172.16.3.129
```

### DHCP Relay Agent

> Se configuraron los multilayers para que el servidor DHCP pueda asignar IP a cada VLAN, en este caso mostramos de ejemplo la configuración para la VLAN 10:

```
int vlan 10
no sh
ip add 172.16.1.1 255.255.255.128
ip helper-address 172.16.3.130
exit
do wr
```

> En la sala de servidores, al utilizar otra máscara de subred, la configuración es la siguiente:

```
int vlan 60
no sh
ip add 172.16.3.129 255.255.255.240
ip helper-address 172.16.3.130
exit
do wr
```

### Configuración de Port Address Translation en los routers centrales

```
ip nat inside source list 1 int se0/2/0 overload
ip nat inside source list 1 int se0/2/1 overload
access-list 1 permit 172.16.1.0 0.0.0.127
access-list 1 permit 172.16.1.128 0.0.0.127
access-list 1 permit 172.16.2.0 0.0.0.127
access-list 1 permit 172.16.2.128 0.0.0.127
access-list 1 permit 172.16.3.0 0.0.0.127
access-list 1 permit 172.16.3.128 0.0.0.15
int range gig0/0-1
ip nat inside
yes
exit
int se0/2/0
ip nat outside
int se0/2/1
ip nat outside
do wr
do sh ip nat translation
```

### Configuración de Default Route

Con el propósito de mejorar la redundancia de la red se configuró en los multilayers y routers una ruta por defecto en caso de enviar paquetes que no tenga ruta especifica configurada.

> Multilayers

```
ip route 0.0.0.0 0.0.0.0 gig1/0/1
ip route 0.0.0.0 0.0.0.0 gig1/0/2 70
do wr
```

> Routers

```
ip route 0.0.0.0 0.0.0.0 se0/2/0
ip route 0.0.0.0 0.0.0.0 se0/2/1 70
do wr
```
