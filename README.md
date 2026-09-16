# HSEQ Field Master Pro — Grupo Eulen Colombia 🛡️🇨🇴

Sistema progresivo (PWA) de inspección técnica en terreno, captura fotográfica con estampa corporativa inviolable de alta resolución, monitoreo satelital continuo (GPS Alta Precisión) y asistente automatizado de bitácora y reporte diario.

---

## 📑 Contenido del Proyecto

```
hseq-eulen-colombia/
├── index.html                 # Aplicación web completa y optimizada para móviles
├── manifest.json              # Manifiesto PWA para instalación nativa (Android / iOS)
├── sw.js                      # Service Worker para caché offline y estabilidad en campo
├── vercel.json                # Configuración de políticas de seguridad y permisos para Vercel
├── assets/
│   ├── logo-eulen-dark.jpg    # Logo Grupo Eulen sobre fondo azul (para overlay nocturno/azul)
│   └── logo-eulen-light.jpg   # Logo Grupo Eulen sobre fondo blanco (para header y overlay claro)
└── README.md                  # Este manual técnico de despliegue e integraciones
```

---

## 🚀 1. Guía Paso a Paso: Publicar en GitHub y Vercel (Para Pruebas Hoy)

Para que los navegadores móviles (Chrome en Android y Safari en iOS) permitan el uso del **Chip GPS** y de la **Cámara Móvil**, es **obligatorio** que la aplicación se sirva bajo un dominio con **certificado SSL activo (HTTPS)**. 
Vercel y GitHub proporcionan este entorno seguro de manera instantánea y gratuita.

### Opción A: Despliegue Directo vía GitHub y Vercel (Recomendado)

