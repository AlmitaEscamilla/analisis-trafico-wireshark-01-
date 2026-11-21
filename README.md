# analisis-trafico-wireshark-01-
Esta es mi primera práctica de Wireshark. 
# Análisis de Tráfico de Red: Infección por Exploit Kit (Wireshark)

## Resumen del Incidente
Análisis de una captura de tráfico (`2014-11-16-analysis-exercise.pcap`) para identificar una máquina virtual infectada, rastrear el sitio web comprometido, la cadena de redirección del Exploit Kit y el malware descargado.

> **Fuente del ejercicio:** Archivo `2014-11-16-analysis-exercise.pcap` obtenido de [Malware-Traffic-Analysis.net](https://www.malware-traffic-analysis.net/) *(2014-11-16 - TRAFFIC ANALYSIS EXERCISE: QUESTIONS ABOUT EXPLOIT KIT (EK) TRAFFIC)*.

## Herramientas y Comandos Utilizados
*   **Wireshark:** Análisis de paquetes, exportación de objetos HTTP y reconstrucción de flujos (`Follow > HTTP Stream`).
*   **Terminal:** Extracción del hash SHA-256 del archivo sospechoso de forma segura sin ejecutarlo (`sha256sum` en MacOs/Linux).
*   **VirusTotal:** Verificación de reputación de dominios, IPs y hashes.
*   **Filtros principales en Wireshark:**
    *   `http.request` (para listar peticiones web y agregar columnas `Host` y `Referer`)
    *   `dhcp` y `ip.addr`(para identificar el IP sospechoso y el hostname)
   
## Hallazgos y Preguntas Resueltas (IOCs)
| Pregunta / Indicador | Respuesta |
| :--- | :--- |
| **IP de la VM infectada** | `172.16.165.165` |
| **Hostname / MAC de la víctima** | `K34EN6W3N-PC` / `f0:19:af:02:9b:f1` |
| **Sitio web comprometido** | `www.ciniholland.nl` (`82.150.140.30`) |
| **Dominio del Exploit Kit** | `stand.trustandprobaterealty.com` (`37.200.69.143`) |
| **Hash SHA-256 del malware** | `0e3fac547536f773bf1a21180a2294a10be97e956f091d24e168f147ecf5fafd` |
| **Malware identificado** |Troyano (según VirusTotal) |

## Evidencia y Metodología

### 1. Identificación de tráfico anormal: 
<img width="1600" height="901" alt="image" src="https://github.com/user-attachments/assets/261cbbba-2008-423a-84af-dce032abece0" />
Utilicé la ventana de Statistics - Conversations - IPv4 para descubrir la IP que mandaba más packets, encontrando así una conversación entre la IP 172.16.165.165 y la IP 37.200.69.143 que sobrepasaba el tráfico habitual.

### 2. Aplicación de filtro http.request
<img width="1600" height="895" alt="image" src="https://github.com/user-attachments/assets/5acb460e-0387-493b-9d4c-27c0572548f4" />
Después analicé las peticiones web con el filtro http.request e identifiqué aquellas que contuvieraan dominios con nombres aleatorios. Durante esta fase, me centré en las dos IP que ya había identificado para revisar su información.

### 3. Localización del Host Name y MAC de la víctima
<img width="1600" height="913" alt="image" src="https://github.com/user-attachments/assets/07be4f26-1eee-4eab-bfdd-64f23b5ebc5c" />
Apliqué el filtro dhcp and ip.addr == 172.16.165.165, seleccioné uno de los paquetes e hice doble clic para visualizar su información. Una vez dentro, busqué el apartado Client MAC Address `f0:19:af:02:9b:f1`  y de Host Name `K34EN6W3N-PC`.

### 4. Identificación de archivos sospechosos y posible sitio comprometido
<img width="1600" height="888" alt="image" src="https://github.com/user-attachments/assets/5fe150f9-0b9f-467a-a075-d3db09dbe21a" />

Para saber exactamente cual era el dominio que estaba comprometido, utilicé la ventana File - Export Objects - HTTP. Aquí encontré que el sitio `stand.trustandprobaterealty.com` contenía archivos ejecutables (application/x-msdownload)
### 5. Análisis de código HTML
<img width="1600" height="909" alt="image" src="https://github.com/user-attachments/assets/78a781e6-1a9c-4cc7-afd7-6d372d407878" />
Usé la herramienta Follow - HTTP Stream y encontré que contenía un script con numeros y letras aleatorios.

### 6. Extracción del archivo y análisis con SHA-256
<img width="1144" height="428" alt="image" src="https://github.com/user-attachments/assets/d2717393-337d-447e-827b-f020b7dacf03" />
Exporté el archivo sospechoso desde `File > Export Objects > HTTP`. Para analizarlo sin riesgo de infección, obtuve su huella digital desde la terminal de MacOs usando el comando `sha256sum` y el nombre del archivo `2014-11-16-analysis-exercise.pcap`

### 7. Análisis por medio de una threat intel (VirusTotal)
<img width="1600" height="411" alt="image" src="https://github.com/user-attachments/assets/d5ffbe18-301a-45c4-a607-882b3b773e66" />
Una vez obtenida la huella digital, la ingresé en VirusTotal, confirmando mis sospechas de que en efecto había un sitio comprometido y descubriendo el tipo de malware que contenía.
### 8. Cadena de infección
Después de revisar varios paquetes y las IP mencionadas anteriormente, descubrí por medio del campo REFER como fue que el usuario llegó al sitio comprometido y hacia dónde fue redirigido para llegar al EK.

--- 1. El usuario ingresó primero a bing y buscó el sitio `www.ciniholland.nl`
<img width="1600" height="929" alt="image" src="https://github.com/user-attachments/assets/6361f239-ff4b-4ead-b470-f3d16e7b56ad" />

--- 2. Al acceder a este sitio, posiblemente hizo clic en un anuncio de la supuesta tienda `24corp-shop.com` <img width="1600" height="910" alt="image" src="https://github.com/user-attachments/assets/e2eeeb28-f594-4527-9793-dd335a749972" />

--- 3. Finalmente, esta página llevó al sitio que contenía el EK con el malware: `stand.trustandprobaterealty.com`
<img width="1600" height="923" alt="image" src="https://github.com/user-attachments/assets/dbb8cdbb-5993-4c77-84b6-ecafc03b9618" />


