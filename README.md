
# Guía de laboratorio: AD DS, DNS y DHCP en Windows Server 2022 con clientes Windows 10

**Autor:** Estudiante · Ingeniería de Software y Datos · IU Digital de Antioquia
**Fecha:** 16 de noviembre de 2026

---

## Contenido

- [1. Objetivo y topología del laboratorio](#1-objetivo-y-topología-del-laboratorio)
  - [1.1 Topología y valores del laboratorio](#11-topología-y-valores-del-laboratorio)
  - [1.2 Requisitos de las máquinas virtuales](#12-requisitos-de-las-máquinas-virtuales)
- [2. Preparación del servidor](#2-preparación-del-servidor)
  - [2.1 Nombre, IP fija, DNS y zona horaria](#21-nombre-ip-fija-dns-y-zona-horaria)
- [3. Instalación de AD DS y promoción a controlador de dominio](#3-instalación-de-ad-ds-y-promoción-a-controlador-de-dominio)
  - [3.1 Instalar el rol](#31-instalar-el-rol)
  - [3.2 Promover a controlador de dominio](#32-promover-a-controlador-de-dominio)
- [4. Configuración de DNS](#4-configuración-de-dns)
  - [4.1 Revisar la zona directa](#41-revisar-la-zona-directa)
  - [4.2 Crear la zona inversa](#42-crear-la-zona-inversa)
  - [4.3 Crear el PTR del servidor](#43-crear-el-ptr-del-servidor)
  - [4.4 Reenviadores (opcional)](#44-reenviadores-opcional)
- [5. Instalación y configuración de DHCP](#5-instalación-y-configuración-de-dhcp)
  - [5.1 Instalar el rol y autorizarlo](#51-instalar-el-rol-y-autorizarlo)
  - [5.2 Crear el ámbito](#52-crear-el-ámbito)
- [6. Estructura de Active Directory: OUs, usuarios y grupos](#6-estructura-de-active-directory-ous-usuarios-y-grupos)
  - [6.1 Diseño de la estructura](#61-diseño-de-la-estructura)
  - [6.2 Crear las OU, los grupos y los usuarios](#62-crear-las-ou-los-grupos-y-los-usuarios)
- [7. Recursos compartidos: carpetas y permisos](#7-recursos-compartidos-carpetas-y-permisos)
  - [7.1 Permisos por carpeta](#71-permisos-por-carpeta)
  - [7.2 Crear y compartir las carpetas](#72-crear-y-compartir-las-carpetas)
- [8. Políticas de seguridad con GPO](#8-políticas-de-seguridad-con-gpo)
  - [8.1 GPO del laboratorio](#81-gpo-del-laboratorio)
  - [8.2 Contraseñas, restricciones y seguridad del equipo](#82-contraseñas-restricciones-y-seguridad-del-equipo)
  - [8.3 Mapeo de unidades](#83-mapeo-de-unidades)
- [9. Unir el cliente Windows 10 al dominio](#9-unir-el-cliente-windows-10-al-dominio)
  - [9.1 Comprobar la red (en el cliente)](#91-comprobar-la-red-en-el-cliente)
  - [9.2 Unir el cliente al dominio](#92-unir-el-cliente-al-dominio)
  - [9.3 Mover el equipo a su OU (en el servidor)](#93-mover-el-equipo-a-su-ou-en-el-servidor)
  - [9.4 Iniciar sesión con un usuario del dominio (solo interfaz gráfica)](#94-iniciar-sesión-con-un-usuario-del-dominio-solo-interfaz-gráfica)
- [10. Verificación final](#10-verificación-final)
  - [10.1 Active Directory (en el servidor)](#101-active-directory-en-el-servidor)
  - [10.2 DNS con nslookup (en el cliente)](#102-dns-con-nslookup-en-el-cliente)
  - [10.3 DHCP](#103-dhcp)
  - [10.4 Cliente conectado al dominio](#104-cliente-conectado-al-dominio)
  - [10.5 GPO aplicadas](#105-gpo-aplicadas)
  - [10.6 Recursos compartidos](#106-recursos-compartidos)
  - [10.7 Evidencias para el informe (capturas propias)](#107-evidencias-para-el-informe-capturas-propias)
- [11. Solución de problemas comunes](#11-solución-de-problemas-comunes)
- [12. Fuentes de las capturas](#12-fuentes-de-las-capturas)

---

## Cómo usar esta guía

| Marca | Qué identifica | Ejemplo |
| --- | --- | --- |
| `N.` | Sección | **4.** Configuración de DNS |
| `N.M` | Paso de la sección | **4.2** Crear la zona inversa |
| `N.M.1` | El paso hecho con **PowerShell** | **4.2.1** PowerShell |
| `N.M.2` | **Comprobación** del paso en la interfaz gráfica | **4.2.2** Comprobar en la interfaz gráfica |
| `N.M.K.J` | Instrucción J dentro de una comprobación o de un paso de interfaz gráfica: su número empieza por el del título que la contiene | **4.2.2.1** dentro de 4.2.2 |
| `N.M.1.K` | Recuadro K de comandos de PowerShell o de `cmd`: su número empieza por el del título que lo contiene | **4.2.1.1** dentro de 4.2.1 |
| `10.7.K` | Evidencia para el informe | **10.7.3** Consola DHCP |

- Todo se configura con **PowerShell**. La interfaz gráfica aparece solo para **comprobar** lo que hizo PowerShell, o cuando no hay un comando equivalente: las asignaciones de unidades (8.3.2) y el inicio de sesión del usuario (9.4). Esos títulos dicen **Solo interfaz gráfica**.
- Ejecuta PowerShell **como administrador**. Si un comando no se reconoce en PowerShell 7, ejecútalo en **Windows PowerShell 5.1**, que viene con Windows Server y Windows 10.
- Cada sección tiene **su propio color**, que se repite en sus pasos, viñetas, tablas y bloques de código. Los colores, tamaños y márgenes se ven en la **vista previa de VS Code** (`Ctrl+Shift+V`, o `Ctrl+K V` para verla al lado del texto) y se adaptan al tema claro u oscuro.
- Los comandos van en **recuadros numerados**. Copia cada recuadro **completo** y pégalo en PowerShell (clic derecho o `Ctrl+V`); si la última línea queda escrita sin ejecutarse, pulsa **Enter**. Dentro de un recuadro, cada comando lleva encima un comentario (`# …`) que dice qué hace; PowerShell ignora esas líneas.
  - <span class="pega">pegar todo junto</span> El recuadro tiene varios comandos que pueden pegarse **a la vez**: PowerShell los ejecuta uno tras otro, en el orden en que aparecen.
  - Sin esa etiqueta, el recuadro tiene **un solo comando**. Algunos dicen **Pégalo solo**: reinician el equipo, pueden pedir una respuesta S/N o muestran algo que debes leer antes de seguir. Si los pegaras junto con otros, la línea siguiente se tomaría como respuesta o no llegaría a ejecutarse.
  - <span class="orden sec">secuencial</span> Configura algo. Pega estos recuadros en el orden de su número, cada uno cuando termine el anterior, y en la misma ventana: si un recuadro define una variable (`$base`, `$raiz`…), los siguientes la necesitan.
  - <span class="orden ind">comprobación</span> Solo consulta. Pégalo después de los recuadros secuenciales del mismo paso.
  - Cuando un paso tiene varios recuadros, la nota **Orden de ejecución** que aparece encima indica la secuencia.
- Los comandos se copian tal cual; solo cambia lo que el texto indique (por ejemplo, el nombre del adaptador de red). El texto después de `#` al final de una línea es un comentario con el resultado esperado.

> **Sobre las capturas de pantalla.** Las imágenes de este documento son capturas reales publicadas por terceros (la lista está en la sección 12). **No se copiaron**: el Markdown las enlaza desde su sitio original y cada una lleva su fuente debajo. Pertenecen a sus autores y se usan solo como referencia de estudio.
>
> - Todas las capturas están en **inglés**, y varias son de Windows Server 2016 o 2019; las consolas (*Administrador del servidor*, *DHCP*, *DNS*, *Usuarios y equipos de Active Directory*, *Administración de directivas de grupo*) son iguales en Windows Server 2022. Las ventanas de Windows Server 2022 en español tienen los mismos campos y en el mismo orden. Debajo de cada imagen se indica el texto equivalente en español.
> - Los nombres de dominio y las IP de las capturas son los del autor de cada tutorial. **Usa siempre los de la tabla de la sección 1.1.**
> - **Verificación (17/11/2026):** se abrieron en un navegador las 24 imágenes y se comprobó que cada una muestra el paso o el resultado descrito en su pie. Cuando una captura difiere de este laboratorio (por ejemplo, un DNS 8.8.8.8, otra versión de Windows Server o una opción distinta marcada), el pie lo indica.
> - **Dónde se ven:** en la prueba cargaron todas, tanto pidiéndolas desde otro sitio web como sin indicar la página de origen, así que deberían verse en GitHub, VS Code, Typora, Obsidian o publicadas en una página web. Las de Windows OS Hub están en formato WebP, que todos esos visores admiten. Si una imagen se moviera en el futuro, abre el enlace de la fuente que aparece debajo.
> - **Resolución:** las imágenes que se veían borrosas porque su sitio las publicaba reducidas se cambiaron por versiones más grandes. Las ventanas de diálogo de Windows (asistentes, *Propiedades*, *Nuevo objeto*) miden de verdad entre 330 y 500 píxeles de ancho; esas capturas están a su tamaño real y se ven nítidas al 100 %, pero se pixelan si las amplías más, porque no existe una versión más grande.
> - Las capturas que entregues como evidencia (sección 10.7) deben ser **de tu propio laboratorio**.
---

## 1. Objetivo y topología del laboratorio

Al terminar tendrás un controlador de dominio `SRV-DC01` que presta AD DS, DNS y DHCP al dominio `iudigital.local`. Tendrás también un cliente Windows 10 unido al dominio, una carpeta compartida con permisos por grupo y políticas de seguridad aplicadas por GPO.

### 1.1 Topología y valores del laboratorio

```mermaid
flowchart LR
    subgraph SRV["SRV-DC01 · Windows Server 2022 · 192.168.10.10"]
        DHCP["DHCP<br/>Ámbito .100–.200"]
        DNS["DNS<br/>iudigital.local + zona inversa"]
        ADDS["AD DS<br/>OUs, usuarios, grupos, GPO"]
        FS["Servidor de archivos<br/>Publico · Docentes · Estudiantes"]
    end
    CLI["CLI-W10-01<br/>Windows 10 Pro<br/>IP por DHCP"]
    CLI -- "1 · pide IP" --> DHCP
    CLI -- "2 · localiza el DC (SRV)" --> DNS
    CLI -- "3 · se une y autentica" --> ADDS
    CLI -- "4 · mapea S:, T: y U:" --> FS
```

*Figura 1. Topología del laboratorio: un solo servidor con cuatro roles y un cliente en la red interna 192.168.10.0/24. Diagrama propio en Mermaid; se dibuja en GitHub, GitLab, Obsidian y Typora. En VS Code, si ves el código en lugar del diagrama, instala la extensión *Markdown Preview Mermaid Support*.*

| Elemento | Valor de ejemplo | Notas |
| --- | --- | --- |
| Dominio (FQDN) | `iudigital.local` | Usa `.local` o `.lab`, nunca un dominio público real |
| Nombre NetBIOS | `IUDIGITAL` | Se genera al promover el servidor |
| Red | `192.168.10.0/24` | Máscara 255.255.255.0 |
| Servidor | `SRV-DC01` · `192.168.10.10` (fija) | AD DS + DNS + DHCP + archivos |
| Puerta de enlace | `192.168.10.1` | Déjala vacía si tu red interna no tiene router |
| DNS de los clientes | `192.168.10.10` | Siempre el DC, nunca 8.8.8.8 |
| Ámbito DHCP | `192.168.10.100` – `192.168.10.200` | Exclusión: .100 – .110 |
| Cliente | `CLI-W10-01` · IP por DHCP | Windows 10 Pro, Education o Enterprise; la edición Home no se une a dominios |
| Usuarios de prueba | `bienestaru`, `jdocente` | En la OU Usuarios |

### 1.2 Requisitos de las máquinas virtuales

- Windows Server 2022 Standard o Datacenter con **Experiencia de escritorio**: 2 vCPU, 4 GB de RAM y 50 GB de disco.
- Windows 10 Pro: 2 vCPU, 2–4 GB de RAM y 40 GB de disco.
- Las dos VM en la **misma red interna** y **sin el DHCP del hipervisor** en esa red.
- **Un solo adaptador de red por VM** durante el laboratorio. Con dos adaptadores, el controlador de dominio registra las dos IP en DNS y el cliente puede recibir la equivocada. Si el servidor necesita Internet un momento (por ejemplo, para actualizaciones), agrégale un adaptador NAT, actualiza y vuelve a quitarlo.
- Toma una **instantánea (snapshot)** antes de empezar y después de cada sección.

---

## 2. Preparación del servidor

Un controlador de dominio necesita IP fija, un nombre definitivo y su propia IP como DNS antes de instalar cualquier rol.

### 2.1 Nombre, IP fija, DNS y zona horaria

#### 2.1.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega **2.1.1.1** → **2.1.1.2** en este orden, cada recuadro cuando termine el anterior.

<span class="cmd">2.1.1.1</span><span class="orden sec">secuencial</span> **Pégalo solo** y mira el resultado: muestra los adaptadores de red. Si el tuyo no se llama *Ethernet*, cambia ese nombre en el recuadro siguiente antes de pegarlo.

```powershell
Get-NetAdapter   # nombre del adaptador, normalmente "Ethernet"
```

<span class="cmd">2.1.1.2</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> IP fija, DNS, zona horaria y nombre del equipo. La última línea **reinicia el servidor**, por eso va al final.

```powershell
# Asigna la IP fija, la máscara y la puerta de enlace
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.1

# Usa el propio servidor (127.0.0.1) como DNS
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1

# Ajusta la zona horaria a Bogotá
Set-TimeZone -Id "SA Pacific Standard Time"

# Cambia el nombre del equipo y reinicia el servidor
Rename-Computer -NewName "SRV-DC01" -Restart
```

Después del reinicio:

<span class="cmd">2.1.1.3</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba el nombre, la red y la zona horaria.

```powershell
# Nombre del equipo
hostname   # SRV-DC01

# IP, puerta de enlace y DNS del adaptador
Get-NetIPConfiguration -InterfaceAlias "Ethernet"   # 192.168.10.10, puerta 192.168.10.1, DNS 127.0.0.1

# Confirma que el adaptador ya no usa DHCP
(Get-NetIPInterface -InterfaceAlias "Ethernet" -AddressFamily IPv4).Dhcp   # Disabled

# Zona horaria
Get-TimeZone | Select-Object DisplayName   # (UTC-05:00) Bogotá, Lima, Quito...
```

#### 2.1.2 — Comprobar en la interfaz gráfica

- <span class="paso">2.1.2.1</span> **Administrador del servidor → Servidor local**: *Nombre del equipo* = `SRV-DC01`, *Ethernet* = `192.168.10.10` y *Zona horaria* = **(UTC-05:00) Bogotá, Lima, Quito**.
- <span class="paso">2.1.2.2</span> **Win + R** → `ncpa.cpl` → clic derecho en **Ethernet** → **Propiedades** → **Protocolo de Internet versión 4 (TCP/IPv4)** → **Propiedades**: *Usar la siguiente dirección IP* con los valores del paso 2.1.1 y *Servidor DNS preferido* = `127.0.0.1`.

![Propiedades de Ethernet en Windows Server 2022](https://www.layerstack.com/img/docs/resources/windows22staticip5.png)

*Figura 2. Ventana **Propiedades de Ethernet** en Windows Server 2022: selecciona **Protocolo de Internet versión 4 (TCP/IPv4)** y haz clic en **Propiedades** para ver los valores. La captura es de un servidor en la nube (adaptador *Red Hat VirtIO*); en tu VM el adaptador tendrá otro nombre. Fuente: [LayerStack — Static IP Address on Windows Server 2022](https://www.layerstack.com/resources/tutorials/How-to-configure-the-Static-IP-Address-Network-Card-for-Windows-2022-Cloud-Servers). Interfaz en inglés.*

---

## 3. Instalación de AD DS y promoción a controlador de dominio

Al promover el servidor como primer DC de un bosque nuevo, se instala y configura DNS solo, con la zona `iudigital.local` integrada en AD.

### 3.1 Instalar el rol

#### 3.1.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **3.1.1.1** y espera a que termine; después, la comprobación **3.1.1.2**.

<span class="cmd">3.1.1.1</span><span class="orden sec">secuencial</span> Instala el rol AD DS con sus consolas de administración. Tarda unos minutos.

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

<span class="cmd">3.1.1.2</span><span class="orden ind">comprobación</span> Comprueba que el rol quedó instalado.

```powershell
Get-WindowsFeature AD-Domain-Services   # Install State = Installed
```

#### 3.1.2 — Comprobar en la interfaz gráfica

- <span class="paso">3.1.2.1</span> **Administrador del servidor → Panel**: en el panel izquierdo aparece **AD DS**, lo que confirma que el rol quedó instalado.
- <span class="paso">3.1.2.2</span> La bandera de notificaciones muestra un aviso con el enlace **Promover este servidor a controlador de dominio**. **No hagas clic en él**: la promoción se hace con PowerShell en el paso 3.2.

### 3.2 Promover a controlador de dominio

#### 3.2.1 — PowerShell (como administrador)

<span class="cmd">3.2.1.1</span> Crea el bosque y el dominio `iudigital.local` con DNS. Es **un solo comando** en cuatro líneas (las tres primeras terminan en `` ` ``): **pégalo solo**, porque pide la contraseña DSRM y después reinicia el servidor.

```powershell
Install-ADDSForest `
  -DomainName "iudigital.local" -DomainNetbiosName "IUDIGITAL" `
  -ForestMode "WinThreshold" -DomainMode "WinThreshold" -InstallDns `
  -SafeModeAdministratorPassword (Read-Host -AsSecureString "Contraseña DSRM") -Force
```

`WinThreshold` es el nivel funcional **Windows Server 2016**, el más alto disponible en Windows Server 2022. El servidor se reinicia solo; después inicia sesión como `IUDIGITAL\Administrador` y ejecuta:

<span class="cmd">3.2.1.2</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba el dominio, el bosque y los servicios.

```powershell
# Nombre del dominio, nombre NetBIOS y nivel funcional del dominio
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode   # iudigital.local, IUDIGITAL, Windows2016Domain

# Nivel funcional del bosque
Get-ADForest | Select-Object ForestMode   # Windows2016Forest

# Servicios de AD DS y DNS
Get-Service NTDS, DNS, Netlogon, ADWS | Select-Object Name, Status   # todos Running
```

#### 3.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">3.2.2.1</span> **Administrador del servidor**: en el panel izquierdo aparecen **AD DS** y **DNS**.
- <span class="paso">3.2.2.2</span> **Herramientas → Dominios y confianzas de Active Directory** → clic derecho en `iudigital.local` → **Propiedades**: *Nivel funcional del dominio* y *Nivel funcional del bosque* = **Windows Server 2016** (figura 3).

![Propiedades del dominio con los niveles funcionales del dominio y del bosque](https://petri.com/wp-content/uploads/2026/01/2026-01-16_09-57-56-1024x703.png)

*Figura 3. Ventana **Properties** (*Propiedades*) del dominio, pestaña **General**: *Domain functional level* y *Forest functional level* (*Nivel funcional del dominio* y *Nivel funcional del bosque*) = **Windows Server 2016**, como debe verse en `iudigital.local`. El autor la abrió desde *Usuarios y equipos de Active Directory* (clic derecho en el dominio → *Propiedades*); desde *Dominios y confianzas de Active Directory* se abre la misma ventana. Fuente: [Petri — Active Directory Functional Levels Explained](https://petri.com/active-directory-functional-levels/). Interfaz en inglés.*

- <span class="paso">3.2.2.3</span> **Herramientas → Usuarios y equipos de Active Directory** → `iudigital.local` → **Domain Controllers**: aparece `SRV-DC01`.

---

## 4. Configuración de DNS

La zona directa `iudigital.local` ya existe después de la promoción. Faltan la zona inversa, para que `nslookup` resuelva de IP a nombre, el registro PTR del servidor y los reenviadores.

### 4.1 Revisar la zona directa

#### 4.1.1 — PowerShell

<span class="cmd">4.1.1.1</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Revisa las zonas y los registros A y SRV.

```powershell
# Lista las zonas del servidor DNS
Get-DnsServerZone

# Registros A de la zona directa
Get-DnsServerResourceRecord -ZoneName "iudigital.local" -RRType A

# Registro SRV de LDAP que usan los clientes para encontrar el DC
Get-DnsServerResourceRecord -ZoneName "iudigital.local" -RRType SRV -Name "_ldap._tcp"
```

Debe aparecer la zona `iudigital.local`, el registro A `srv-dc01` → `192.168.10.10` y el registro SRV `_ldap._tcp` → `srv-dc01.iudigital.local`.

#### 4.1.2 — Comprobar en la interfaz gráfica

- <span class="paso">4.1.2.1</span> **Herramientas → DNS → SRV-DC01 → Zonas de búsqueda directa → iudigital.local**: registros **SOA**, **NS** y **A** (`srv-dc01 → 192.168.10.10`) y las carpetas `_msdcs`, `_sites`, `_tcp` y `_udp`, como en la figura 18.

### 4.2 Crear la zona inversa

#### 4.2.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **4.2.1.1** y espera a que termine; después, la comprobación **4.2.1.2**.

<span class="cmd">4.2.1.1</span><span class="orden sec">secuencial</span> Crea la zona inversa integrada en AD, con actualizaciones seguras.

```powershell
Add-DnsServerPrimaryZone -NetworkId "192.168.10.0/24" -ReplicationScope "Domain" -DynamicUpdate Secure
```

<span class="cmd">4.2.1.2</span><span class="orden ind">comprobación</span> Comprueba la zona creada.

```powershell
Get-DnsServerZone -Name "10.168.192.in-addr.arpa" | Select-Object ZoneName, IsDsIntegrated, DynamicUpdate
```

#### 4.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">4.2.2.1</span> **Herramientas → DNS → SRV-DC01 → Zonas de búsqueda inversa**: aparece `10.168.192.in-addr.arpa`.
- <span class="paso">4.2.2.2</span> Clic derecho en la zona → **Propiedades** → pestaña **General**: *Tipo* = **Integrada en Active Directory** y *Actualizaciones dinámicas* = **Solo seguras**.

### 4.3 Crear el PTR del servidor

#### 4.3.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **4.3.1.1** y espera a que termine; después, la comprobación **4.3.1.2**.

<span class="cmd">4.3.1.1</span><span class="orden sec">secuencial</span> Crea el registro PTR del servidor.

```powershell
Add-DnsServerResourceRecordPtr -ZoneName "10.168.192.in-addr.arpa" -Name "10" -PtrDomainName "srv-dc01.iudigital.local"
```

Si responde que el registro ya existe, el servidor lo registró solo al crear la zona: no es un error, sigue con la comprobación.

<span class="cmd">4.3.1.2</span><span class="orden ind">comprobación</span> Comprueba el PTR.

```powershell
Get-DnsServerResourceRecord -ZoneName "10.168.192.in-addr.arpa" -RRType Ptr
```

#### 4.3.2 — Comprobar en la interfaz gráfica

- <span class="paso">4.3.2.1</span> En la zona `10.168.192.in-addr.arpa` aparece un **Puntero (PTR)** `192.168.10.10` → `srv-dc01.iudigital.local`, como en la parte superior de la figura 15.

### 4.4 Reenviadores (opcional)

#### 4.4.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **4.4.1.1** y espera a que termine; después, la comprobación **4.4.1.2**.

<span class="cmd">4.4.1.1</span><span class="orden sec">secuencial</span> Configura los reenviadores.

```powershell
Set-DnsServerForwarder -IPAddress 8.8.8.8, 1.1.1.1
```

<span class="cmd">4.4.1.2</span><span class="orden ind">comprobación</span> Comprueba los reenviadores.

```powershell
Get-DnsServerForwarder
```

#### 4.4.2 — Comprobar en la interfaz gráfica

- <span class="paso">4.4.2.1</span> **Herramientas → DNS** → clic derecho en `SRV-DC01` → **Propiedades** → pestaña **Reenviadores**: aparecen `8.8.8.8` y `1.1.1.1`.

![Pestaña Reenviadores en las propiedades del servidor DNS](https://learn.microsoft.com/en-us/training/wwl-windows-server/implement-windows-server-dns/media/m12-dns-3.png)

*Figura 4. Pestaña **Forwarders** (*Reenviadores*) en las propiedades del servidor DNS, con un reenviador configurado. En tu servidor deben aparecer `8.8.8.8` y `1.1.1.1`. Fuente: [Microsoft Learn — Implement DNS forwarding](https://learn.microsoft.com/en-us/training/modules/implement-windows-server-dns/5-implement-dns-forwarding).*

---

## 5. Instalación y configuración de DHCP

El DHCP entrega a cada cliente su IP, la puerta de enlace y, lo más importante, el DNS `192.168.10.10`. En un dominio, el servidor DHCP tiene que estar **autorizado en AD**.

### 5.1 Instalar el rol y autorizarlo

#### 5.1.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **5.1.1.1** y espera a que termine; después, la comprobación **5.1.1.2**.

<span class="cmd">5.1.1.1</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Instala el rol DHCP, crea sus grupos de seguridad, lo autoriza en AD y quita el aviso del Administrador del servidor. La instalación tarda unos minutos; las líneas siguientes esperan a que termine.

```powershell
# Instala el rol DHCP con su consola
Install-WindowsFeature DHCP -IncludeManagementTools

# Crea los grupos locales Administradores de DHCP y Usuarios de DHCP
netsh dhcp add securitygroups

# Reinicia el servicio para que use esos grupos
Restart-Service dhcpserver

# Autoriza el servidor DHCP en AD
Add-DhcpServerInDC -DnsName "srv-dc01.iudigital.local" -IPAddress 192.168.10.10

# Quita el aviso Completar la configuración de DHCP del Administrador del servidor
Set-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\ServerManager\Roles\12 -Name ConfigurationState -Value 2
```

<span class="cmd">5.1.1.2</span><span class="orden ind">comprobación</span> Comprueba la autorización.

```powershell
Get-DhcpServerInDC   # debe listar srv-dc01.iudigital.local
```

La línea `Set-ItemProperty` quita el aviso *Completar la configuración de DHCP* del Administrador del servidor.

#### 5.1.2 — Comprobar en la interfaz gráfica

- <span class="paso">5.1.2.1</span> **Herramientas → DHCP** → `srv-dc01.iudigital.local`: el nodo **IPv4** tiene una **marca verde**, es decir, el servidor está autorizado. Si muestra una flecha roja, repite `Add-DhcpServerInDC`.

![Consola DHCP con el nodo IPv4 y los nodos del ámbito](https://learn.microsoft.com/en-us/training/wwl-windows-server/deploy-manage-dynamic-host-configuration-protocol/media/m11-dynamic-host-configuration-protocol-1.png)

*Figura 5. Consola **DHCP** con el servidor `sea-dc1`: el nodo **IPv4** con el icono verde (servidor autorizado) y, debajo, el ámbito con *Address Pool* (*Grupo de direcciones*), *Address Leases* (*Concesiones de direcciones*), *Reservations* (*Reservas*) y *Scope Options* (*Opciones de ámbito*), los nodos que revisarás en el paso 5.2.2. En tu servidor el nodo es `srv-dc01.iudigital.local` y el ámbito, `[192.168.10.0] LAN-IUDigital`. Fuente: [Microsoft Learn — Install and configure the DHCP role](https://learn.microsoft.com/en-us/training/modules/deploy-manage-dynamic-host-configuration-protocol/3-install-configure-dynamic-host-configuration-protocol). Interfaz en inglés.*

- <span class="paso">5.1.2.2</span> La bandera de notificaciones del Administrador del servidor no muestra el aviso *Completar la configuración de DHCP*.

### 5.2 Crear el ámbito

#### 5.2.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **5.2.1.1** y espera a que termine; después, la comprobación **5.2.1.2**.

<span class="cmd">5.2.1.1</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Crea el ámbito, la exclusión y las opciones.

```powershell
# Crea el ámbito LAN-IUDigital y lo activa
Add-DhcpServerv4Scope -Name "LAN-IUDigital" -StartRange 192.168.10.100 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0 -State Active

# Excluye las direcciones .100–.110
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.10.0 -StartRange 192.168.10.100 -EndRange 192.168.10.110

# Configura las opciones del ámbito: puerta de enlace, DNS y dominio
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -Router 192.168.10.1 -DnsServer 192.168.10.10 -DnsDomain "iudigital.local"
```

<span class="cmd">5.2.1.2</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba el ámbito, la exclusión y las opciones.

```powershell
# Muestra el ámbito
Get-DhcpServerv4Scope

# Muestra la exclusión
Get-DhcpServerv4ExclusionRange -ScopeId 192.168.10.0

# Muestra las opciones del ámbito
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
```

#### 5.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">5.2.2.1</span> **Herramientas → DHCP → srv-dc01.iudigital.local → IPv4**: el ámbito **[192.168.10.0] LAN-IUDigital** está **Activo**, como en la figura 17.
- <span class="paso">5.2.2.2</span> **Grupo de direcciones**: intervalo `192.168.10.100` – `192.168.10.200` y exclusión `192.168.10.100` – `192.168.10.110`.
- <span class="paso">5.2.2.3</span> **Opciones de ámbito**: `003 Enrutador` = `192.168.10.1`, `006 Servidores DNS` = `192.168.10.10` y `015 Nombre de dominio DNS` = `iudigital.local`.

---

## 6. Estructura de Active Directory: OUs, usuarios y grupos

Las GPO se vinculan a unidades organizativas (OU), no a los contenedores predeterminados *Users* y *Computers*. Por eso primero se crea una OU propia.

### 6.1 Diseño de la estructura

| Objeto | Tipo | Ubicación | Miembros |
| --- | --- | --- | --- |
| IUDigital | OU | raíz del dominio | — |
| Usuarios / Equipos / Grupos | OU | IUDigital | — |
| bienestaru (Bienestar Universitario) | Usuario | IUDigital/Usuarios | GG_Estudiantes |
| jdocente (Docente Prueba) | Usuario | IUDigital/Usuarios | GG_Docentes |
| GG_Estudiantes, GG_Docentes | Grupo global de seguridad | IUDigital/Grupos | — |

### 6.2 Crear las OU, los grupos y los usuarios

#### 6.2.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega **6.2.1.1** → **6.2.1.2** en este orden, cada recuadro cuando termine el anterior; después, la comprobación **6.2.1.3**.

<span class="cmd">6.2.1.1</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Crea las OU y los grupos.

```powershell
# Define la variable con el dominio; la usan todos los comandos siguientes
$base = "DC=iudigital,DC=local"

# Crea la OU IUDigital
New-ADOrganizationalUnit -Name "IUDigital" -Path $base

# Crea las OU hijas Usuarios, Equipos y Grupos
"Usuarios","Equipos","Grupos" | ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path "OU=IUDigital,$base" }

# Crea el grupo GG_Estudiantes
New-ADGroup -Name "GG_Estudiantes" -GroupScope Global -GroupCategory Security -Path "OU=Grupos,OU=IUDigital,$base"

# Crea el grupo GG_Docentes
New-ADGroup -Name "GG_Docentes"    -GroupScope Global -GroupCategory Security -Path "OU=Grupos,OU=IUDigital,$base"
```

<span class="cmd">6.2.1.2</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Pide la contraseña inicial, crea los usuarios con ella y los agrega a sus grupos. Al pegar el recuadro aparece *Contraseña inicial*: escríbela (no se ve mientras la escribes) y pulsa **Enter**; los comandos siguientes continúan solos. Los comandos `New-ADUser` ocupan varias líneas (terminan en `` ` ``): el recuadro se copia entero.

```powershell
# Pide la contraseña inicial de los usuarios y la guarda en $clave
$clave = Read-Host -AsSecureString "Contraseña inicial"

# Crea el usuario bienestaru
New-ADUser -Name "Bienestar Universitario" -GivenName "Bienestar" -Surname "Universitario" -SamAccountName "bienestaru" `
  -UserPrincipalName "bienestaru@iudigital.local" -Path "OU=Usuarios,OU=IUDigital,DC=iudigital,DC=local" `
  -AccountPassword $clave -Enabled $true -ChangePasswordAtLogon $true

# Crea el usuario jdocente
New-ADUser -Name "Docente Prueba" -SamAccountName "jdocente" -UserPrincipalName "jdocente@iudigital.local" `
  -Path "OU=Usuarios,OU=IUDigital,DC=iudigital,DC=local" -AccountPassword $clave -Enabled $true

# Agrega bienestaru a GG_Estudiantes
Add-ADGroupMember -Identity "GG_Estudiantes" -Members "bienestaru"

# Agrega jdocente a GG_Docentes
Add-ADGroupMember -Identity "GG_Docentes"    -Members "jdocente"
```

<span class="cmd">6.2.1.3</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba las OU y los miembros de los grupos.

```powershell
# Lista las OU creadas
Get-ADOrganizationalUnit -Filter * -SearchBase "OU=IUDigital,DC=iudigital,DC=local" | Select-Object Name

# Miembros de GG_Estudiantes
Get-ADGroupMember GG_Estudiantes | Select-Object SamAccountName   # bienestaru

# Miembros de GG_Docentes
Get-ADGroupMember GG_Docentes    | Select-Object SamAccountName   # jdocente
```

#### 6.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">6.2.2.1</span> **Herramientas → Usuarios y equipos de Active Directory** → `iudigital.local → IUDigital`: aparecen las OU **Usuarios**, **Equipos** y **Grupos**, como en la figura 19.
- <span class="paso">6.2.2.2</span> En **Grupos**, doble clic en `GG_Estudiantes` → pestaña **Miembros**: `Bienestar Universitario`. En `GG_Docentes`: `Docente Prueba`.
- <span class="paso">6.2.2.3</span> En **Usuarios**, doble clic en `Bienestar Universitario` → pestaña **Cuenta**: *Nombre de inicio de sesión* = `bienestaru@iudigital.local`.

---

## 7. Recursos compartidos: carpetas y permisos

Por la red se aplica el permiso **más restrictivo** entre el del recurso compartido y el NTFS. Lo recomendado es dar al recurso *Usuarios autentificados: Control total* y controlar el acceso real con NTFS, por grupos.

### 7.1 Permisos por carpeta

| Carpeta (en `C:\Compartidos`) | Recurso | GG_Docentes | GG_Estudiantes | Administradores |
| --- | --- | --- | --- | --- |
| Publico | `\\SRV-DC01\Publico` | Modificar el contenido, sin poder borrar la carpeta | Lectura y ejecución | Control total |
| Docentes | `\\SRV-DC01\Docentes` | Modificar el contenido, sin poder borrar la carpeta | Sin acceso | Control total |
| Estudiantes | `\\SRV-DC01\Estudiantes` | Modificar el contenido, sin poder borrar la carpeta | Leer todo y crear; modificar o borrar **solo lo que cada uno creó** | Control total |

- **Docentes:** reciben dos entradas. Sobre la carpeta misma, leer y crear, pero **sin Eliminar**. Sobre lo que hay dentro, **Modificar**. Así trabajan con el contenido, pero no pueden borrar la carpeta.
- **Carpeta superior `C:\Compartidos`:** solo **Administradores** y **SYSTEM** pueden modificarla; los demás usuarios solo la leen. Para borrar una carpeta hace falta *Eliminar* sobre ella o *Eliminar subcarpetas y archivos* sobre la carpeta que la contiene, y ningún grupo de usuarios tiene ninguno de los dos.
- **Estudiantes en `Estudiantes`:** leen todo y pueden crear archivos y carpetas. La entrada **CREATOR OWNER** (`S-1-3-0`) les da *Modificar* solo sobre lo que cada uno creó, así que no pueden borrar los trabajos de otros.

### 7.2 Crear y compartir las carpetas

#### 7.2.1 — PowerShell (como administrador)

El script resuelve los grupos integrados por su SID, así funciona con Windows en español o en inglés. Si ya habías creado las carpetas con una versión anterior de esta guía, puedes ejecutarlo igual: quita los permisos anteriores de los grupos antes de asignar los nuevos.

<span class="nota-orden">Orden de ejecución</span> Pega primero **7.2.1.1** y espera a que termine; después, la comprobación **7.2.1.2**.

<span class="cmd">7.2.1.1</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Crea las carpetas, las comparte y asigna los permisos NTFS. El bloque `ForEach-Object { … }` es un solo comando: el recuadro se copia entero.

```powershell
# Variables: Usuarios autentificados y Administradores (por su SID),
# los dos grupos del dominio y la carpeta raíz
$auth   = (New-Object Security.Principal.SecurityIdentifier "S-1-5-11").Translate([Security.Principal.NTAccount]).Value
$admins = (New-Object Security.Principal.SecurityIdentifier "S-1-5-32-544").Translate([Security.Principal.NTAccount]).Value
$doc    = "IUDIGITAL\GG_Docentes"
$est    = "IUDIGITAL\GG_Estudiantes"
$raiz   = "C:\Compartidos"

# Crea la carpeta superior C:\Compartidos
New-Item -Path $raiz -ItemType Directory -Force | Out-Null

# Carpeta superior: solo Administradores y SYSTEM pueden modificarla
# (los demás solo la leen y no pueden borrar las carpetas desde aquí)
icacls $raiz /inheritance:r /grant "*S-1-5-32-544:(OI)(CI)F" "*S-1-5-18:(OI)(CI)F" "*S-1-5-11:(RX)" | Out-Null

# Para cada carpeta: la crea, la comparte con enumeración basada en el acceso
# y asigna permisos NTFS (GG_Docentes sin Eliminar sobre la carpeta misma)
"Publico","Docentes","Estudiantes" | ForEach-Object {
  $c = "$raiz\$_"
  New-Item -Path $c -ItemType Directory -Force | Out-Null
  if (-not (Get-SmbShare -Name $_ -ErrorAction SilentlyContinue)) {
    New-SmbShare -Name $_ -Path $c -FullAccess $auth, $admins -FolderEnumerationMode AccessBased | Out-Null
  }
  icacls $c /inheritance:r /grant "*S-1-5-32-544:(OI)(CI)F" "*S-1-5-18:(OI)(CI)F" | Out-Null
  icacls $c /remove:g $doc | Out-Null
  icacls $c /remove:g $est | Out-Null
  icacls $c /remove:g "*S-1-3-0" | Out-Null
  icacls $c /grant "${doc}:(RX,W)" "${doc}:(OI)(CI)(IO)M" | Out-Null   # docentes: sin Eliminar sobre la carpeta
}

# Da a GG_Estudiantes lectura en Publico
icacls "$raiz\Publico"     /grant "${est}:(OI)(CI)RX"

# Da a GG_Estudiantes lectura y creación en Estudiantes, y a CREATOR OWNER modificar solo lo que cada uno crea
icacls "$raiz\Estudiantes" /grant "${est}:(OI)(CI)RX" "${est}:(W)" "*S-1-3-0:(OI)(CI)(IO)M"
```

<span class="cmd">7.2.1.2</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba los recursos compartidos y los permisos.

```powershell
# Muestra los tres recursos compartidos y su modo de enumeración
Get-SmbShare -Name Publico, Docentes, Estudiantes | Select-Object Name, Path, FolderEnumerationMode

# Permisos del recurso compartido Docentes
Get-SmbShareAccess -Name Docentes

# Permisos NTFS de Docentes
icacls "C:\Compartidos\Docentes"

# Permisos NTFS de Estudiantes
icacls "C:\Compartidos\Estudiantes"
```

En la salida de `icacls`, `GG_Docentes` debe tener dos líneas, `(RX,W)` y `(OI)(CI)(IO)(M)`, y en `Estudiantes` debe aparecer `CREATOR OWNER:(OI)(CI)(IO)(M)`.

Para comprobar que nadie, salvo los administradores, puede borrar las carpetas:

<span class="cmd">7.2.1.3</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Define las variables y muestra quién puede borrar cada carpeta: la columna `PuedeBorrarla` debe ser **False** en todas las filas, y en `C:\Compartidos` solo deben aparecer **Administradores** y **SYSTEM**.

```powershell
# Define las variables: la carpeta raíz y los permisos que se van a buscar
$raiz = "C:\Compartidos"
$del = [Security.AccessControl.FileSystemRights]::Delete
$delHijos = [Security.AccessControl.FileSystemRights]::DeleteSubdirectoriesAndFiles
$soloHijos = [Security.AccessControl.PropagationFlags]::InheritOnly

# Permisos de los grupos sobre cada carpeta misma: la columna PuedeBorrarla debe ser False en todas las filas
foreach ($c in "Publico","Docentes","Estudiantes") {
  (Get-Acl "$raiz\$c").Access |
    Where-Object { $_.IdentityReference -match 'GG_' -and (($_.PropagationFlags -band $soloHijos) -eq 0) } |
    Select-Object @{n='Carpeta';e={$c}}, IdentityReference, FileSystemRights,
                  @{n='PuedeBorrarla';e={($_.FileSystemRights -band $del) -ne 0}}
}

# Quién puede borrar dentro de C:\Compartidos: solo deben salir Administradores y SYSTEM
(Get-Acl $raiz).Access |
  Where-Object { (($_.PropagationFlags -band $soloHijos) -eq 0) -and (($_.FileSystemRights -band ($del -bor $delHijos)) -ne 0) } |
  Select-Object IdentityReference, FileSystemRights
```

#### 7.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">7.2.2.1</span> En el **Explorador de archivos**, clic derecho en `C:\Compartidos\Docentes` → **Propiedades** → pestaña **Compartir**: *Ruta de acceso de red* = `\\SRV-DC01\Docentes`.

![Pestaña Compartir de las propiedades de una carpeta compartida](https://cdn.sanity.io/images/r09655ln/production/2f34aac247116486bb032ed42120824c18973d3a-356x480.png)

*Figura 6. Pestaña **Sharing** (*Compartir*) de una carpeta compartida: *Network Path* (*Ruta de acceso de red*) y el botón **Advanced Sharing…** (*Uso compartido avanzado*). En tu servidor la ruta será `\\SRV-DC01\Docentes`. Fuente: [Netwrix — NTFS vs. Share Permissions](https://netwrix.com/en/resources/blog/ntfs-vs-share-permissions/). Interfaz en inglés.*

- <span class="paso">7.2.2.2</span> **Uso compartido avanzado**: *Compartir esta carpeta* está marcada.

- <span class="paso">7.2.2.3</span> **Permisos**: solo **Usuarios autentificados** y **Administradores**, ambos con *Control total*.

![Permisos del recurso compartido con Usuarios autentificados y Administradores](https://activedirectorypro.com/images/posts/ntfs-vs-share-permissions/ntfs-vs-share-permissions-2.png)

*Figura 7. **Permissions for share** (*Permisos del recurso*) con *Authenticated Users* (*Usuarios autentificados*) y *Administrators* (*Administradores*), los mismos grupos del laboratorio. En la captura *Authenticated Users* tiene solo *Read*; en tu servidor los dos deben tener *Full Control* (*Control total*), y no debe aparecer *Everyone* (*Todos*). Fuente: [ActiveDirectoryPro — NTFS vs Share Permissions](https://activedirectorypro.com/ntfs-vs-share-permissions/). Interfaz en inglés.*

- <span class="paso">7.2.2.4</span> Pestaña **Seguridad → Opciones avanzadas**: la herencia está deshabilitada y hay dos entradas para `GG_Docentes`, una con *Se aplica a* = **Solo esta carpeta** (sin *Eliminar*) y otra con **Solo subcarpetas y archivos** (*Modificar*). En `Estudiantes`, además, **CREATOR OWNER** con *Modificar* en **Solo subcarpetas y archivos**.

![Pestaña Seguridad de una carpeta con el botón Opciones avanzadas](https://activedirectorypro.com/images/posts/ntfs-vs-share-permissions/ntfs-vs-share-permissions-1.png)

*Figura 8. Pestaña **Security** (*Seguridad*) de una carpeta: la lista de grupos (aquí, entre otros, *CREATOR OWNER* y *Authenticated Users*) y el botón **Advanced** (*Opciones avanzadas*), que abre la configuración de seguridad avanzada. Fuente: [ActiveDirectoryPro](https://activedirectorypro.com/ntfs-vs-share-permissions/). Interfaz en inglés.*

- <span class="paso">7.2.2.5</span> Haz doble clic en la entrada de `GG_Docentes` que dice **Solo esta carpeta**: se abre **Entrada de permiso**, donde la casilla *Eliminar* debe estar desmarcada. En la otra entrada, la de **Solo subcarpetas y archivos**, *Eliminar* sí está marcada.
- <span class="paso">7.2.2.6</span> En **Opciones avanzadas** → pestaña **Acceso efectivo** → **Seleccionar un usuario** → `jdocente` → **Ver acceso efectivo**: *Eliminar* aparece con una **X roja** en `Publico`, `Docentes` y `Estudiantes`. Repite en `Estudiantes` con `bienestaru`, y en `C:\Compartidos` con los dos usuarios: ahí también *Eliminar subcarpetas y archivos* debe tener la X roja.
- <span class="paso">7.2.2.7</span> **Administrador del servidor → Servicios de archivos y de almacenamiento → Recursos compartidos** → clic derecho en `Docentes` → **Propiedades → Configuración**: *Habilitar la enumeración basada en el acceso* está marcada (figura 9).

![Propiedades de un recurso compartido en el Administrador del servidor con la enumeración basada en el acceso habilitada](https://woshub.com/wp-content/uploads/2019/01/server-manager-share-enable-access-based-enumeration.jpg.webp)

*Figura 9. **Server Manager → File and Storage Services → Shares** (*Administrador del servidor → Servicios de archivos y de almacenamiento → Recursos compartidos*): en las propiedades del recurso, sección **Settings** (*Configuración*), la casilla **Enable access-based enumeration** (*Habilitar la enumeración basada en el acceso*) está marcada. En la captura el recurso se llama *Docs*; en tu servidor, `Docentes`. Fuente: [Windows OS Hub — Enable Access-based Enumeration (ABE) on Shared Folders](https://woshub.com/enable-access-based-enumeration-in-windows-server/). Interfaz en inglés.*

---

## 8. Políticas de seguridad con GPO

### 8.1 GPO del laboratorio

| GPO | Vinculada a | Ruta en el editor | Configuración |
| --- | --- | --- | --- |
| Default Domain Policy | iudigital.local | Configuración del equipo → Directivas → Configuración de Windows → Configuración de seguridad → Directivas de cuenta | Contraseña: mínimo 8, complejidad, vigencia 90 días, historial 5 · Bloqueo: 5 intentos, 15 min |
| GPO_Restricciones_Usuarios | OU Usuarios | Configuración de usuario → Directivas → Plantillas administrativas → Panel de control / Sistema | *Prohibir el acceso a Configuración de PC y a Panel de control* · *Impedir el acceso al símbolo del sistema* · *Impedir el acceso a herramientas de edición del Registro* |
| GPO_Mapeo_Unidades | OU Usuarios | Configuración de usuario → Preferencias → Configuración de Windows → Asignaciones de unidades | `S:` → `\\SRV-DC01\Publico` · `T:` → `\\SRV-DC01\Docentes`, solo para GG_Docentes · `U:` → `\\SRV-DC01\Estudiantes`, para GG_Estudiantes y GG_Docentes |
| GPO_Seguridad_Equipos | OU Equipos | Valores del Registro de *Directivas locales → Opciones de seguridad* | Aviso legal al iniciar sesión · *No mostrar el último inicio de sesión* |

> Las restricciones van en la OU Usuarios, donde no está la cuenta Administrador; así no te bloqueas a ti mismo.

### 8.2 Contraseñas, restricciones y seguridad del equipo

#### 8.2.1 — PowerShell (como administrador)

<span class="nota-orden">Orden de ejecución</span> Pega primero **8.2.1.1** y espera a que termine; después, la comprobación **8.2.1.2**.

<span class="cmd">8.2.1.1</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Configura la directiva de contraseñas y crea, vincula y configura las dos GPO.

```powershell
# Carga el módulo de directivas de grupo
Import-Module GroupPolicy

# Define las variables con las OU Usuarios y Equipos
$ouUsr = "OU=Usuarios,OU=IUDigital,DC=iudigital,DC=local"
$ouEq  = "OU=Equipos,OU=IUDigital,DC=iudigital,DC=local"

# Configura la directiva de contraseñas y de bloqueo del dominio (Default Domain Policy)
Set-ADDefaultDomainPasswordPolicy -Identity iudigital.local -MinPasswordLength 8 -ComplexityEnabled $true `
  -MaxPasswordAge 90.00:00:00 -PasswordHistoryCount 5 -LockoutThreshold 5 -LockoutDuration 00:15:00 -LockoutObservationWindow 00:15:00

# Crea GPO_Restricciones_Usuarios y la vincula a la OU Usuarios
New-GPO -Name "GPO_Restricciones_Usuarios" | New-GPLink -Target $ouUsr

# Bloquea el Panel de control
Set-GPRegistryValue -Name "GPO_Restricciones_Usuarios" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName NoControlPanel -Type DWord -Value 1

# Bloquea el símbolo del sistema
Set-GPRegistryValue -Name "GPO_Restricciones_Usuarios" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName DisableCMD -Type DWord -Value 1

# Bloquea el Editor del Registro
Set-GPRegistryValue -Name "GPO_Restricciones_Usuarios" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" -ValueName DisableRegistryTools -Type DWord -Value 1

# Crea GPO_Seguridad_Equipos y la vincula a la OU Equipos
New-GPO -Name "GPO_Seguridad_Equipos" | New-GPLink -Target $ouEq

# Define la variable con la clave del Registro que usan los tres comandos siguientes
$k = "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System"

# Título del aviso de inicio de sesión
Set-GPRegistryValue -Name "GPO_Seguridad_Equipos" -Key $k -ValueName legalnoticecaption -Type String -Value "Aviso - IU Digital"

# Texto del aviso de inicio de sesión
Set-GPRegistryValue -Name "GPO_Seguridad_Equipos" -Key $k -ValueName legalnoticetext -Type String -Value "Uso exclusivo para actividades académicas."

# No mostrar el último usuario que inició sesión
Set-GPRegistryValue -Name "GPO_Seguridad_Equipos" -Key $k -ValueName DontDisplayLastUserName -Type DWord -Value 1
```

<span class="cmd">8.2.1.2</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba la directiva de contraseñas y los vínculos de las GPO.

```powershell
# Directiva de contraseñas del dominio
Get-ADDefaultDomainPasswordPolicy

# GPO vinculadas a la OU Usuarios
(Get-GPInheritance -Target "OU=Usuarios,OU=IUDigital,DC=iudigital,DC=local").GpoLinks | Select-Object DisplayName, Enabled

# GPO vinculadas a la OU Equipos
(Get-GPInheritance -Target "OU=Equipos,OU=IUDigital,DC=iudigital,DC=local").GpoLinks | Select-Object DisplayName, Enabled
```

#### 8.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">8.2.2.1</span> **Herramientas → Administración de directivas de grupo** → `iudigital.local → IUDigital`: bajo **Usuarios** aparece vinculada `GPO_Restricciones_Usuarios`, y bajo **Equipos**, `GPO_Seguridad_Equipos` (figura 11).

![Abrir Administración de directivas de grupo desde Herramientas](https://cdn.sanity.io/images/r09655ln/production/69299f12cf6ecb4e21f894cd9d53811c9d5a3cbc-1024x441.jpg)

*Figura 10. **Tools → Group Policy Management** (*Herramientas → Administración de directivas de grupo*). Fuente: [Netwrix — Group Policy Management](https://netwrix.com/en/resources/blog/group-policy-management/). Interfaz en inglés.*

![Consola Administración de directivas de grupo con GPO vinculadas a OU](https://learn.microsoft.com/en-us/training/wwl-windows-server/implement-group-policy-objects/media/m8-group-policy-1-3047661e.png)

*Figura 11. Consola **Group Policy Management** (*Administración de directivas de grupo*): en el árbol, cada OU muestra debajo las GPO vinculadas (aquí *Marketing → Marketing Department Desktop Settings*), y la pestaña **Linked Group Policy Objects** (*Objetos de directiva de grupo vinculados*) lista las del dominio. En tu consola, `GPO_Restricciones_Usuarios` debe colgar de `Usuarios` y `GPO_Seguridad_Equipos`, de `Equipos`. Fuente: [Microsoft Learn — Define GPOs](https://learn.microsoft.com/en-us/training/modules/implement-group-policy-objects/2-define-gpos). Interfaz en inglés.*

- <span class="paso">8.2.2.2</span> Selecciona `GPO_Restricciones_Usuarios` → pestaña **Configuración** → **Mostrar todo**: en *Configuración de usuario → Directivas → Plantillas administrativas* aparecen las tres directivas como **Habilitada**.
- <span class="paso">8.2.2.3</span> Selecciona `GPO_Seguridad_Equipos` → pestaña **Configuración**: los tres valores (`legalnoticecaption`, `legalnoticetext` y `DontDisplayLastUserName`) aparecen bajo *Configuración adicional del Registro*, porque PowerShell los guarda como valores del Registro. En el cliente tienen el mismo efecto que configurarlos en *Opciones de seguridad*.
- <span class="paso">8.2.2.4</span> Selecciona `Default Domain Policy` → pestaña **Configuración**: en *Directivas de cuenta* aparecen los valores de contraseña y de bloqueo de la tabla 8.1.

### 8.3 Mapeo de unidades

#### 8.3.1 — PowerShell (como administrador): crear y vincular la GPO

<span class="cmd">8.3.1.1</span> Crea `GPO_Mapeo_Unidades` y la vincula a la OU `Usuarios`.

```powershell
New-GPO -Name "GPO_Mapeo_Unidades" | New-GPLink -Target "OU=Usuarios,OU=IUDigital,DC=iudigital,DC=local"
```

#### 8.3.2 — Solo interfaz gráfica: asignar las unidades

Las asignaciones de unidades son *preferencias* de directiva de grupo y no tienen cmdlet; se configuran en el editor.

- <span class="paso">8.3.2.1</span> En **Administración de directivas de grupo**, clic derecho en `GPO_Mapeo_Unidades` → **Editar**.
- <span class="paso">8.3.2.2</span> Ve a *Configuración de usuario → Preferencias → Configuración de Windows → Asignaciones de unidades* → **Nuevo → Unidad asignada**.
- <span class="paso">8.3.2.3</span> Acción **Actualizar**, ubicación `\\SRV-DC01\Publico`, letra **S:** → **Aceptar**.
- <span class="paso">8.3.2.4</span> Repite con `\\SRV-DC01\Docentes` y la letra **T:**. En la pestaña **Común** marca **Destino de nivel de elemento** → **Grupo de seguridad** → `IUDIGITAL\GG_Docentes`, para que solo los docentes reciban `T:`.
- <span class="paso">8.3.2.5</span> Repite con `\\SRV-DC01\Estudiantes` y la letra **U:**. En **Destino de nivel de elemento** agrega dos elementos **Grupo de seguridad**: `IUDIGITAL\GG_Estudiantes` y `IUDIGITAL\GG_Docentes`. Selecciona el segundo y, en **Opciones de elemento**, cámbialo a **O**, para que `U:` la reciba cualquiera de los dos grupos.

#### 8.3.3 — Comprobar en la interfaz gráfica

- <span class="paso">8.3.3.1</span> Selecciona `GPO_Mapeo_Unidades` → pestaña **Configuración** → **Mostrar todo**: en *Configuración de usuario → Preferencias → Asignaciones de unidades* aparecen `S:`, `T:` y `U:`, con el destino de nivel de elemento de `T:` y `U:`.
- <span class="paso">8.3.3.2</span> La prueba en el cliente está en las secciones 10.5 y 10.6.

---

## 9. Unir el cliente Windows 10 al dominio

El cliente solo encuentra el dominio si su DNS es `192.168.10.10`. Si recibe la IP del DHCP de la sección 5, ya lo tiene.

### 9.1 Comprobar la red (en el cliente)

Inicia sesión en el cliente con su administrador local y abre PowerShell como administrador.

<span class="nota-orden">Orden de ejecución</span> Pega **9.1.1** (IP por DHCP) **o** **9.1.2** (IP a mano), y espera a que termine; después, la comprobación **9.1.3**.

<span class="cmd">9.1.1</span><span class="orden sec">secuencial</span><span class="pega">pegar todo junto</span> Con la IP por DHCP: renueva la IP del cliente y muestra la configuración: IP `192.168.10.1xx` y DNS `192.168.10.10`.

```powershell
# Libera la IP actual
ipconfig /release

# Pide una IP nueva al DHCP del servidor
ipconfig /renew

# Muestra la configuración: IP 192.168.10.1xx y DNS 192.168.10.10
ipconfig /all
```

<span class="cmd">9.1.2</span><span class="orden sec">secuencial</span> Solo si **no** usas DHCP y configuras la IP del cliente a mano: además de la IP, el DNS debe ser el DC. Cambia *Ethernet* si tu adaptador tiene otro nombre (`Get-NetAdapter`).

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10
```

<span class="cmd">9.1.3</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Comprueba que el servidor responde y que el cliente resuelve el dominio.

```powershell
# Comprueba que el servidor responde
ping 192.168.10.10

# Comprueba que el cliente resuelve el dominio
nslookup iudigital.local
```

### 9.2 Unir el cliente al dominio

#### 9.2.1 — PowerShell (en el cliente, como administrador)

<span class="cmd">9.2.1.1</span> Une el equipo al dominio con el nombre `CLI-W10-01` y lo **reinicia**.

```powershell
Add-Computer -DomainName "iudigital.local" -NewName "CLI-W10-01" -Credential IUDIGITAL\Administrador -Restart
```

Después del reinicio:

<span class="cmd">9.2.1.2</span> Nombre del equipo, dominio y pertenencia al dominio.

```powershell
(Get-CimInstance Win32_ComputerSystem) | Select-Object Name, Domain, PartOfDomain   # CLI-W10-01, iudigital.local, True
```

#### 9.2.2 — Comprobar en la interfaz gráfica

- <span class="paso">9.2.2.1</span> **Win + R** → `sysdm.cpl` → **Aceptar**.

![Propiedades del sistema con sysdm.cpl](https://cdn.techloris.com/app/uploads/2022/10/sysdm-cpl.png)

*Figura 12. **Win + R** abre *Ejecutar*; escribe `sysdm.cpl` → **Aceptar** para abrir *Propiedades del sistema*. Fuente: [TechLoris](https://techloris.com/join-domains-in-windows-10/).*

- <span class="paso">9.2.2.2</span> Pestaña **Nombre de equipo**: *Nombre completo del equipo* = `CLI-W10-01.iudigital.local` y *Dominio* = `iudigital.local`.

![Pestaña Nombre de equipo de Propiedades del sistema con el dominio](https://adamtheautomator.com/wp-content/uploads/2021/04/Untitled-2021-04-07T103248.287.png)

*Figura 13. Pestaña **Computer Name** (*Nombre de equipo*) de *Propiedades del sistema* en un Windows 10 ya unido: *Full computer name* = `Win10Demo.HomeLab.Local` y *Domain* = `HomeLab.Local`. En tu equipo debe decir `CLI-W10-01.iudigital.local` y **Dominio: iudigital.local**. El aviso *Changes will take effect after you restart this computer* solo aparece antes de reiniciar; como `Add-Computer -Restart` reinicia solo, tú no lo verás. Fuente: [ATA Learning — Add a Computer to a Domain](https://adamtheautomator.com/add-computer-to-domain/). Interfaz en inglés.*

### 9.3 Mover el equipo a su OU (en el servidor)

Tras unirse, el equipo aparece en el contenedor **Computers**.

#### 9.3.1 — PowerShell (en el servidor)

<span class="nota-orden">Orden de ejecución</span> Pega primero **9.3.1.1** y espera a que termine; después, la comprobación **9.3.1.2**.

<span class="cmd">9.3.1.1</span><span class="orden sec">secuencial</span> Mueve el equipo a la OU `Equipos`.

```powershell
Get-ADComputer CLI-W10-01 | Move-ADObject -TargetPath "OU=Equipos,OU=IUDigital,DC=iudigital,DC=local"
```

<span class="cmd">9.3.1.2</span><span class="orden ind">comprobación</span> Comprueba su nueva ubicación.

```powershell
Get-ADComputer CLI-W10-01 | Select-Object DistinguishedName   # ...OU=Equipos,OU=IUDigital,...
```

#### 9.3.2 — Comprobar en la interfaz gráfica

- <span class="paso">9.3.2.1</span> **Usuarios y equipos de Active Directory** → `IUDigital → Equipos`: aparece `CLI-W10-01`, y ya no está en **Computers**.

![Equipo dentro de una OU en Usuarios y equipos de Active Directory](https://techdirectarchive.com/wp-content/uploads/2020/04/screenshot-2020-04-06-at-14.45.41.png)

*Figura 14. **Active Directory Users and Computers** (*Usuarios y equipos de Active Directory*) con la OU *EmployeePC* seleccionada: el equipo movido aparece dentro de la OU y ya no en *Computers*. En tu consola selecciona `IUDigital → Equipos`: debe aparecer `CLI-W10-01`. Fuente: [TechDirectArchive — Move a Computer Object from One Container or OU to Another](https://techdirectarchive.com/2020/04/06/how-to-move-a-computer-object-from-one-container-ou-to-another/). Interfaz en inglés.*

### 9.4 Iniciar sesión con un usuario del dominio (solo interfaz gráfica)

Inicia sesión en el cliente con **Otro usuario** → `IUDIGITAL\bienestaru` y reinícialo una vez más para que reciba las GPO de equipo.

---

## 10. Verificación final

La práctica queda aprobada cuando se cumplen tres cosas: AD DS responde con sus servicios en ejecución, `nslookup` resuelve en ambos sentidos y encuentra los registros SRV del DC, y el cliente reporta `iudigital.local` como su dominio con un usuario del dominio conectado.

> Ejecuta las pruebas del cliente en PowerShell como `IUDIGITAL\Administrador`, salvo las de la sección 10.5, que se hacen con la sesión de `bienestaru`.

### 10.1 Active Directory (en el servidor)

<span class="cmd">10.1.1</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Datos del dominio y del DC, servicios, recursos compartidos, OU del cliente y permisos de SYSVOL.

```powershell
# Datos del dominio y DC que tiene el rol de emulador PDC
Get-ADDomain | Select DNSRoot, NetBIOSName, DomainMode, PDCEmulator

# Controlador de dominio: nombre, IP y catálogo global
Get-ADDomainController | Select HostName, IPv4Address, IsGlobalCatalog

# Servicios de AD DS, DNS y DHCP
Get-Service ADWS, KDC, Netlogon, NTDS, DNS, DHCPServer | Select Name, Status

# Recursos compartidos del servidor
net share   # NETLOGON, SYSVOL, Publico, Docentes, Estudiantes

# OU del equipo cliente
Get-ADComputer CLI-W10-01 | Select Name, DistinguishedName

# Permisos de SYSVOL
icacls C:\Windows\SYSVOL   # Usuarios autentificados solo (RX); no aparecen GG_Estudiantes ni GG_Docentes
```

SYSVOL guarda las GPO y los scripts de inicio de sesión: todos los usuarios y equipos del dominio deben poder **leerlo**, así que no se restringe a `GG_Estudiantes` ni a `GG_Docentes`. Lo que se comprueba es que ningún grupo de usuarios tenga más que lectura y ejecución.

### 10.2 DNS con nslookup (en el cliente)

| Comando | Qué prueba | Resultado esperado |
| --- | --- | --- |
| `nslookup srv-dc01.iudigital.local` | Registro A | 192.168.10.10 |
| `nslookup 192.168.10.10` | Registro PTR (zona inversa) | srv-dc01.iudigital.local |
| `nslookup iudigital.local` | Registro del dominio | 192.168.10.10 |
| `nslookup -type=SRV _ldap._tcp.dc._msdcs.iudigital.local` | Localización del DC | srv-dc01.iudigital.local, puerto 389 |
| `nslookup -type=SRV _kerberos._tcp.iudigital.local` | Kerberos | puerto 88 |
| `nslookup -type=SRV _gc._tcp.iudigital.local` | Catálogo global | puerto 3268 |
| `nslookup cli-w10-01.iudigital.local` | Registro dinámico del cliente | IP del cliente |
| `nslookup www.google.com` | Reenviadores | respuesta *no autoritativa* |

Si la primera línea de `nslookup` dice *Servidor: UnKnown*, falta la zona inversa o el PTR.

![nslookup resolviendo IP a nombre y nombre a IP](https://markg001.github.io/SysOps-Ledger/assets/img/posts/dns-setup/nslookup.png)

*Figura 15. Resultado correcto en Windows Server 2022: arriba, la zona inversa con su PTR; abajo, `nslookup` muestra el nombre del DC en *Default Server* y resuelve la IP → nombre y el nombre → IP. Fuente: [SysOps Ledger (Mark G.)](https://markg001.github.io/SysOps-Ledger/posts/AD-DNS-SETUP/).*

![nslookup con el servidor como UnKnown](https://activedirectorypro.com/images/posts/configure-dns-reverse-lookup-zones-ptr-records/ptr-record-verify.jpg)

*Figura 16. Resultado cuando existe el PTR del equipo consultado pero **no** el del servidor DNS (captura recortada por su autor, con los nombres tachados): el nombre se resuelve, pero la línea *Server* dice **UnKnown**. Si ves esto, crea el PTR de `srv-dc01` (sección 4.3). Fuente: [ActiveDirectoryPro](https://activedirectorypro.com/configure-dns-reverse-lookup-zones-ptr-records/).*

### 10.3 DHCP

- En el cliente, `ipconfig /all` debe mostrar *DHCP habilitado: Sí*, *Servidor DHCP: 192.168.10.10* y una IP dentro del ámbito.
- En el servidor, `Get-DhcpServerv4Lease -ScopeId 192.168.10.0` debe listar la concesión de `CLI-W10-01`.

![Ámbito activo en la consola DHCP](https://i0.wp.com/techdirectarchive.com/wp-content/uploads/2024/05/Scope-Active.jpg)

*Figura 17. Consola DHCP con el ámbito en estado **Activo**. En tu servidor, abre *Concesiones de direcciones* dentro del ámbito para ver la de `CLI-W10-01`. Fuente: [TechDirectArchive](https://techdirectarchive.com/2024/05/04/comprehensive-guide-to-install-dhcp-server-on-windows-server/).*

### 10.4 Cliente conectado al dominio

<span class="cmd">10.4.1</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> Dominio, usuario, DC que validó el inicio de sesión, canal seguro y pertenencia al dominio.

```powershell
# Dominio al que pertenece el equipo (en Windows en inglés, cambia "Dominio" por "Domain")
systeminfo | findstr /B /C:"Dominio"

# Usuario con el que iniciaste sesión
whoami

# DC que validó el inicio de sesión
$env:LOGONSERVER

# DC que el cliente encuentra para el dominio
nltest /dsgetdc:iudigital.local

# Estado del canal seguro con el dominio
nltest /sc_verify:iudigital.local

# Canal seguro con el dominio: debe devolver True
Test-ComputerSecureChannel -Verbose

# Nombre del equipo, dominio y pertenencia al dominio
(Get-CimInstance Win32_ComputerSystem) | Select Name, Domain, PartOfDomain
```

Resultado esperado: *Dominio: iudigital.local*, `$env:LOGONSERVER` = `\\SRV-DC01`, `Test-ComputerSecureChannel` = **True** y `PartOfDomain` = **True**.

### 10.5 GPO aplicadas

**Dónde:** en el cliente, con la sesión de `IUDIGITAL\bienestaru` abierta. Las GPO de usuario se comprueban sobre él, porque `Administrador` no está en la OU `Usuarios`.

<span class="nota-orden">Orden de ejecución</span> Pega primero **10.5.1** y espera a que termine; después, la comprobación **10.5.2**.

<span class="cmd">10.5.1</span><span class="orden sec">secuencial</span> En PowerShell **normal** (sin ejecutar como administrador; `cmd` está bloqueado para `bienestaru`). **Pégalo solo**: aplica de inmediato las GPO. Si pregunta si quieres cerrar la sesión o reiniciar, responde **N**; si lo pegaras junto con otros comandos, la línea siguiente se tomaría como respuesta.

```powershell
gpupdate /force
```

<span class="cmd">10.5.2</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> En PowerShell **como administrador**: clic derecho → *Ejecutar como administrador* y, cuando Windows pida credenciales, usa `IUDIGITAL\Administrador`. Sin elevar, `gpresult` solo muestra la parte del usuario; elevado con otra cuenta, mostraría el usuario Administrador, por eso se indica `/user IUDIGITAL\bienestaru`.

```powershell
# Resumen de las GPO aplicadas al equipo y a bienestaru
gpresult /r /user IUDIGITAL\bienestaru

# Informe completo en HTML (/f lo sobrescribe si ya existe)
gpresult /h C:\Users\Public\reporte_gpo.html /user IUDIGITAL\bienestaru /f
```

`gpresult` debe listar `GPO_Seguridad_Equipos` y `Default Domain Policy` en la parte del equipo, y `GPO_Restricciones_Usuarios` y `GPO_Mapeo_Unidades` en la del usuario. Con `bienestaru`, el Panel de control y `cmd` aparecen bloqueados y las unidades `S:` y `U:` están visibles.

### 10.6 Recursos compartidos

- `bienestaru`: tiene las unidades `S:` y `U:`; puede escribir en `Estudiantes` (`U:`), solo leer en `Publico` (`S:`), y no ve `Docentes` o recibe *Acceso denegado*.
- `jdocente`: tiene las unidades `S:`, `T:` y `U:`, y puede escribir en las tres carpetas.
- Ninguno de los dos puede borrar las carpetas `Publico`, `Docentes` ni `Estudiantes` (compruébalo con el paso 7.2.2.6).
- Prueba de **CREATOR OWNER**: con `bienestaru` crea un archivo en `U:`; con `jdocente` crea otro. Vuelve a `bienestaru` e intenta borrar el de `jdocente`: debe salir *Acceso denegado*, pero su propio archivo sí lo puede borrar.

### 10.7 Evidencias para el informe (capturas propias)

Las capturas del informe deben ser **de tu laboratorio**. Para cada evidencia se indica dónde tomarla, qué debe verse y con qué figura o sección de esta guía puedes compararla.

> **Sobre los ejemplos de texto.** Los bloques de salida de esta sección son **ejemplos orientativos escritos con los valores del laboratorio**; no son capturas. En tu equipo cambiarán la IP exacta del cliente, las fechas y las direcciones MAC, y algunos rótulos pueden variar según el idioma y la versión de Windows. Lo que debe coincidir son los **valores** resaltados en cada caso.

**Lista de control**

- [ ] [E1 (10.7.1)](#1071--e1--ipconfig-all-del-servidor-y-del-cliente). `ipconfig /all` del servidor (IP fija) y del cliente (IP por DHCP)
- [ ] [E2 (10.7.2)](#1072--e2--consola-dns-zona-directa-zona-inversa-y-registros-srv). Consola DNS con la zona directa, la zona inversa y los registros SRV
- [ ] [E3 (10.7.3)](#1073--e3--consola-dhcp-ámbito-opciones-y-concesión). Consola DHCP con el ámbito activo, las opciones 003/006/015 y la concesión del cliente
- [ ] [E4 (10.7.4)](#1074--e4--get-addomain-y-servicios-de-ad). `Get-ADDomain` y servicios de AD
- [ ] [E5 (10.7.5)](#1075--e5--pruebas-de-nslookup). Las pruebas de `nslookup` de la tabla 10.2
- [ ] [E6 (10.7.6)](#1076--e6--usuarios-y-equipos-de-active-directory). Usuarios y equipos de AD con las OU, los usuarios, los grupos y `CLI-W10-01` en Equipos
- [ ] [E7 (10.7.7)](#1077--e7--unión-al-dominio-y-canal-seguro). Dominio en `sysdm.cpl` y `Test-ComputerSecureChannel`
- [ ] [E8 (10.7.8)](#1078--e8--gpresult-r-y-bloqueos-de-las-gpo). `gpresult /r` y el bloqueo del Panel de control y de `cmd`
- [ ] [E9 (10.7.9)](#1079--e9--unidades-asignadas-y-acceso-denegado). Unidades `S:` y `U:` asignadas y acceso denegado a `Docentes`

#### 10.7.1 — E1 · `ipconfig /all` del servidor y del cliente

**Dónde:** en cada VM, en `cmd` o PowerShell. En el cliente hazlo con el administrador local o con `jdocente`, porque a `bienestaru` se le bloquea `cmd` (puede usar PowerShell).

Servidor (fragmento):

```text
   Nombre de host. . . . . . . . . . . . . . : SRV-DC01
   Sufijo DNS principal  . . . . . . . . . . : iudigital.local

Adaptador de Ethernet Ethernet:

   DHCP habilitado . . . . . . . . . . . . . : no
   Dirección IPv4. . . . . . . . . . . . . . : 192.168.10.10(Preferido)
   Máscara de subred . . . . . . . . . . . . : 255.255.255.0
   Puerta de enlace predeterminada . . . . . : 192.168.10.1
   Servidores DNS. . . . . . . . . . . . . . : 127.0.0.1
```

Cliente (fragmento):

```text
   Nombre de host. . . . . . . . . . . . . . : CLI-W10-01
   Sufijo DNS principal  . . . . . . . . . . : iudigital.local

Adaptador de Ethernet Ethernet:

   Sufijo DNS específico para la conexión. . : iudigital.local
   DHCP habilitado . . . . . . . . . . . . . : sí
   Dirección IPv4. . . . . . . . . . . . . . : 192.168.10.111(Preferido)
   Máscara de subred . . . . . . . . . . . . : 255.255.255.0
   Puerta de enlace predeterminada . . . . . : 192.168.10.1
   Servidor DHCP . . . . . . . . . . . . . . : 192.168.10.10
   Servidores DNS. . . . . . . . . . . . . . : 192.168.10.10
```

**Debe coincidir:** en el servidor, *DHCP habilitado: no* y la IP `192.168.10.10`; en el servidor también puede aparecer `::1` como DNS IPv6. En el cliente, *DHCP habilitado: sí*, *Servidor DHCP* y *Servidores DNS* = `192.168.10.10`, y una IP entre `.111` y `.200` (de `.100` a `.110` están excluidas).

**No debe salir:** una dirección `169.254.x.x` (eso significa que no llegó al DHCP) ni un DNS distinto de `192.168.10.10`.

**Compara con:** sección 2.1, sección 9.1 y sección 10.3.

#### 10.7.2 — E2 · Consola DNS: zona directa, zona inversa y registros SRV

**Dónde:** en el servidor, **Herramientas → DNS** → expande `SRV-DC01`.

**Debe verse:**

- **Zonas de búsqueda directa → `iudigital.local`**: registros *Inicio de autoridad (SOA)*, *Servidor de nombres (NS)*, *Host (A)* de `srv-dc01` → `192.168.10.10` y de `cli-w10-01` → IP del cliente, y las carpetas `_msdcs`, `_sites`, `_tcp` y `_udp`. También aparece la zona `_msdcs.iudigital.local`.
- **Dentro de `_tcp`**: registros *Ubicación del servicio (SRV)* `_gc`, `_kerberos`, `_kpasswd` y `_ldap`, todos apuntando a `srv-dc01.iudigital.local`.
- **Zonas de búsqueda inversa → `10.168.192.in-addr.arpa`**: el *Puntero (PTR)* `192.168.10.10` → `srv-dc01.iudigital.local` y, si el cliente se registró, el suyo.

![Zona directa con registros de recursos y zona inversa en la consola DNS](https://learn.microsoft.com/en-us/training/wwl-windows-server/implement-windows-server-dns/media/m12-dns-1.png)

*Figura 18. Referencia de cómo se ve la consola DNS: la zona `_msdcs.Contoso.com`, la zona directa `Contoso.com` con sus carpetas `_msdcs`, `_sites`, `_tcp` y `_udp` y sus registros *SOA*, *NS* y *Host (A)*, y la zona inversa `10.16.172.in-addr.arpa`. En tu captura la zona será `iudigital.local` y la inversa `10.168.192.in-addr.arpa`. Fuente: [Microsoft Learn — Work with DNS zones and records](https://learn.microsoft.com/en-us/training/modules/implement-windows-server-dns/3-work-dns-zones-records). Interfaz en inglés.*

**Compara con:** figura 15 (parte superior: zona inversa con su PTR) y secciones 4.1 a 4.3. Para la carpeta `_tcp`, la evidencia E5 confirma los mismos registros desde el cliente.

#### 10.7.3 — E3 · Consola DHCP: ámbito, opciones y concesión

**Dónde:** en el servidor, **Herramientas → DHCP** → `srv-dc01.iudigital.local` → **IPv4**.

**Debe verse:**

- **IPv4** con una marca verde: el servidor está **autorizado** en AD.
- El ámbito `[192.168.10.0]` en estado **Activo**.
- **Grupo de direcciones**: el intervalo `192.168.10.100` – `192.168.10.200` y la exclusión `.100` – `.110`.
- **Concesiones de direcciones**: `CLI-W10-01.iudigital.local` con su IP.
- **Opciones de ámbito**: `003 Enrutador` = `192.168.10.1`, `006 Servidores DNS` = `192.168.10.10` y `015 Nombre de dominio DNS` = `iudigital.local`.

Si prefieres una captura de PowerShell:

```text
PS C:\> Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0

OptionId   Name            Type        Value
--------   ----            ----        -----
51         Lease           DWord       {691200}
15         DNS Domain Name String      {iudigital.local}
6          DNS Servers     IPv4Address {192.168.10.10}
3          Router          IPv4Address {192.168.10.1}

PS C:\> Get-DhcpServerv4Lease -ScopeId 192.168.10.0 | Select IPAddress, HostName, AddressState

IPAddress       HostName                   AddressState
---------       --------                   ------------
192.168.10.111  CLI-W10-01.iudigital.local Active
```

*La opción 51 es la duración de la concesión: 691 200 segundos = 8 días, el valor predeterminado del asistente.*

**Compara con:** figura 5 (nodo **IPv4** autorizado), figura 17 (ámbito *Activo*) y sección 5.2. Si **IPv4** muestra una flecha roja, el servidor no está autorizado: vuelve al paso 5.1.

#### 10.7.4 — E4 · `Get-ADDomain` y servicios de AD

**Dónde:** en el servidor, PowerShell como administrador.

```text
PS C:\> Get-ADDomain | Select DNSRoot, NetBIOSName, DomainMode, PDCEmulator

DNSRoot         NetBIOSName DomainMode        PDCEmulator
-------         ----------- ----------        -----------
iudigital.local IUDIGITAL   Windows2016Domain SRV-DC01.iudigital.local

PS C:\> Get-Service NTDS, DNS, Netlogon, ADWS, KDC | Select Name, Status

Name       Status
----       ------
ADWS      Running
DNS       Running
KDC       Running
Netlogon  Running
NTDS      Running
```

**Debe coincidir:** `Get-ADDomain` muestra `iudigital.local`, `IUDIGITAL`, el nivel funcional `Windows2016Domain` (el más alto en Windows Server 2022) y `SRV-DC01` como maestro de operaciones. Los cinco servicios deben estar en **Running**; si alguno está detenido, revisa la sección 11.

**Compara con:** sección 10.1.

#### 10.7.5 — E5 · Pruebas de `nslookup`

**Dónde:** en el cliente, `cmd` o PowerShell. Toma una captura con las ocho consultas de la tabla 10.2.

```text
C:\> nslookup srv-dc01.iudigital.local
Servidor:  srv-dc01.iudigital.local
Address:  192.168.10.10

Nombre:  srv-dc01.iudigital.local
Address:  192.168.10.10

C:\> nslookup 192.168.10.10
Servidor:  srv-dc01.iudigital.local
Address:  192.168.10.10

Nombre:  srv-dc01.iudigital.local
Address:  192.168.10.10

C:\> nslookup -type=SRV _ldap._tcp.dc._msdcs.iudigital.local
Servidor:  srv-dc01.iudigital.local
Address:  192.168.10.10

_ldap._tcp.dc._msdcs.iudigital.local    SRV service location:
          priority       = 0
          weight         = 100
          port           = 389
          svr hostname   = srv-dc01.iudigital.local
srv-dc01.iudigital.local        internet address = 192.168.10.10

C:\> nslookup www.google.com
Servidor:  srv-dc01.iudigital.local
Address:  192.168.10.10

Respuesta no autoritativa:
Nombre:  www.google.com
Addresses:  ...
```

**Debe coincidir:** la línea *Servidor* dice `srv-dc01.iudigital.local`, nunca *UnKnown*; las consultas SRV devuelven los puertos 389, 88 y 3268. `www.google.com` solo responde si el servidor tiene salida a Internet y reenviadores (sección 4.4); en una red interna aislada es normal que falle.

**Compara con:** figura 15 (resultado correcto) y figura 16 (*UnKnown*: falta el PTR del servidor).

#### 10.7.6 — E6 · Usuarios y equipos de Active Directory

**Dónde:** en el servidor, **Herramientas → Usuarios y equipos de Active Directory** → expande `iudigital.local → IUDigital`. Toma una captura por OU.

**Debe verse:**

- `IUDigital → Usuarios`: `Bienestar Universitario` (`bienestaru`) y `jdocente`.
- `IUDigital → Grupos`: `GG_Docentes` y `GG_Estudiantes`, de tipo *Grupo de seguridad – Global*. En la pestaña **Miembros** de cada grupo, su usuario.
- `IUDigital → Equipos`: `CLI-W10-01`. Si sigue en el contenedor **Computers**, no hiciste el paso 9.3.

![Consola Usuarios y equipos de Active Directory con el contenedor Computers seleccionado](https://learn.microsoft.com/en-us/training/wwl-azure/create-manage-active-directory-objects/media/objects.png)

*Figura 19. Referencia de la consola **Active Directory Users and Computers** (*Usuarios y equipos de Active Directory*) con el contenedor *Computers* del dominio `Contoso.com` seleccionado y, en el árbol, las OU del dominio. En tu captura debe verse la OU `IUDigital` con sus tres OU hijas. Fuente: [Microsoft Learn — Define organizational units](https://learn.microsoft.com/en-us/training/modules/create-manage-active-directory-objects/3-define-organizational-units). Interfaz en inglés.*

Si prefieres una captura de PowerShell:

<span class="cmd">10.7.6.1</span><span class="orden ind">comprobación</span><span class="pega">pegar todo junto</span> OU, usuarios, miembros de `GG_Estudiantes` y OU del equipo cliente.

```powershell
# OU de IUDigital
Get-ADOrganizationalUnit -Filter * -SearchBase "OU=IUDigital,DC=iudigital,DC=local" | Select Name

# Usuarios de la OU Usuarios
Get-ADUser -Filter * -SearchBase "OU=Usuarios,OU=IUDigital,DC=iudigital,DC=local" | Select SamAccountName

# Miembros de GG_Estudiantes
Get-ADGroupMember GG_Estudiantes | Select SamAccountName

# OU del equipo cliente
Get-ADComputer CLI-W10-01 | Select DistinguishedName   # ...OU=Equipos,OU=IUDigital,...
```

**Compara con:** sección 6.2 y figura 14 (equipo dentro de una OU, como debe quedar `CLI-W10-01` en `Equipos` tras el paso 9.3).

#### 10.7.7 — E7 · Unión al dominio y canal seguro

**Dónde:** en el cliente.

- <span class="paso">10.7.7.1</span> **Win + R → `sysdm.cpl`**, pestaña **Nombre de equipo**: *Dominio* = `iudigital.local` (figura 13). Al unir el equipo con `Add-Computer` no aparece el mensaje *Bienvenido al dominio*; esta captura lo reemplaza.
- <span class="paso">10.7.7.2</span> En PowerShell como administrador:

```text
PS C:\> Test-ComputerSecureChannel
True

PS C:\> (Get-CimInstance Win32_ComputerSystem) | Select Name, Domain, PartOfDomain

Name       Domain          PartOfDomain
----       ------          ------------
CLI-W10-01 iudigital.local         True

PS C:\> echo $env:LOGONSERVER
\\SRV-DC01
```

**Debe coincidir:** `True` en las dos pruebas y `\\SRV-DC01` como servidor de inicio de sesión (este último, con la sesión de un usuario del dominio). Con `-Verbose`, la línea adicional indica que el canal seguro con `iudigital.local` funciona correctamente. Si sale `False`, revisa la sección 11.

![Test-ComputerSecureChannel devolviendo False](https://woshub.com/wp-content/uploads/2021/03/test-computersecurechannel-the-secure-channel-bet.png.webp)

*Figura 20. Resultado **incorrecto**, para que lo reconozcas: `Test-ComputerSecureChannel -Verbose` devuelve **False** y la línea *VERBOSE* dice que el canal seguro con el dominio está roto (*is broken*). En tu laboratorio debe salir **True**. Fuente: [Windows OS Hub — Repairing the Domain Trust Relationship](https://woshub.com/repair-trust-relationship-workstation-with-ad-domain/). Interfaz en inglés.*

**Compara con:** sección 9.2.2 (figuras 12 y 13) y figura 14.

#### 10.7.8 — E8 · `gpresult /r` y bloqueos de las GPO

**Dónde:** en el cliente, con la sesión de `IUDIGITAL\bienestaru`, siguiendo la sección 10.5: `gpupdate /force` en PowerShell normal (no en `cmd`, que está bloqueado) y `gpresult /r /user IUDIGITAL\bienestaru` en PowerShell como administrador, con las credenciales de `IUDIGITAL\Administrador`.

**Debe verse en `gpresult`:**

- En la parte del **equipo**, bajo *Objetos de directiva de grupo aplicados*: `GPO_Seguridad_Equipos` y `Default Domain Policy`.
- En la parte del **usuario**, bajo *Objetos de directiva de grupo aplicados*: `GPO_Restricciones_Usuarios` y `GPO_Mapeo_Unidades`.
- Es normal que `Default Domain Policy` aparezca en la parte del usuario entre las GPO **no aplicadas** con el motivo *vacío*: solo tiene configuración de equipo.

**Capturas de los bloqueos (con `bienestaru`):**

- **Panel de control:** al abrirlo aparece una ventana **Restricciones** que dice que la operación se canceló por las restricciones del equipo.
- **`cmd`:** se abre la ventana, pero muestra que el administrador deshabilitó el símbolo del sistema y pide presionar una tecla para cerrarla.
- **`regedit`:** aparece un aviso de que el administrador deshabilitó la edición del Registro.
- Al cerrar sesión y volver a entrar, el **aviso legal** de `GPO_Seguridad_Equipos` aparece antes de la pantalla de inicio de sesión.

![gpresult /r: GPO aplicadas al equipo](https://activedirectorypro.com/images/posts/gpresult-tool/gpresult-command-example-1a.png)

*Figura 21. Parte **COMPUTER SETTINGS** (*Configuración del equipo*) de `gpresult /r` en un cliente de otro dominio: el recuadro marca **Applied Group Policy Objects** (*Objetos de directiva de grupo aplicados*). Debajo están las GPO no aplicadas, con el motivo *Not Applied (Empty)*. En tu captura deben aparecer `GPO_Seguridad_Equipos` y `Default Domain Policy`. Fuente: [ActiveDirectoryPro — GPResult: How to Check Applied GPOs](https://activedirectorypro.com/gpresult-tool/). Interfaz en inglés.*

![gpresult /r: GPO aplicadas al usuario](https://activedirectorypro.com/images/posts/gpresult-tool/gpresult-command-example-1b.png)

*Figura 22. Parte **USER SETTINGS** (*Configuración del usuario*): GPO aplicadas al usuario y, debajo, las filtradas. En tu captura deben aparecer `GPO_Restricciones_Usuarios` y `GPO_Mapeo_Unidades`. Fuente: [ActiveDirectoryPro](https://activedirectorypro.com/gpresult-tool/).*

![cmd bloqueado por directiva](https://woshub.com/wp-content/uploads/2026/09/error-the-command-prompt-has-been-disabled-by-you.png.webp)

*Figura 23. `cmd` bloqueado: **The command prompt has been disabled by your administrator. Press any key to continue…** (*El administrador deshabilitó el símbolo del sistema. Presione una tecla para continuar…*). Fuente: [Windows OS Hub — Re-Enable Command Prompt Blocked by Administrator](https://woshub.com/command-prompt-blocked-administrator-policy/). Interfaz en inglés.*

![Ventana Restricciones](https://woshub.com/wp-content/uploads/2026/09/restrictions-this-operation-has-been-cancelled-due.png.webp)

*Figura 24. Ventana **Restrictions** (*Restricciones*): *This operation has been cancelled due to restrictions in effect on this computer. Please contact your system administrator.* En el tutorial aparece al bloquear `cmd.exe` con otra directiva; es la misma ventana que verás al abrir el Panel de control con `bienestaru`. Fuente: [Windows OS Hub](https://woshub.com/command-prompt-blocked-administrator-policy/). Interfaz en inglés.*

**Compara con:** sección 8.1 (tabla de GPO) y sección 10.5.

#### 10.7.9 — E9 · Unidades asignadas y acceso denegado

**Dónde:** en el cliente, en el **Explorador de archivos → Este equipo**.

**Debe verse:**

- Con `bienestaru`: en *Ubicaciones de red*, las unidades **`Publico (\\SRV-DC01) (S:)`** y **`Estudiantes (\\SRV-DC01) (U:)`**. Al abrir `\\SRV-DC01\Docentes`, un mensaje de que Windows no puede acceder porque **no tienes permiso** y debes contactar al administrador de red.
- Con `jdocente`: las unidades **`S:`**, **`T:`** y **`U:`**, y puede crear archivos en las tres carpetas.

En PowerShell, `net use` debe listar las unidades (con `bienestaru`):

```text
PS C:\> net use
Estado       Local     Remoto                    Red
-------------------------------------------------------------------------------
OK           S:        \\SRV-DC01\Publico        Microsoft Windows Network
OK           U:        \\SRV-DC01\Estudiantes    Microsoft Windows Network
```

Con `jdocente` aparece además `T:` → `\\SRV-DC01\Docentes`.

![Unidad de red asignada por GPO en el Explorador de archivos](https://woshub.com/wp-content/uploads/2021/03/mapped-network-drive-on-windows-10-file-explorer.png.webp)

*Figura 25. Windows 10, **This PC → Network locations** (*Este equipo → Ubicaciones de red*) con una unidad asignada por GPO, en la captura *WorkSpace (M:)*. En la tuya debe verse `Publico (\\SRV-DC01) (S:)`. Fuente: [Windows OS Hub — Mapping Network Drives via Group Policy](https://woshub.com/map-network-drives-shared-folders-gpo/). Interfaz en inglés.*

**Compara con:** sección 7.1 (tabla de permisos), sección 8.3 (mapeo de unidades) y sección 10.6.

---

## 11. Solución de problemas comunes

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| *No se pudo contactar con un controlador de dominio* | El cliente usa otro DNS | Corrige la opción 006 del DHCP o el DNS del cliente; `ipconfig /flushdns` |
| El cliente recibe una IP 169.254.x.x | No llega al DHCP o el servidor no está autorizado | Misma red interna; autoriza el servidor (5.1.1) y activa el ámbito (5.2.1); en el cliente, repite 9.1.1 |
| El cliente recibe una IP de otra red (10.0.2.x) | Responde el DHCP del hipervisor | Pon el adaptador en *Red interna* |
| `nslookup` dice *Servidor: UnKnown* | Falta la zona inversa o el PTR | Secciones 4.2 y 4.3 |
| Los SRV no responden | Netlogon no registró los registros | `ipconfig /registerdns`; `Restart-Service Netlogon`; repite las consultas SRV de la tabla 10.2 |
| No aparece la opción *Dominio* | Windows 10 Home | Usa Pro, Education o Enterprise |
| Error de Kerberos o de hora | Diferencia de reloj mayor de 5 min | Misma zona horaria; en el cliente, `w32tm /resync` |
| La GPO no se aplica | Objeto fuera de la OU, o falta reiniciar | Mueve el objeto a su OU (9.3.1); en el cliente, `gpupdate /force` y reinicia; comprueba con `gpresult` (10.5) |
| *Acceso denegado* inesperado | Gana el permiso más restrictivo, o el token de grupos es viejo | Revisa ambos permisos; cierra y abre sesión |

---

## 12. Fuentes de las capturas

| Fuente | Qué ilustra | Versión e idioma de la interfaz | Figuras |
| --- | --- | --- | --- |
| [LayerStack — Static IP Address on Windows Server 2022](https://www.layerstack.com/resources/tutorials/How-to-configure-the-Static-IP-Address-Network-Card-for-Windows-2022-Cloud-Servers) | Propiedades de Ethernet | Windows Server 2022 · inglés | 2 |
| [Petri — Active Directory Functional Levels Explained](https://petri.com/active-directory-functional-levels/) | Niveles funcionales del dominio y del bosque | Windows Server · inglés | 3 |
| [Microsoft Learn — Implement DNS forwarding](https://learn.microsoft.com/en-us/training/modules/implement-windows-server-dns/5-implement-dns-forwarding) | Reenviadores DNS | Oficial (módulo de formación) · inglés | 4 |
| [Microsoft Learn — Install and configure the DHCP role](https://learn.microsoft.com/en-us/training/modules/deploy-manage-dynamic-host-configuration-protocol/3-install-configure-dynamic-host-configuration-protocol) | Consola DHCP con IPv4 autorizado | Oficial (módulo de formación) · inglés | 5 |
| [Netwrix — NTFS vs. Share Permissions](https://netwrix.com/en/resources/blog/ntfs-vs-share-permissions/) | Pestaña Compartir | Windows Server · inglés | 6 |
| [ActiveDirectoryPro — NTFS vs Share Permissions](https://activedirectorypro.com/ntfs-vs-share-permissions/) | Permisos del recurso y pestaña Seguridad | Windows Server · inglés | 7, 8 |
| [Windows OS Hub — Enable Access-based Enumeration (ABE)](https://woshub.com/enable-access-based-enumeration-in-windows-server/) | Enumeración basada en el acceso en el Administrador del servidor | Windows Server · inglés | 9 |
| [Netwrix — Group Policy Management](https://netwrix.com/en/resources/blog/group-policy-management/) | Abrir la consola de GPO | Windows Server · inglés | 10 |
| [Microsoft Learn — Define GPOs](https://learn.microsoft.com/en-us/training/modules/implement-group-policy-objects/2-define-gpos) | Consola de GPO con vínculos a OU | Oficial (módulo de formación) · inglés | 11 |
| [TechLoris — How to Join a Domain in Windows 10](https://techloris.com/join-domains-in-windows-10/) | Abrir *Propiedades del sistema* con `sysdm.cpl` | Windows 10 · inglés | 12 |
| [ATA Learning — Add a Computer to a Domain](https://adamtheautomator.com/add-computer-to-domain/) | Propiedades del sistema con el dominio | Windows 10 · inglés | 13 |
| [TechDirectArchive — Move a Computer Object to Another OU](https://techdirectarchive.com/2020/04/06/how-to-move-a-computer-object-from-one-container-ou-to-another/) | Equipo dentro de una OU | Windows Server · inglés | 14 |
| [SysOps Ledger (Mark G.) — AD DS and DNS Setup on Windows Server 2022](https://markg001.github.io/SysOps-Ledger/posts/AD-DNS-SETUP/) | Zona inversa con PTR y `nslookup` | Windows Server 2022 · inglés | 15 |
| [ActiveDirectoryPro — Configure DNS Reverse Lookup Zones and PTR Records](https://activedirectorypro.com/configure-dns-reverse-lookup-zones-ptr-records/) | `nslookup` con *UnKnown* | Windows Server · inglés | 16 |
| [TechDirectArchive — DHCP Server on Windows Server 2022](https://techdirectarchive.com/2024/05/04/comprehensive-guide-to-install-dhcp-server-on-windows-server/) | Ámbito DHCP activo | Windows Server 2022 · inglés | 17 |
| [Microsoft Learn — Work with DNS zones and records](https://learn.microsoft.com/en-us/training/modules/implement-windows-server-dns/3-work-dns-zones-records) | Consola DNS con zonas y registros | Oficial (módulo de formación) · inglés | 18 |
| [Microsoft Learn — Define organizational units](https://learn.microsoft.com/en-us/training/modules/create-manage-active-directory-objects/3-define-organizational-units) | Consola Usuarios y equipos de AD | Oficial (módulo de formación) · inglés | 19 |
| [Windows OS Hub — Repairing the Domain Trust Relationship](https://woshub.com/repair-trust-relationship-workstation-with-ad-domain/) | `Test-ComputerSecureChannel` incorrecto | Windows · inglés | 20 |
| [ActiveDirectoryPro — GPResult: How to Check Applied GPOs](https://activedirectorypro.com/gpresult-tool/) | `gpresult /r` | Windows 11 · inglés | 21, 22 |
| [Windows OS Hub — Re-Enable Command Prompt Blocked by Administrator](https://woshub.com/command-prompt-blocked-administrator-policy/) | `cmd` bloqueado y ventana *Restricciones* | Windows 11 · inglés | 23, 24 |
| [Windows OS Hub — Mapping Network Drives via Group Policy](https://woshub.com/map-network-drives-shared-folders-gpo/) | Unidad asignada por GPO | Windows 10 · inglés | 25 |
