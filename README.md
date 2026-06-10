# Informe de Auditoría Técnica de Seguridad: Metasploitable 2

Este documento constituye el informe técnico final de la auditoría de seguridad y pruebas de penetración (Pentesting) de caja negra realizada sobre la máquina virtual de entrenamiento **Metasploitable 2** (Dirección IP de destino: `10.0.0.2`).

El objetivo primordial de esta auditoría fue identificar puertos abiertos expuestos de manera insegura, explotar vulnerabilidades críticas de forma controlada para demostrar el impacto real de un ataque, y redactar recomendaciones sólidas de mitigación para blindar el sistema.

---

## 📋 Información General de la Auditoría

* **Auditor / Consultor:** Alejandro (Junior Security Consultant)
* **Objetivo de Evaluación:** Metasploitable 2 (`10.0.0.2`)
* **Metasploitable:** Linux-based Operating System
* **Metodología:** OWASP / PTES (Penetration Testing Execution Standard)
* **Fecha de Ejecución:** Junio, 2026

---

## 🔍 Fases Técnicas de la Auditoría

### 1. Reconocimiento y Escaneo de Conectividad (Nmap)

La primera fase consistió en un análisis activo de la superficie de ataque utilizando la herramienta **Nmap** para detectar puertos de escucha TCP, identificar los servicios asociados y determinar con precisión las versiones de software instaladas.

Se ejecutó el siguiente comando de escaneo de alta visibilidad:

```bash
nmap -sV -Pn 10.0.0.2
```

#### Resultados y Análisis del Escaneo:

El escaneo reveló una superficie de ataque extremadamente amplia y peligrosa. A continuación, se detallan los hallazgos prioritarios:

* **Puerto `1524/tcp` (Abierto - Servicio: `ingreslock`):** Se identificó este puerto ejecutando un servicio que actúa como un "backdoor" o puerta trasera interactiva, la cual no requiere ningún tipo de autenticación previa para otorgar una consola de comandos.
* **Puerto `21/tcp` (Abierto - Servicio: `vsftpd 2.3.4`):** Versión de servidor FTP conocida en el ámbito de la ciberseguridad por contener un backdoor integrado de fábrica en su código fuente original.
* **Puerto `80/tcp` (Abierto - Servicio: `Apache httpd 2.2.8`):** Servidor web obsoleto con múltiples vulnerabilidades de divulgación de información y denegación de servicio asociadas.
* **Puerto `3306/tcp` (Abierto - Servicio: `MySQL 5.0.51a`):** Gestor de bases de datos desactualizado expuesto directamente a la red.

---

### 2. Explotación Práctica y Acceso Inicial (Netcat)

Utilizando los datos del escaneo previo, se procedió a realizar una explotación directa e inmediata sobre el puerto crítico `1524`. Dado que el servicio `ingreslock` en esta máquina está configurado para levantar una shell interactiva al recibir conexiones entrantes, se utilizó la herramienta **Netcat** para efectuar la conexión de red.

Se ejecutó el comando de explotación:

```bash
nc 10.0.0.2 1524
```

#### Resultado de la Explotación:

La conexión fue exitosa e inmediata. Al establecer el enlace, el sistema objetivo no solicitó ningún nombre de usuario ni contraseña, entregando de forma instantánea una shell interactiva directa al sistema de archivos de Linux.

Para auditar el nivel de privilegios obtenido con esta shell, se ejecutaron comandos de reconocimiento interno:

```bash
whoami
id
```

La consola retornó el identificador `root` (UID 0), confirmando que el acceso obtenido posee los **privilegios máximos de administración del sistema operativo**. Esto significa que el atacante tiene control total sobre el servidor, los procesos, la red y la información almacenada.

---

### 3. Post-Explotación, Exfiltración y Criptoanálisis (John the Ripper)

Con privilegios de administrador (`root`) consolidados en el servidor objetivo, se procedió a auditar la seguridad de las credenciales de los usuarios locales del sistema mediante una simulación de robo de identidad (exfiltración de contraseñas).

#### 1. Exfiltración de Datos Confidenciales
Se accedió al archivo restringido del sistema donde se almacenan de manera segura las contraseñas cifradas en Linux (`/etc/shadow`). Se copió el hash de autenticación del usuario administrador secundario y se trasladó a un archivo de texto llamado `hash.txt` en la máquina auditora.

El hash exfiltrado correspondiente al usuario `msfadmin` presentaba la siguiente estructura cifrada:
```text
msfadmin:$1$SgIp8Sg7$6vY6ZsdzPfS1.Y3sh69f31:14472:0:99999:7:::
```

#### 2. Criptoanálisis y Fuerza Bruta
Se utilizó la herramienta **John the Ripper** para realizar un ataque de diccionario y descifrado sobre el hash MD5crypt extraído:

```bash
john hash.txt
```

#### Resultado del Descifrado:

En menos de un segundo, la herramienta procesó exitosamente el hash criptográfico y reveló la contraseña en texto plano:

* **Usuario Identificado:** `msfadmin`
* **Contraseña Descifrada:** `msfadmin`

**Diagnóstico de Seguridad:** Este hallazgo representa una vulnerabilidad de severidad alta debido al uso de **credenciales por defecto** (el usuario y la contraseña son idénticos). Cualquier intruso podría haber ingresado de forma legítima al sistema utilizando estas credenciales a través de servicios expuestos como SSH o Telnet.

---

## 🔒 Recomendaciones de Mitigación y Hardening

Para solucionar de manera definitiva las brechas de seguridad identificadas en esta auditoría, se recomienda al equipo de infraestructura aplicar las siguientes contramedidas de inmediato:

### 1. Cierre de Puertos y Servicios Innecesarios
* **Deshabilitar Backdoors:** Detener de forma inmediata el servicio `ingreslock` asociado al puerto `1524`.
* **Configuración de Firewall:** Implementar un firewall local (como `iptables` o `ufw`) para bloquear de manera estricta cualquier puerto que no requiera exposición pública. La política predeterminada del firewall debe ser denegar todo el tráfico entrante (`DROP`) salvo aquel explícitamente autorizado.

### 2. Políticas de Robustecimiento de Credenciales (Hardening)
* **Cambio de Contraseñas:** Reemplazar de forma obligatoria la contraseña predeterminada del usuario `msfadmin` por una clave robusta que cumpla con políticas de complejidad (mínimo 12 caracteres, mayúsculas, minúsculas, números y caracteres especiales).
* **Migración Criptográfica:** Configurar el sistema operativo para que los nuevos hashes generados en `/etc/shadow` utilicen algoritmos modernos y resistentes a ataques de fuerza bruta como **SHA-512** (con rondas de hashing configurables) o **bcrypt**, reemplazando el obsoleto estándar MD5crypt.
