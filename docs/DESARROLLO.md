# PDF Crawler · Code Toys

Visor PDF nativo en **Rust** con una araña joya que lee contigo: camina sobre las palabras reales,
arranca las cifras del papel (montos, metrados, porcentajes) y, cuando te detienes, teje una
telaraña orbital sobre la cifra más importante.

Coded by Jhon Mires.

## Colaboradores en «Acerca de»

Edita `colaboradores.txt` (junto al `.bat`): una línea por colaborador y `[Categoría]` para
agrupar. La primera categoría es la principal (nombres más grandes y con brillo). Al compilar,
`build.rs` graba la lista dentro del ejecutable, ofuscada y con suma de verificación: el `.txt`
no se distribuye, y si alguien altera los nombres dentro del `.exe`, la sección no se muestra.
Cada cambio en el archivo se aplica en la siguiente compilación. Si la lista es larga, se
desplaza en la tarjeta como los créditos de una película.

## Dos modos: LECTURA y PLANOS

El interruptor **LECTURA | PLANOS** de la barra superior cambia el modo de vista. Con la marca
**AUTO** el visor elige solo al abrir cada archivo:

- Se puntúa cada página por tamaño de hoja (A3 o mayor, horizontal), código de lámina en el
  rótulo (`E-01`, `A-02`), textos de escala (`ESC 1:50`) y poca densidad de texto corrido.
- En documentos mixtos (memoria + láminas) el modo sigue a la página: al llegar leyendo a una
  lámina pasa a PLANOS; al elegir una página de texto en la lista de láminas vuelve a LECTURA.
- Si cambias el modo a mano, la elección se recuerda para ese archivo
  (`%APPDATA%\PDFCrawler\modos.txt`).

### Modo planos

- **Lámina en 2D** con vista previa de la hoja completa y mosaicos de 1024 px solo de lo visible,
  con presupuesto de memoria y expulsión LRU. Zoom con la rueda anclado al cursor.
- **Trazos vivos:** la geometría vectorial del PDF (líneas, curvas, rectángulos, bloques) se lee
  en el hilo de PDFium y se indexa en un hilo aparte (rejilla espacial compacta). Al pasar el
  cursor se enciende el trazo completo con su longitud real según la escala de la vista; un clic
  lo guarda como medida (con área si el contorno es cerrado). Al abrir cada lámina un barrido de
  escaneo enciende los trazos.
- **Medir:** HILO, ARCO (longitud, radio y ángulo), ÁNGULO, ÁREA (un clic dentro de un contorno
  cerrado) y CALIBRAR. Los clics se ajustan a extremos, puntos medios o sobre la línea (Mayús
  para un punto libre). Cada medida usa la escala de la vista donde empieza.
- **Marcar:** NUBE, LIDAR (su haz hace brillar las líneas del plano que toca), MARCO (muestra el
  tamaño real), CÍRCULO, FLECHA y TELARAÑA. Cada marca tiene texto, prioridad y estado; se
  mueven, se ajustan desde sus puntos y se cambian de estilo con un clic.
- **Borrar y deshacer:** botón BORRAR, tecla Supr o la ✕ de la lista; Ctrl+Z deshace.
- **Guardar en el PDF:** GUARDAR PDF crea una copia `_revisado.pdf` con cada marca como anotación
  (se ve en cualquier visor y Acrobat la lista como comentario, con su texto). Los datos
  editables viajan adjuntos dentro del mismo PDF: al reabrirlo aquí las marcas vuelven a ser
  editables y no se duplican.
- **Contenido de la lámina** (vistas y escalas, acero por diámetro, niveles, cotas) y búsqueda en
  todas las láminas; la araña te lleva a cada resultado.

Probado con una lámina A1 de unos 386 000 segmentos (tramas, círculos y bloques repetidos):
la geometría queda lista en unos 2 s, ocupa ~20 MB, y la imagen se mantiene en decenas de MB
gracias a los mosaicos.

### Exportar a Word y Excel

En la pestaña MARCAS, **EXPORTAR WORD** y **EXPORTAR EXCEL** generan el informe sin necesitar Office instalado:

- **Word (.docx):** encabezado con archivo, láminas y fecha; resumen en cifras (observaciones, prioridad alta, pendientes, medidas); una ficha por observación con el recorte de la lámina donde se ve la marca dibujada; tabla de medidas al final.
- **Excel (.xlsx):** dos hojas, Observaciones y Medidas, con encabezado fijo, filtros, colores por prioridad, filas alternas y columnas ajustadas. Está configurado para imprimir en A4 horizontal.

En PDFs protegidos el Word sale sin recortes, porque no se dibuja nada sobre un documento protegido. El PDF original no se modifica: las marcas se dibujan en memoria solo para el recorte.

### Atajos del modo planos

| Herramienta | Tecla |
|---|---|
| Navegar | V |
| Hilo · Arco · Ángulo · Área · Calibrar | L · R · G · A · K |
| Nube · Lidar · Marco · Círculo · Flecha · Telaraña | N · D · H · C · F · W |
| Cancelar / soltar selección | Esc |
| Borrar la marca seleccionada | Supr |
| Deshacer | Ctrl+Z |
| Cerrar un área | doble clic o Enter |

## Compilar en Windows: un solo .exe portable

1. Instala Rust desde <https://rustup.rs> (toolchain MSVC, la opción por defecto).
2. Doble clic en `compilar_y_ejecutar.bat` (o `./build.ps1` en PowerShell).
   El script descarga `pdfium.dll` en `vendor/`, compila y deja **`dist/PDFCrawler.exe`**.
3. Ese único archivo es la aplicación: cópialo a cualquier PC con Windows 10/11 de 64 bits
   (USB, red, correo) y ejecútalo, sin instalar nada. También acepta un PDF como argumento o
   arrastrado a la ventana.

