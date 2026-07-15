# Colirios antiglaucomatosos — Meixoeiro (CHUVI)

Web estática de **consulta rápida** de colirios antiglaucomatosos para el Servicio
de Oftalmología del Hospital Meixoeiro (CHUVI, Vigo). Pensada para usarse en
consulta, con el paciente delante, casi siempre desde el móvil.

> «El paciente dice que usa Vizitrav — ¿qué principio activo es, lleva conservante
> y lo va a encontrar en la farmacia?» → respuesta en menos de 3 segundos.

- **Sin login, sin build, sin frameworks.** Un único archivo: `index.html`.
- Los datos se leen de un **Google Sheet publicado como CSV** en cada carga.
- Si no hay red o la hoja se despublica, la web **sigue funcionando** con una copia
  local embebida (avisando de que puede no estar actualizada).

---

## 1. Cómo actualizar los datos (rutina semanal)

Los datos viven en el Google Sheet. **No hace falta tocar el código para cambiar
disponibilidad, precios de conservante, añadir o quitar colirios.**

1. Abre el Google Sheet (la pestaña que está publicada como CSV).
2. Edita lo que necesites respetando **el orden y el nombre de las 7 columnas**:

   | Columna | Valores admitidos |
   |---|---|
   | `Stock` | `Sí` / `No` |
   | `Grupo / clase` | una de las 8 clases (ver más abajo) |
   | `Principio activo` | texto libre |
   | `Nombre comercial (España)` | texto libre |
   | `Presentación` | texto libre (p. ej. `Unidosis`, `Multidosis (Polyquad)`) |
   | `Posología orientativa` | texto libre (p. ej. `1 gota 1×/día (noche)`) |
   | `Conservante` | `Sin` / `Con` / `Ambos` |

3. Guarda. Google Sheets guarda solo. **Los cambios se ven en la web en la
   siguiente carga** (la web pide el CSV con un parámetro anticaché en cada visita,
   así que basta con recargar la página).

### Reglas que conviene respetar

- **`Stock` y `Conservante`:** si dejas la celda vacía o pones un valor raro, la web
  lo trata como **«desconocido»** (gris, con `?`). Nunca lo dará por disponible ni
  por «sin conservante». Es a propósito: mejor un `?` que una afirmación falsa.
- **Clases (`Grupo / clase`):** escríbelas **exactamente** como abajo para que caigan
  en el orden clínico correcto. Una clase escrita distinta (o nueva) no se pierde:
  aparece, pero al final de la lista.

  1. `Análogo de prostaglandina`
  2. `Betabloqueante`
  3. `Agonista alfa-2`
  4. `Inhibidor anhidrasa carbónica`
  5. `Miótico`
  6. `Combinación (PG + timolol)`
  7. `Combinación (otras, x2/día)`
  8. `Combinación con ROCK`

- **Comas dentro de una celda** (p. ej. `Multidosis (pistola, ~3 meses)`): no pasa
  nada, Google las entrecomilla solo al exportar el CSV y la web las parsea bien.

### Recomendable: actualizar también la copia local (fallback)

La web lleva embebida una **copia de seguridad** de los datos para funcionar sin red.
Esa copia **no se actualiza sola**. Si haces un cambio importante (p. ej. un colirio
pasa a agotado de forma prolongada), conviene reflejarlo también en el código para que
el modo sin conexión no muestre datos viejos:

1. En `index.html`, busca `const FALLBACK = [`.
2. Es un array con una fila por colirio, con las mismas claves del CSV
   (`stock`, `clase`, `principio`, `comercial`, `presentacion`, `posologia`, `conservante`).
3. Edítalo para que coincida con la hoja y guarda. (Ver §3 para redeployar.)

> Actualizar el fallback es opcional para cambios menores: el CSV en vivo manda
> siempre que haya conexión. Es solo el «plan B».

---

## 2. Cómo está montado (para quien mantenga el código)

- **`index.html`** — todo: HTML + CSS + JS en un solo archivo, comentado en las zonas
  clave (carga de datos, filtrado, render, impresión). Única dependencia externa:
  [PapaParse](https://www.papaparse.com/) por CDN, para parsear el CSV.
- **`.nojekyll`** — evita que GitHub Pages procese el sitio con Jekyll.
- **`.claude/launch.json`** — solo para previsualizar en local con las herramientas de
  desarrollo; no afecta a producción.

La URL del CSV está en `index.html`, en:

```js
const CONFIG = {
  CSV_URL: "https://docs.google.com/.../pub?output=csv",
  ...
};
```

Si algún día cambias de hoja, actualiza ahí la URL (debe terminar en `output=csv`).

### Cómo se obtiene la URL del CSV (si hay que rehacerla)

En el Google Sheet: **Archivo → Compartir → Publicar en la Web → pestaña concreta →
formato «Valores separados por comas (.csv)» → Publicar**. Copia la URL resultante.

> ⚠️ **Privacidad:** «Publicar en la Web» hace esa pestaña **pública**. Que contenga
> únicamente las 7 columnas de colirios. Nada de datos de pacientes.

---

## 3. Cómo redeployar (GitHub Pages)

El sitio se sirve directamente desde el repositorio con **GitHub Pages**.

- **Cambios solo en la hoja de datos:** no hay que redeployar nada. Se reflejan al
  recargar la web.
- **Cambios en `index.html`** (diseño, fallback, URL del CSV): haz commit y push a la
  rama publicada. GitHub Pages redespliega solo en 1–2 minutos.

  ```bash
  git add index.html
  git commit -m "Actualiza fallback / diseño"
  git push
  ```

### Configuración de Pages (solo la primera vez)

En el repositorio de GitHub: **Settings → Pages → Build and deployment → Source:
«Deploy from a branch» → rama `main` (o la que uses) / carpeta `/ (root)`**. Guarda.
La URL pública aparece ahí en un par de minutos.

---

## 4. Probar en local antes de publicar

Desde la carpeta del proyecto:

```bash
python3 -m http.server 8000
```

Y abre <http://localhost:8000>. (Hace falta servirlo por HTTP, no abrir el archivo con
`file://`, porque el navegador bloquea el `fetch` del CSV en ese modo. Si estás sin red,
verás el aviso «Datos locales» y la copia embebida — es el comportamiento esperado.)

---

## 5. Aviso clínico

La información es de **consulta rápida**. La posología es **orientativa** y **no
sustituye a la ficha técnica**. La disponibilidad en farmacia se actualiza **manualmente**
y puede no reflejar la situación real en tiempo real.
