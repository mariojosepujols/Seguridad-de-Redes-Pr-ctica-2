# Seguridad-de-Redes-Pr-ctica-2

## Video demostrativo

**Enlace:** 


## Propósito

Implementar una VPN IPsec site-to-site entre dos FortiGate para comunicar un usuario de la VLAN 10 con un servidor web remoto. Se comprueba la conectividad mediante ping, acceso a la página de Apache y traceroute. El objetivo final es demostrar que la comunicación funciona con la VPN activa y falla cuando el túnel está desactivado.

## Topología

La nube Net de PNETLab representa la conexión al ISP. Los equipos FortiGate utilizan direcciones WAN privadas asignadas por DHCP dentro de la red NATeada del laboratorio.


## Direccionamiento

| Sitio o interfaz | Dirección o red | Máscara | Uso |
|---|---|---|---|
| Site 1, port1 | 192.168.230.151 | /24 | WAN asignada por DHCP desde Net |
| Site 1, VLAN 10 | 10.14.53.1 | /25 | Gateway y DHCP de usuarios |
| Usuarios, VLAN 10 | 10.14.53.10–10.14.53.126 | /25 | Rango DHCP configurado |
| Site 2, port1 | 192.168.230.152 | /24 | WAN asignada por DHCP desde Net |
| Site 2, port2 | 10.14.53.129 | /28 | Gateway de la red del servidor |
| Servidor web | 10.14.53.130 | /28 | Dirección observada en las pruebas |

Las direcciones `192.168.230.151` y `192.168.230.152` son privadas y están dentro de la red NATeada de PNETLab. Por lo tanto, en este laboratorio simulan la conexión WAN, pero no son direcciones públicas de Internet.

## Configuración del Site 1

En el FortiGate Site 1, `port1` recibió `192.168.230.151/24`. Sobre `port2` se configuró la interfaz VLAN `Vlan10-Usuarios`, con la dirección `10.14.53.1/25` y el rango DHCP `10.14.53.10–10.14.53.126`.


El DHCP está configurado en el FortiGate. Las capturas de esta entrega no muestran la dirección específica que recibió el cliente.

El túnel `VPN-SITE-2` aparece activo en el FortiGate Site 1.


### Políticas del Site 1

Las políticas permiten tráfico entre la red de usuarios y la interfaz VPN en ambos sentidos. En las capturas se observan acción `ACCEPT`, servicio `ALL` y NAT desactivado.



### Rutas del Site 1

La tabla muestra una ruta por defecto mediante `port1` y rutas hacia la red remota por el túnel VPN. También aparece una ruta blackhole hacia la red remota.


## Configuración del Site 2

En el FortiGate Site 2, `port1` recibió `192.168.230.152/24`. La interfaz `WEB-SERVER-LAN`, correspondiente a `port2`, tiene la dirección `10.14.53.129/28`.



El túnel aparece activo. En el monitor IPsec se muestra como peer remoto `192.168.230.151`, con Phase 1 y Phase 2 activas y tráfico entrante y saliente.


### Políticas del Site 2

Las políticas permiten tráfico entre la interfaz VPN y la red del servidor en ambos sentidos. Las capturas muestran acción `ACCEPT`, servicio `ALL` y NAT desactivado.



### Rutas del Site 2

La tabla muestra una ruta por defecto mediante `port1` y rutas hacia la red remota por el túnel VPN. También aparece una ruta blackhole hacia la red remota.



## NAT

Las políticas que permiten la comunicación entre las dos redes muestran NAT desactivado. Así, el tráfico entre los usuarios y el servidor conserva sus direcciones originales al cruzar la VPN.

La red Net de PNETLab proporciona la salida NATeada del laboratorio. 


## Resultados de las pruebas

### Ping desde el cliente al servidor

Desde Ubuntu Desktop se ejecutó un ping hacia `10.14.53.130`. La captura muestra cuatro respuestas del servidor.



**Resultado:** conectividad ICMP entre el cliente y el servidor confirmada mientras el túnel aparece activo.

### Acceso a la página web

Firefox muestra la página predeterminada de Apache en `10.14.53.130`.



### Traceroute desde el cliente al servidor

El traceroute llegó a `10.14.53.130` en tres saltos.

| Salto | Dirección | Tiempo observado |
|---|---|---|
| 1 | 10.14.53.1 | 4.075 ms, 2.070 ms y 1.067 ms |
| 2 | 192.168.230.152 | 9.735 ms, 9.650 ms y 9.630 ms |
| 3 | 10.14.53.130 | 9.542 ms, 9.520 ms y 9.501 ms |


**Resultado:** el destino fue alcanzado. La dirección WAN del Site 2 aparece como segundo salto.


## Conclusión

Las capturas muestran que el túnel IPsec está activo y que el cliente alcanza el servidor `10.14.53.130`: el ping recibe respuestas, Firefox carga la página de Apache y el traceroute llega al destino en tres saltos. Las políticas entre los sitios muestran NAT desactivado.