Cómo funciona:
- `build.rs` incrusta `vendor/pdfium.dll` dentro del ejecutable. La primera vez que se abre, la
  app la extrae a `%LOCALAPPDATA%\PDFCrawler\pdfium-<huella>\` y la reutiliza en adelante (la
  huella cambia si actualizas PDFium, así nunca se mezclan versiones).
- `.cargo/config.toml` enlaza el runtime de C de forma estática: no necesita Visual C++
  Redistributable.
- Si junto al .exe hay un `pdfium.dll`, se usa ese (útil para probar otra versión).
- Sin `vendor/pdfium.dll` el proyecto compila igual y busca la librería junto al ejecutable.

En Linux/macOS: coloca `libpdfium.so` / `libpdfium.dylib` (de
<https://github.com/bblanchon/pdfium-binaries/releases>) en `vendor/` y `cargo build --release`.

## Icono

`assets/pdf_crawler.ico` trae los tamaños de 16 a 256 px. Los de 16 a 32 px usan una versión con trazos más gruesos para que se lean bien en la barra de tareas. Al compilar se incrusta en el `.exe`, junto con el nombre del producto que muestra el Explorador. La ventana usa `assets/icono_256.png`. El diseño original está en `assets/icono.svg`.

## Atajos

| Acción | Tecla / gesto |
|---|---|
| Desplazar | rueda, arrastrar, flechas, PageUp/PageDown, Espacio |
| Inicio / fin | Home / End |
| Zoom | Ctrl + rueda, pellizco, − / +, escribir el % en el campo, menú ▾ |
| Zoom 100 % (tamaño real) | Ctrl+0 |
| Ajustar al ancho / página completa | Ctrl+1 / Ctrl+2 |
| Saltar | clic en el minimapa o en un hallazgo |
| Cerrar «Acerca de» | Esc o clic |

## Sonido

Todo el sonido se sintetiza en tiempo real (`src/audio.rs`), sin archivos: pasos de cristal en
escala pentatónica, soplidos cuando una cifra vuela, campanas FM que forman una melodía al
atrapar cifras, arpa Karplus-Strong al tejer y un acorde arpegiado al capturar, todo con
reverberación. Se activa o silencia con el botón SONIDO. Para escuchar todos los efectos sin
abrir la app: `cargo test demo_wav -- --ignored` (genera `sonidos_demo.wav`).

## Tema claro

Botón CLARO en la barra superior (o `PDFC_LIGHT=1` para iniciar en claro).

## Arquitectura

```
src/
  main.rs     ventana (eframe + wgpu: DirectX 12 / Vulkan / Metal)
  engine.rs   hilo dedicado de PDFium: abrir, franjas y mosaicos, palabras, geometría, guardar
  doc.rs      maquetación, líneas de lectura, búsqueda de apoyo, clasificación de cifras
  spider.rs   araña: IK de 2 segmentos, marcha alternada, dibujo de la gema
  theme.rs    paleta y primitivas GPU (mallas con degradados, segmentos afilados)
  app.rs      visor virtual, caché de texturas, efectos, tejido, HUD, panel, botones de telaraña
  audio.rs    sintetizador procedural (campanas FM, arpa, cristal, reverberación)
  plan.rs     modo planos: vista 2D, trazos vivos, medidas, marcas, panel y ayudas flotantes
  vecs.rs     índice de geometría: trazos por continuidad, rejilla CSR, ajuste a puntos
  sheet.rs    lectura de la lámina (vistas, escalas, acero, niveles, ejes) y detección de planos
```

Rendimiento con PDFs muy grandes (probado con 800 páginas: abre al instante, ~190 MB de RAM
estables durante el scroll y ~14 MB de VRAM):

- **Nada bloquea la UI.** PDFium vive en su propio hilo; la UI publica en cada frame una cola
  priorizada (lo más cercano al centro de la vista primero) y el hilo descarta lo que ya no se ve.
- **Solo se rasteriza lo visible**, en franjas de 1024 px por página, con escalas agrupadas
  (2^(n/4)) para no re-rasterizar en cada paso de zoom. Mientras llega la franja nítida se muestra
  la de otra escala.
- **Caché LRU de texturas** con un presupuesto de 768 MB de VRAM (`TEX_BUDGET` en `app.rs`).
- **Texto bajo demanda:** solo se extraen las palabras de las páginas visibles ±2, con las cajas
  exactas de cada carácter que da PDFium.
- **Abrir es instantáneo:** al abrir solo se leen los tamaños de página.
- **Cero conversión de píxeles en la UI:** PDFium escribe RGBA directo y la imagen se arma en
  su hilo; el hilo de la UI solo la sube a la GPU (con mipmaps para que el texto se vea nítido
  al alejar el zoom).
- **Precarga:** se rasteriza una pantalla extra arriba y abajo para que el scroll rápido no
  muestre páginas en blanco. Durante una animación de zoom se pide ya la resolución final.
- **Hallazgos en orden de lectura:** las marcas visibles y la cifra a tejer se buscan con
  búsqueda binaria; la lista del panel se cachea. Escala a decenas de miles de cifras.
- **Repintado adaptativo:** 60+ FPS mientras hay movimiento, ~30 FPS en reposo.

## Modos de prueba

- `PDFC_DEMO=20` hace scroll automático durante 20 s (útil para grabar un video).
- `PDFC_ABOUT=1` abre «Acerca de» al iniciar.

## Licencias de terceros

PDFium (BSD-3-Clause / Apache-2.0), egui/eframe (MIT/Apache-2.0), wgpu (MIT/Apache-2.0),
pdfium-render (MIT/Apache-2.0). Todas permiten distribuir el programa gratis o de forma comercial.
