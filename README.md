# Windows Server IT Support Lab

**Laboratorio práctico de administración de sistemas Windows, infraestructura de red y soporte IT**

## 1. Objetivo

He diseñado y desplegado un entorno virtual que simula la infraestructura IT de una pequeña empresa, organizada en tres departamentos.

El proyecto reproduce tareas habituales de administración de sistemas y soporte IT: gestión de identidades, configuración de servicios de red, aplicación de directivas de grupo y administración de recursos compartidos.

El objetivo es comprender cómo se relacionan estos componentes y cómo verificar su funcionamiento desde un equipo cliente.

## 2. Arquitectura y entorno

| Componente | Configuración |
|---|---|
| Hipervisor | Oracle VirtualBox |
| Servidor | Windows Server 2025 Standard Evaluation |
| Equipo cliente | Windows 10 Pro |
| Controlador de dominio | SRV-AD01 |
| Dominio | ad.lab.test |
| Red virtual | NAT Network: Lab-AD |
| Subred IPv4 | 192.168.10.0/24 |
| IP del servidor | 192.168.10.10 (estática) |
| Puerta de enlace | 192.168.10.1 |
| Servidor DHCP | VirtualBox: 192.168.10.2 |
| Rango DHCP | 192.168.10.100–192.168.10.200 |

### Máquinas virtuales

| Máquina | RAM | CPU |
|---|---:|---:|
| SRV-AD01 | 5,5 GB | 2 vCPU |
| CLIENTE-W10 | 4 GB | 2 vCPU |

Durante las pruebas observé un rendimiento insuficiente del cliente con 1 vCPU. Tras aumentar su asignación a 2 vCPU, experimenté una mejora notable de fluidez.

## 3. Active Directory y DNS

He instalado y configurado Active Directory Domain Services (AD DS) en SRV-AD01 y he promovido el servidor como controlador de dominio de un nuevo bosque.

- Bosque: `ad.lab.test`
- Dominio: `ad.lab.test`
- Nombre NetBIOS: `AD`
- Nivel funcional del dominio: Windows2025Domain
- Nivel funcional del bosque: Windows2025Forest
- Zonas DNS: `ad.lab.test` y `_msdcs.ad.lab.test`

He configurado DNS integrado con Active Directory y comprobado la resolución de nombres internos y externos.

También he verificado la localización del controlador de dominio mediante registros DNS SRV.

## 4. Estructura organizativa y gestión de identidades

He creado una estructura de unidades organizativas (OU) para representar tres departamentos:

- Administración
- Soporte IT
- Dirección

He creado tres cuentas de usuario y tres grupos de seguridad globales:

| Usuario | Departamento | Grupo |
|---|---|---|
| Laura Garcia | Administración | GG_Administracion |
| Carlos Ruiz | Soporte IT | GG_SoporteIT |
| Marta Lopez | Dirección | GG_Direccion |

He asociado cada usuario al grupo correspondiente y he comprobado las pertenencias mediante PowerShell.

## 5. Equipo cliente y unión al dominio

He instalado Windows 10 Pro como equipo cliente y configurado su conexión a la red virtual `Lab-AD`.

He instalado VirtualBox Guest Additions para mejorar la integración de la máquina virtual con el equipo anfitrión.

Posteriormente he unido `CLIENTE-W10` al dominio `ad.lab.test`.

He validado la pertenencia al dominio y he iniciado sesión utilizando las cuentas de Laura Garcia, Carlos Ruiz y Marta Lopez durante las pruebas de acceso a recursos.

## 6. Directivas de grupo (GPO)

He creado la directiva:

`GPO-Administracion-Restriccion-PanelControl`

La he vinculado a la OU `Administracion` para restringir el acceso al Panel de control y a la aplicación Configuración de Windows.

He comprobado su aplicación desde la sesión de Laura mediante:

- `gpupdate /force`
- `gpresult /r /scope:user`

También he verificado que la restricción impide abrir el Panel de control desde el equipo cliente.

## 7. Carpetas compartidas y permisos SMB/NTFS

He creado tres carpetas departamentales en el servidor:

```text
C:\Compartidos
├── Administracion
├── SoporteIT
└── Direccion
```

He configurado permisos NTFS mediante `icacls` y he publicado los recursos compartidos utilizando PowerShell y `New-SmbShare`.

He utilizado grupos de seguridad globales para centralizar la asignación de permisos, aplicando el principio de mínimo privilegio.

| Recurso | Grupo autorizado | Permiso NTFS | Permiso SMB |
|---|---|---|---|
| Administracion | GG_Administracion | Modificar | Change |
| SoporteIT | GG_SoporteIT | Modificar | Change |
| Direccion | GG_Direccion | Modificar | Change |

Los grupos departamentales disponen de permisos de modificación sobre su propio recurso. Los administradores conservan el control total y SYSTEM mantiene sus permisos administrativos NTFS.

### Validación desde el equipo cliente