#### Paso 1: Subir el proyecto a GitHub
1. Ingresa a [github.com](https://github.com) e inicia sesión con tu cuenta.
2. Haz clic en **New Repository** (Nuevo Repositorio).
3. Asígnale el nombre: `hseq-eulen-colombia` (puedes dejarlo público o privado).
4. Si tienes Git instalado en tu equipo, ejecuta en la carpeta del proyecto:
   ```bash
   git init
   git add .
   git commit -m "feat: version inicial HSEQ Grupo Eulen Colombia con logos y permisos"
   git branch -M main
   git remote add origin https://github.com/TU-USUARIO/hseq-eulen-colombia.git
   git push -u origin main
   ```
   *(Si no utilizas Git por consola, puedes pulsar el botón "uploading an existing file" en GitHub y arrastrar todos los archivos de esta carpeta).*

#### Paso 2: Conectar con Vercel
1. Entra a [vercel.com](https://vercel.com) e inicia sesión con tu cuenta de GitHub.
2. En el panel principal (Dashboard), haz clic en **Add New...** > **Project**.
3. Verás el repositorio `hseq-eulen-colombia` en la lista. Haz clic en **Import**.
4. En la pantalla de configuración:
   - **Framework Preset**: Dejar en *Other* (Detecta automáticamente el sitio estático).
   - **Root Directory**: `./`
5. Haz clic en **Deploy**.
6. En menos de 30 segundos, Vercel generará una URL pública segura, por ejemplo:
   `https://hseq-eulen-colombia.vercel.app`

#### Paso 3: Enlace para el equipo de Eulen
- Comparte esa URL por WhatsApp o correo al equipo de supervisores.
- Al abrirlo en el smartphone, aparecerá el banner de bienvenida y la solicitud automática de permisos.
- Pueden pulsar **"Agregar a pantalla de inicio"** en Chrome o Safari para tener la App como un icono nativo en su teléfono.

---

## 🌐 2. Integración con el Ecosistema de Google (Drive, Sheets, Maps, Looker Studio, Gemini)

¿Es posible vincular esta aplicación con las plataformas y apps de Google? **Sí, al 100%**, y existen dos arquitecturas muy potentes:

### Arquitectura 1: Conexión Serviceless con Google Sheets y Google Drive (Sin costo de servidores)
A través de un **Google Apps Script Web App**, la aplicación web puede enviar directamente cada fotografía capturada y cada fila de inspección a tu Google Drive institucional y a una hoja de cálculo en tiempo real.

#### ¿Cómo funciona el flujo?
```
[Smartphone Inspector] 
        │ (POST con Foto Base64 + Coordenadas GPS + Datos Eulen)
        ▼
[Google Apps Script Webhook]
   ├──> Guarda la fotografía estampada en una Carpeta de Google Drive (ej: "Fotos_HSEQ_Eulen_2026")
   └──> Inserta una fila en Google Sheets ("Bitácora_Maestra_HSEQ") con fecha, inspector, cliente, coordenadas y link a la foto.
```

#### Código listo para implementar en Google Apps Script:
1. Ve a [script.google.com](https://script.google.com) y crea un nuevo proyecto: `Receptor_HSEQ_Eulen`.
2. Pega el siguiente código en el editor:

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    
    // 1. Conectar con Google Sheet
    var sheetId = "TU_ID_DE_GOOGLE_SHEET_AQUI";
    var sheet = SpreadsheetApp.openById(sheetId).getActiveSheet();
    
    // 2. Conectar con Carpeta de Google Drive para fotos
    var folderId = "TU_ID_DE_CARPETA_DRIVE_AQUI";
    var folder = DriveApp.getFolderById(folderId);
    
    var photoUrl = "Sin foto";
    if (data.imageBase64) {
      var imageBytes = Utilities.base64Decode(data.imageBase64.split(',')[1]);
      var blob = Utilities.newBlob(imageBytes, 'image/png', 'HSEQ_' + data.inspector + '_' + new Date().getTime() + '.png');
      var file = folder.createFile(blob);
      file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
      photoUrl = file.getUrl();
    }
    
    // 3. Insertar fila en Google Sheet
    sheet.appendRow([
      new Date(),
      data.inspector,
      data.cliente,
      data.proyecto,
      data.categoria,
      data.actividad,
      data.coordenadas,
      data.clima,
      data.descripcion,
      photoUrl
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({ status: "success", photoUrl: photoUrl }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ status: "error", message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```
3. Haz clic en **Implementar** (Deploy) > **Nueva implementación** > Tipo: **Aplicación Web**.
   - Acceso: *Cualquier usuario* (Anyone).
4. Copias la URL generada y con solo un `fetch(GAS_URL, { method: 'POST', body: JSON.stringify(payload) })` desde la app, todo queda centralizado en el Workspace de Google de Grupo Eulen.

---

### Arquitectura 2: Tableros Ejecutivos en Google Looker Studio
- Al tener los datos cayendo en vivo a tu Google Sheet, abres [lookerstudio.google.com](https://lookerstudio.google.com).
- Seleccionas la hoja de cálculo como fuente de datos.
- Puedes generar de inmediato:
  - **Mapa de geolocalización en vivo** con los pines de cada supervisor en Colombia.
  - **Gráfico de torta** por tipo de novedad (Acto inseguro, Condición, Ambiental, Seguimiento).
  - **KPI de cobertura diaria** de clientes y frentes de trabajo.

---

### Arquitectura 3: Inteligencia Artificial con Google Gemini Vision API
- Es posible añadir un botón **"Analizar Riesgos con Gemini"** en la app.
- Al capturar la foto, el modelo multimodal de Gemini analiza la imagen en 2 segundos e identifica automáticamente:
  - Uso correcto de EPP (casco, chaleco reflectivo, botas, gafas).
  - Condiciones de orden y aseo o riesgos de caída.
  - Redacción preliminar técnica del hallazgo para ahorrar tiempo al inspector.

---

## 📱 3. Protocolo de Pruebas para Hoy con el Equipo

Para realizar las pruebas piloto hoy con supervisores e ingenieros de Grupo Eulen Colombia:

1. **Prueba de Permisos**:
   - Abrir el enlace en el navegador móvil.
   - Verificar que aparezca la solicitud nativa: *"Permitir que el sitio use la cámara y tu ubicación"*.
   - Presionar **Permitir**.
   - En caso de dudas, tocar el botón **"Permisos"** en la barra superior para ver el diagnóstico en vivo.

2. **Prueba de Captura y Estampa**:
   - Tocar **"Abrir Cámara"**, apuntar a una estación de trabajo o EPP y pulsar **"CAPTURAR FOTO"**.
   - Comprobar que en la vista previa aparece automáticamente el **Logo oficial de Grupo Eulen Colombia** con proporción nítida, las coordenadas exactas de la sede, fecha, hora y descripción.
   - Pulsar **"Descargar Foto Estampada"** y verificar que guarde el archivo en la galería en alta resolución.

3. **Prueba de Monitoreo GPS**:
   - Ir a la pestaña **"Rastreo GPS"**.
   - Dar unos pasos por el sitio y comprobar que la tabla registra los puntos en coordenadas Decimales y Grados/Minutos/Segundos.
   - Probar el botón **"Compartir WhatsApp / Mapa"** para enviar la ubicación con pin de Google Maps a la jefatura.
   - Descargar el archivo `.xlsx` de ruta.

4. **Prueba del Asistente Bitácora**:
   - Ir a la pestaña **"Asistente Bitácora"**.
   - Responder las 6 preguntas rápidas del Chatbot de Eulen.
   - Verificar la redacción formal del informe diario y descargar el consolidado en Excel.
