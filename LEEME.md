🕵️‍♂️#4 SOC-Incident-Report-Traffic-Analysis-NEMOTODES
Trabajas como analista en un Centro de Operaciones de Seguridad (SOC) para una instalación de investigación médica especializada en nematodos. Las alertas de tráfico en tu red indican que alguien ha sido infectado. No sabes qué es más asqueroso, si los nematodos o el malware.

🌎 Contexto
Se detectó una intrusión en la red que involucra a un endpoint de Windows (10.11.26.183). La cadena de infección se activó mediante un escenario de "drive-by download" o "web-inject". El usuario visitó inicialmente un sitio web legítimo pero comprometido, classicgrand.com, que sirvió como punto de entrada. Esta visita resultó en una redirección no autorizada a un dominio de entrega malicioso, lo que llevó al despliegue de un Troyano de Acceso Remoto (RAT).

El programa utilizado para analizar los paquetes fue Wireshark 🦈​

🌎​ Detalles del Entorno
- Rango del segmento LAN: 10.11.26[.]0/24 (10.11.26[.]0 a 10.11.26[.]255)
- Dominio: nemotodes[.]health
- Controlador de dominio de Active Directory (AD): 10.11.26[.]3 - NEMOTODES-DC
- Nombre del entorno de AD: NEMOTODES
- Puerta de enlace (Gateway) del segmento LAN: 10.11.26[.]1
- Dirección de broadcast del segmento LAN: 10.11.26[.]255

​🔎​ Hallazgos:
- Dirección IP infectada: 10.11.26.183
- Dirección MAC infectada: d0:57:7b:ce:fc:8b
- Hostname infectado: DESKTOP-B8TQK49
- Nombre de cuenta de usuario de Windows infectada: oboomwald

⚠️ IOCs:
- Direcciones IP maliciosas: 194.180.191.64 (Canal C2 establecido a través de HTTP/HTTPS por el puerto 443) (NetSupport RAT)
- Direcciones IP sospechosas: 104.117.247.184 (Parece que el atacante intentaba geolocalizar a la víctima).
- Dominios maliciosos: modandcrackedapk.com (Fue redireccionado aquí) y classicgrand.com (Inicio de la infección)
- Puerto utilizado: 443 (TCP) (Utilizado para el tráfico de la RAT)
- Protocolo peligroso detectado: SMBv1

🛜 Comportamiento de Red:
Redirección inicial: El análisis de tráfico muestra una referencia de classicgrand.com a modandcrackedapk.com.
Establecimiento de C2: El host infectado estableció una conexión persistente con la IP 194.180.191.64 a través del puerto TCP 443. Aunque utiliza un puerto típicamente reservado para HTTPS, el tráfico fue identificado como actividad del protocolo NetSupport RAT (Check-ins y respuestas de administración remota).
Reconocimiento: El malware realizó una búsqueda de geolocalización a través de ip-api.com (104.26.1.231) para identificar la ubicación física de la víctima, una táctica común para la categorización de botnets.
Vulnerabilidad interna: La detección de SMBv1 e intentos de desbordamiento de NetBIOS sugiere que el malware o el atacante pudieron haber intentado un movimiento lateral o la recolección de credenciales dentro de la red local.

🧐​ Procesos y Filtros:
Como analistas del SOC, tenemos estas alertas:

<img width="1600" height="637" alt="image" src="https://github.com/user-attachments/assets/c0fa310d-f9c8-4332-83bb-e319c34c4a38" />

Estas alertas nos permiten identificar ciertos datos potencialmente útiles. Tenemos direcciones IP tanto externas como internas. Indican el uso del protocolo SMBv1 (inseguro), la presencia de dominios peligrosos como "modandcrackedapk.com", el puerto 443 (que se está utilizando con un propósito potencialmente malicioso), actividad de NetSupport RAT (troyano) y, finalmente, un intento de geolocalización.

<img width="952" height="569" alt="image" src="https://github.com/user-attachments/assets/06b7d2a6-7757-42fd-a5e2-f2174cb47250" />

ip.addr == 10.11.26.183 && nbns
- Con este filtro conocemos el Hostname infectado, la dirección IP infectada y la dirección MAC infectada.

<img width="772" height="719" alt="image" src="https://github.com/user-attachments/assets/48da7698-4f64-4842-b79f-eb00c37ca068" />

ip.addr == 10.11.26.183 && kerberos.CNameString
- Con este filtro conocemos el nombre de la cuenta de usuario de Windows infectada.

<img width="1630" height="363" alt="image" src="https://github.com/user-attachments/assets/f5d34eab-604f-411e-9c9f-121c9c34a193" />

Haciendo beaconing e ip.addr == 10.11.26.183 && dns, podemos descubrir un dominio malicioso.

😎 Conclusión:
El endpoint ha sido comprometido con éxito y reclutado en una infraestructura de Botnet controlada a través de la RAT NetSupport. Esta herramienta proporciona al atacante capacidades completas de escritorio remoto, acceso al sistema de archivos y la capacidad de desplegar cargas útiles secundarias como ransomware o ladrones de información (info-stealers). La actividad se clasifica como un Incidente de Seguridad Crítico debido a la persistencia establecida y al alto nivel de control que posee el actor externo.

🛡️ Recomendaciones como Profesional de Ciberseguridad:
1) Aislar el host: Desconectar inmediatamente la 10.11.26.183 de la red para evitar una mayor exfiltración de datos o movimientos laterales.

2) Restablecimiento de credenciales: Forzar un cambio de contraseña para la cuenta de usuario oboomwald y cualquier otra cuenta utilizada en esa máquina, ya que deben considerarse comprometidas.

3) Erradicación: Realizar una limpieza completa y reinstalar la imagen del sistema operativo de la estación de trabajo afectada. Limpiar manualmente una infección de RAT no es confiable debido a los posibles mecanismos de persistencia ocultos.

4) Endurecimiento de la red (Network Hardening): Desactivar SMBv1 en todo el dominio para mitigar los riesgos de exploits heredados. Bloquear las IPs de los IOC identificados (194.180.191.64, 104.117.247.184) en el firewall corporativo.

5) Filtrado web: Implementar una solución EDR o un Proxy Web para bloquear el acceso a dominios maliciosos conocidos como modandcrackedapk.com.