He realizado las siguientes pruebas de acceso:

| Usuario | Administración | SoporteIT | Dirección |
|---|---|---|---|
| Laura Garcia | Permitido | Denegado | No probado |
| Carlos Ruiz | Denegado | Permitido | No probado |
| Marta Lopez | Denegado | Denegado | Permitido |

He comprobado los accesos permitidos mediante la creación de archivos de prueba y los accesos denegados mediante las respuestas de Windows.

Las dos combinaciones marcadas como no probadas no se presentan como resultados verificados.

## 8. Diagnóstico y observaciones técnicas

Durante la incorporación del cliente detecté una discrepancia en la entrega de DNS mediante DHCP.

Aunque modifiqué la opción DNS del servidor DHCP de VirtualBox, el cliente continuó recibiendo los servidores DNS anteriores.

Comprobé que los valores procedían de DHCP y configuré manualmente el cliente para utilizar `192.168.10.10` como servidor DNS.

Después verifiqué que podía localizar el controlador de dominio mediante el registro SRV de LDAP.

La resolución DNS del cliente quedó operativa, aunque la entrega de la opción DNS por DHCP sigue pendiente de investigar.

También ajusté los recursos de CPU y memoria de las máquinas virtuales después de observar problemas de rendimiento durante las pruebas.

## 9. Evidencias

Las capturas de configuración y validación se encuentran en la carpeta `capturas/`.

### Active Directory y red

- [Configuración DHCP de Lab-AD](capturas/dhcp-lab-ad-configuracion.png)
- [Modo funcional del bosque](capturas/modo-funcional-bosque.png)
- [Modo funcional del dominio](capturas/modo-funcional-dominio.png)
- [Usuarios y grupos](capturas/usuarios-y-grupos.png)
- [Resolución DNS interna](capturas/resolucion-dns-interno.png)
- [Resolución DNS externa](capturas/resolucion-dns-externo.png)
- [Localización del controlador desde el cliente](capturas/resolucion-dns-controlador-dominio.png)

### Dominio y GPO

- [Unión del cliente al dominio](capturas/cliente-unido-dominio.png)
- [Inicio de sesión de Laura](capturas/inicio-sesion-dominio-laura.png)
- [GPO vinculada](capturas/gpo-administracion-vinculada.png)
- [Configuración de la restricción del Panel de control](capturas/gpo-restriccion-panel-control.png)
- [GPO aplicada a Laura](capturas/gpo-aplicada-laura-gpresult.png)
- [Restricción del Panel de control comprobada](capturas/gpo-panel-control-bloqueado.png)

### Recursos compartidos

- [Acceso de Laura a Administración](capturas/acceso-smb-administracion-laura.png)
- [Denegación de acceso de Laura a SoporteIT](capturas/denegacion-smb-soporteit-laura.png)
- [Acceso de Carlos a SoporteIT](capturas/acceso-smb-soporteit-carlos.png)
- [Denegación de acceso de Carlos a Administración](capturas/denegacion-smb-administracion-carlos.png)
- [Acceso de Marta a Dirección](capturas/acceso-smb-direccion-marta.png)
- [Denegación de acceso de Marta a Administración](capturas/denegacion-smb-administracion-marta.png)
- [Denegación de acceso de Marta a SoporteIT](capturas/denegacion-smb-soporteit-marta.png)
- [Permisos NTFS de Dirección](capturas/permisos-ntfs-direccion.png)
- [Permisos SMB de Dirección](capturas/permisos-smb-direccion.png)

## 10. Aprendizajes y conclusiones

Durante el desarrollo he podido comprender de forma práctica cómo se relacionan Active Directory, las identidades, los grupos de seguridad, las directivas de grupo y los permisos sobre recursos compartidos.

He trabajado tanto con herramientas gráficas como con PowerShell para consultar configuraciones, administrar objetos y verificar resultados.

Una de las principales conclusiones ha sido comprender el flujo de diagnóstico de una incidencia de soporte: identificar al usuario afectado, comprobar su cuenta y pertenencia a grupos, revisar las directivas, los permisos y la conectividad, aplicar correcciones autorizadas y validar el resultado desde el equipo cliente.

También he comprobado que entender el funcionamiento de los componentes y sus relaciones es más importante que memorizar comandos de forma aislada.

Este laboratorio me ha permitido practicar tareas relacionadas con la administración de sistemas Windows y el soporte técnico, documentando las configuraciones y los resultados obtenidos.

## 11. Documentación

- [Active Directory y DNS](documentación/active-directory.md)
- [Administración de usuarios y grupos](documentación/usuarios-y-grupos.md)
- [Diagnóstico de red](documentación/diagnostico-red.md)
- [Directivas de grupo (GPO)](documentación/directivas-grupo.md)
- [Carpetas compartidas y permisos SMB/NTFS](documentación/carpetas-compartidas.md)

---

**Proyecto de aprendizaje práctico: infraestructura Windows, administración de sistemas y soporte IT.**

