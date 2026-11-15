# 🛡️ MiniSOC Web

**Panel de alertas básicas de ciberseguridad para micro y pequeñas empresas (mipymes), en lenguaje simple y en dos idiomas (español/inglés).**

MiniSOC traduce eventos técnicos de red (intentos fallidos de acceso, dispositivos desconocidos, puertos abiertos, tráfico inusual) en alertas con **semáforo de riesgo**, una explicación de *qué significa* y una lista de *qué hacer*. Es una sola página (`index.html`): sin servidor, sin instalación y sin dependencias. Todo el análisis ocurre **en el navegador**; nada se envía a ningún servidor.

> Proyecto de **Ingeniería II** · Universidad Nacional Abierta y a Distancia (UNAD) · Escuela de Ciencias Básicas, Tecnología e Ingeniería (ECBTI).
> Prototipo de laboratorio alineado con los **ODS 8 y 9** y con la línea de acción **Tecnológica / Económica–productiva** del OIR.

**Demo en línea:** 

---

## Contenido

1. [Qué problema resuelve](#qué-problema-resuelve)
2. [Características](#características)
3. [Estructura del repositorio](#estructura-del-repositorio)
4. [Uso rápido](#uso-rápido)
5. [Cómo obtener tus propios datos](#cómo-obtener-tus-propios-datos)
6. [Archivos de ejemplo](#archivos-de-ejemplo)
7. [Reglas de riesgo](#reglas-de-riesgo-editables)
8. [Cómo funciona el código](#cómo-funciona-el-código)
9. [Cómo extenderlo](#cómo-extenderlo)
10. [Privacidad y seguridad](#privacidad-y-seguridad)
11. [Limitaciones](#limitaciones)
12. [Publicar en GitHub Pages](#publicar-en-github-pages)
13. [Uso de IA, ética y créditos](#uso-de-ia-ética-y-créditos)

---

## Qué problema resuelve

Las mipymes dependen de redes y servicios digitales, pero casi nunca tienen personal de TI ni presupuesto para un SOC (*Security Operations Center*). Las soluciones comerciales muestran registros técnicos difíciles de interpretar. MiniSOC ofrece una **primera línea de defensa accesible**: le dice al usuario, con palabras simples, si su red necesita atención y qué hacer. **No reemplaza** a un SOC profesional.

## Características

- **Semáforo de estado** (rojo/amarillo/verde) con contadores de alertas por nivel de riesgo.
- **Explicación y pasos a seguir** para cada alerta, con una lista de verificación interactiva.
- **Interfaz bilingüe ES/EN**, con cambio de idioma en tiempo real y sin recargar.
- **Analizador de datos propios**: pega o carga registros y obtén alertas al instante.
- **Historial con filtros** por nivel de riesgo y estado (nueva, atendida, ignorada).
- **Reglas editables** (umbrales y puertos de riesgo) en formato JSON.
- **Diseño adaptable** (celular y escritorio) y **modo oscuro/claro automático**.
- **Accesible**: tipografía de alta legibilidad, foco visible, navegación por teclado y el nivel de riesgo siempre acompañado de texto (no solo color).
- **Privado**: procesamiento local; solo se usa `localStorage` en tu propio navegador.

## Estructura del repositorio

```
.
├── index.html                      # Toda la aplicación (HTML + CSS + JavaScript)
├── README.md                       # Este documento
└── muestras/                       # Archivos de ejemplo para probar el analizador
    ├── auth_ejemplo.log            # Inicios de sesión fallidos (formato sshd)
    ├── arp_ejemplo.txt             # Tabla ARP (formato Windows `arp -a`)
    ├── dispositivos_conocidos.txt  # Lista de MAC conocidas
    └── puertos_ejemplo.txt         # Salida tipo nmap
```

## Uso rápido

**Opción 1: abrir el archivo.** Descarga el repositorio y abre `index.html` en tu navegador.

**Opción 2: servidor local** (si tu navegador restringe algo al abrir el archivo directamente):

```bash
python -m http.server 8000
# abre http://localhost:8000
```

**Primer recorrido:**

1. Abre la pestaña **Analizar datos**.
2. Pulsa **Cargar datos de ejemplo** y luego **Analizar**.
3. Revisa el **Panel** (semáforo en rojo), abre una alerta con **¿Qué hago?**, marca los pasos y pulsa **Marcar como atendida**.
4. Cambia el idioma con **ES | EN** y revisa el **Historial** con sus filtros.

## Cómo obtener tus propios datos

Usa **solo tu propio equipo o una red de pruebas**.

| Dato | Windows | Linux / macOS |
|---|---|---|
| Inicios de sesión fallidos | (registro de eventos; adaptar a formato sshd si aplica) | `sudo grep "Failed password" /var/log/auth.log` o `journalctl -u ssh` |
| Tabla ARP | `arp -a` | `ip neigh` o `arp -a` |
| Puertos abiertos | `netstat -an \| findstr LISTENING` | `ss -tuln` o `netstat -an \| grep LISTEN` |
| Puertos con servicio (opcional) | `nmap 127.0.0.1` | `nmap 127.0.0.1` |
| Tráfico | Valor en Mbit/s que mida tu herramienta (opcional) | igual |

Cada bloque admite **pegar texto** o **cargar un archivo** (`.log`, `.txt`, `.csv`).

**Detectar dispositivos nuevos:** primero analiza tu red en un momento normal y pulsa **Usar los equipos del ARP como conocidos**. Esa lista (MAC) se guarda en tu navegador. En análisis posteriores, cualquier MAC que no esté en la lista genera una alerta. *Si la lista está vacía no se generan alertas de dispositivos.*

## Archivos de ejemplo

La carpeta [`muestras/`](muestras/) permite reproducir el análisis completo sin datos reales.

| Archivo | Qué contiene | Dónde pegarlo |
|---|---|---|
| `auth_ejemplo.log` | 12 intentos fallidos de `root` desde `203.0.113.9`, 5 de `admin` desde `192.168.1.23`, 2 de `caja` desde `192.168.1.40` y un inicio de sesión exitoso (se ignora) | Bloque 1 |
| `arp_ejemplo.txt` | 4 equipos de la red `192.168.1.x`; el último (`192.168.1.47`) no es conocido | Bloque 2 (ARP) |
| `dispositivos_conocidos.txt` | Las 3 MAC de los equipos normales | Bloque 2 (conocidos) |
| `puertos_ejemplo.txt` | Salida tipo `nmap` con los puertos 22, 3389 y 8080 abiertos | Bloque 3 |

Escribe **`12`** en el campo de tráfico (Bloque 4) para completar el ejemplo.

**Resultado esperado: 7 alertas.**

| Nivel | Alerta | Por qué |
|---|---|---|
| 🔴 Alto | 12 intentos fallidos desde `203.0.113.9` | 12 ≥ umbral alto (10) |
| 🔴 Alto | Dispositivo desconocido `192.168.1.47` | MAC fuera de la lista de conocidos |
| 🔴 Alto | Puerto 3389 abierto | Escritorio remoto: puerto de riesgo alto |
| 🟠 Medio | 5 intentos fallidos desde `192.168.1.23` | 5 ≥ umbral medio (3) |
| 🟠 Medio | Tráfico de 12 Mbit/s | 12 ≥ umbral medio (10) |
| 🟡 Bajo | 2 intentos fallidos desde `192.168.1.40` | 2 ≥ umbral bajo (2) |
| 🟡 Bajo | Puerto 8080 abierto | Servidor web alterno: riesgo bajo |

El puerto 22 se ignora porque no está en la lista de puertos de riesgo. El botón **Cargar datos de ejemplo** de la aplicación carga estos mismos datos.

## PASOS AVANZADOS

## Reglas de riesgo (editables)

Las reglas están en un cuadro JSON dentro de **Analizar datos → Reglas de riesgo**. Valores por defecto:

```json
{
  "login_fallidos": { "bajo": 2, "medio": 3, "alto": 10 },
  "trafico_mbps": { "medio": 10, "alto": 50 },
  "dispositivo_desconocido": "alto",
  "puertos_riesgosos": {
    "21":   { "servicio": "FTP", "nivel": "medio" },
    "23":   { "servicio": "Telnet", "nivel": "alto" },
    "445":  { "servicio": "SMB (archivos compartidos)", "nivel": "alto" },
    "3306": { "servicio": "MySQL", "nivel": "medio" },
    "3389": { "servicio": "Escritorio remoto", "nivel": "alto" },
    "5900": { "servicio": "VNC", "nivel": "medio" },
    "8080": { "servicio": "Servidor web alterno", "nivel": "bajo" }
  }
}
```

- `login_fallidos`: cantidad mínima de intentos fallidos **desde una misma IP** para cada nivel.
- `trafico_mbps`: umbrales de tráfico en Mbit/s.
- `dispositivo_desconocido`: nivel que se asigna a un equipo fuera de la lista de conocidos.
- `puertos_riesgosos`: puerto → servicio y nivel (`bajo`, `medio` o `alto`).

Si el JSON no es válido, la aplicación avisa y no analiza. Los cambios a las reglas **no** se guardan al recargar la página.

---

## Cómo funciona el código

Todo está en `index.html`, organizado en tres capas: estilos, datos y lógica. No hay compilación ni librerías externas (solo una tipografía opcional de Google Fonts, con alternativa del sistema).

### 1. Variables CSS (`:root` y `@media`)

- Un sistema de diseño con variables personalizadas: `--bg` (fondo), `--sf` (superficies), `--ink` (texto), `--mu` (texto secundario), `--ln` (líneas), `--br` (color de marca) y las parejas de color por nivel de riesgo (`--tx-a/m/b` para texto y `--bg-a/m/b` para fondos).
- **Modo oscuro/claro automático** con `@media (prefers-color-scheme: dark)`. También se puede forzar con el atributo `data-theme="dark"` o `"light"` en `<html>`.
- **Diseño adaptable (responsive)**: reglas `@media (max-width: 760px)` reorganizan el semáforo, las columnas y las listas.
- **Márgenes seguros** para celulares con muesca mediante `viewport-fit=cover` y `env(safe-area-inset-*)`.
- Se respeta `prefers-reduced-motion` para reducir animaciones.

### 2. Diccionarios de idioma (`UI` y `TY`)

- `UI` (`es` / `en`): todos los textos de la interfaz (menús, botones, etiquetas, mensajes y encabezados).
- `TY`: el contenido de cada **tipo de alerta** (`device`, `login`, `port`, `traffic`). Cada tipo tiene, en ambos idiomas, `t` (título, con marcadores como `{n}`), `m` (qué significa), `s` (pasos a seguir) y `x` (datos técnicos por defecto).
- El idioma activo vive en la variable `lang`; al cambiarlo, `render()` vuelve a pintar todo sin recargar la página.

### 3. Modelo de datos

Cada alerta es un objeto con esta forma:

```js
{
  id: 123456,              // número estable calculado a partir de `fp`
  fp: "login:203.0.113.9", // "huella": identifica la alerta entre análisis
  ty: "login",             // tipo: device | login | port | traffic
  lv: "a",                 // nivel: a (alto) | m (medio) | b (bajo)
  st: "new",               // estado: new | done | ign
  n: 12,                   // dato numérico (intentos, puerto…) para el título
  x: "Cuenta/Account: root · IP: 203.0.113.9", // detalle técnico
  w: ["Analizado 10:42", "Analyzed 10:42"]      // cuándo, en ES y EN
}
```

La lista actual está en la variable `A`. Al inicio contiene datos de ejemplo (`SAMPLE`); tras ejecutar un análisis se reemplaza por las alertas calculadas y el indicador de modo cambia de *Datos de ejemplo* a *Tus datos analizados*.

### 4. Vistas y renderizado dinámico

- `render()` lee la URL (`location.hash`) y pinta la vista dentro de `<main id="app">`. Las rutas son: `#panel`, `#alerta`, `#historial` y `#analizar`.
- **`panel()`**: estado de la red con el semáforo (rojo si hay alertas altas, ámbar si hay medias, verde si no hay alertas abiertas), contadores por nivel y lista de alertas abiertas ordenadas por riesgo.
- **`detail()`**: explicación de la alerta seleccionada (`sel`), pasos con casillas de verificación, datos técnicos dentro de un bloque `<details>` y las acciones *Marcar como atendida*, *Ignorar* y *Volver*.
- **`hist()`**: tabla de las alertas del análisis (abiertas, atendidas e ignoradas) con filtros por nivel y estado, y una barra lateral de configuración.
- **`analyze()`**: formulario con cuatro bloques de entrada (log, ARP + conocidos, puertos y tráfico), carga de archivos, reglas JSON y botones de ejemplo y análisis.
- Un `hashchange` vuelve a llamar a `render()` y mueve el foco al título (`<h1>`) para los lectores de pantalla.

### 5. Motor de análisis (`run()`, `arp()` y funciones auxiliares)

`run()` lee las reglas, procesa cada bloque con expresiones regulares y arma la lista de alertas:

| Fuente | Qué reconoce | Lógica |
|---|---|---|
| Inicios de sesión | `Failed password for [invalid user] USUARIO from IP` (sshd) y `authentication failure … rhost=IP user=USUARIO` (PAM) | Cuenta los intentos **por IP** y los compara con `login_fallidos` mediante `lvl()` |
| ARP | Pares IP + MAC (separadas por `:` o `-`) mediante `arp()`; descarta direcciones de difusión (`255`, `224`, `239`, `ff:ff:ff:ff:ff:ff`) | Cada MAC que no está en la lista de conocidas genera una alerta del nivel `dispositivo_desconocido` |
| Puertos | `PUERTO/tcp open` (nmap) y líneas con `LISTEN`/`LISTENING` (netstat, ss) | Si el puerto está en `puertos_riesgosos`, genera la alerta con su servicio y nivel |
| Tráfico | Número en Mbit/s | Se compara con `trafico_mbps` |

Funciones auxiliares: `lvl(valor, umbrales)` devuelve `"a"`, `"m"`, `"b"` o `null`; `hash()` calcula el `id` estable; `arpMacs()` extrae las MAC únicas de una tabla ARP; `esc()` escapa el texto antes de mostrarlo.

### 6. Persistencia y privacidad

- `localStorage["minisoc_known"]`: lista de MAC conocidas (se guarda al analizar).
- `localStorage["minisoc_st"]`: estado (`done` / `ign`) de cada alerta por su `fp`, de modo que una alerta atendida siga marcada en análisis posteriores.
- Todo se maneja con `try/catch`: si el navegador bloquea el almacenamiento, la aplicación sigue funcionando.
- No hay peticiones de red de datos: ninguna información sale del navegador.

### 7. Eventos

La aplicación usa **delegación de eventos**: un solo `click`, un `change` y un `input` en `document`. Los botones declaran su acción con `data-act` (`open`, `done`, `ign`, `back`, `lang`, `sample`, `learn`, `run`, `save`) y los campos con `data-d` (datos del analizador), `data-f` (filtros y configuración) o `data-fi` (carga de archivos).

### Flujo de datos

```
 Pegar / cargar datos ──► run() ──► alertas (A) ──► render() ──► Panel / Detalle / Historial
        ▲                    │                           │
   reglas JSON               ▼                           ▼
                     localStorage (MAC conocidas, estados de alertas)
```

---

## Cómo extenderlo

**Agregar un nuevo tipo de alerta** (por ejemplo, «Antivirus desactivado»):

1. En `TY`, agrega `av: { es: {t, m, s, x}, en: {t, m, s, x} }` con sus textos en ambos idiomas.
2. En `run()`, agrega su analizador con `add("av", nivel, "av:clave", n, detalle)`.
3. Si usa umbrales, agrégalos al objeto `RULES`.
4. Si necesita datos nuevos, agrega un bloque en `analyze()` y su clave en el objeto `D`.

**Agregar un idioma:** duplica los objetos `es` de `UI` y de cada tipo en `TY`, traduce y añade un botón en `.lang`. Revisa `li()`, que elige el idioma de la hora de cada alerta.

**Ideas futuras:** backend opcional (por ejemplo, en Python) para recolectar datos automáticamente de una red de pruebas, notificaciones por correo, soporte de más formatos de registro (por ejemplo, eventos de Windows) y exportación de alertas.

## Privacidad y seguridad

- El análisis es **100 % local**: los datos pegados o cargados no se envían a ningún servidor.
- Aun así, **anonimiza** IP, cuentas y MAC reales antes de compartir capturas del panel.
- Analiza **únicamente tu propio equipo o una red de pruebas**. Revisar redes ajenas puede ser ilegal.
- No publiques en el repositorio registros, tablas ARP ni listas de dispositivos reales.

## Limitaciones

- **No monitorea en tiempo real**: una página web no puede leer tu red por sí sola; analiza los datos que le entregas.
- Es un **prototipo de laboratorio** y no reemplaza a un SOC profesional ni garantiza detectar ataques reales.
- **Historial limitado**: muestra las alertas del análisis actual; solo se recuerdan entre sesiones los estados (atendida/ignorada) y la lista de MAC conocidas.
- En **Historial → Configuración**, la sensibilidad y el correo ante alertas altas son **elementos de interfaz del prototipo**; aún no modifican el análisis ni envían correos.
- El analizador reconoce el formato sshd/PAM, ARP de Windows/Linux, nmap, netstat y ss. Otros formatos pueden requerir ajustes.
- El idioma no se recuerda al recargar la página (siempre inicia en español).





