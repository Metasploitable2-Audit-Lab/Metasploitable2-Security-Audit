Markdown
# Auditoría de Seguridad: Metasploitable 2

Este repositorio contiene el reporte técnico de la auditoría de seguridad realizada sobre la máquina objetivo **Metasploitable 2** (IP: `10.0.0.2`), con el fin de identificar vulnerabilidades críticas, explotarlas de forma controlada y proponer medidas de mitigación.

---

## 🔍 Fases de la Auditoría

### 1. Descubrimiento y Escaneo de Conectividad de Red (Nmap)

Se ejecutó un escaneo de servicios y versiones para identificar puertos abiertos y vectores de ataque potenciales:

```bash
nmap -sV 10.0.0.2
Resultados destacados:

Puerto 1524/tcp abierto: Ejecuta el servicio ingreslock (Shell interactiva desprotegida / Backdoor).

Otros servicios: FTP (vsftpd 2.3.4), HTTP (Apache 2.2.8), MySQL (5.0.51a).

2. Fase de Explotación y Acceso Inicial (Netcat)
Aprovechando la vulnerabilidad crítica detectada por Nmap en el puerto 1524, se utilizó Netcat para interactuar con la shell desprotegida:

Bash
nc 10.0.0.2 1524
Resultado: Se obtuvo acceso remoto inmediato e interactivo con el nivel de privilegios más alto del sistema operativo (root).

3. Post-Explotación y Criptoanálisis (John the Ripper)
Con acceso total al servidor, se procedió a evaluar la política de credenciales locales:

Se exfiltró el contenido del archivo confidencial de contraseñas (/etc/shadow).

Se trasladó el hash a la carpeta local de auditoría de la máquina atacante.

Se ejecutó John the Ripper para descifrar el hash protegido con el algoritmo MD5crypt:

Bash
john hash.txt
Resultado del cracking: En menos de un segundo, la herramienta descifró la clave del usuario administrador secundario:

Usuario: msfadmin

Contraseña: msfadmin

🔒 Recomendaciones de Mitigación
Cierre de Puertos Críticos: Deshabilitar de forma inmediata el servicio de bindshell en el puerto 1524 y bloquear cualquier tráfico no autorizado mediante reglas de Firewall (iptables).

Robustecimiento de Credenciales: Cambiar la política de contraseñas para prohibir claves por defecto y migrar los hashes de autenticación a algoritmos modernos y resistentes como SHA-512 o bcrypt.
