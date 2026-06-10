# Bolsa de Uniformes — Especificaciones del Producto

> Documento de traspaso. Léelo si vas a mantener, editar o desplegar esta página.
> Última revisión: 2026-06-09

---

## 1. Qué es

Página web de una sola pantalla (single-page) para el **Colegio Ciudad de México, Plantel Polanco**.
Muestra el inventario **en vivo** de la "Bolsa de Uniformes" — uniformes escolares donados que
otras familias pueden llevarse gratis. También comunica el mensaje de **economía circular**
(reusar / reciclar / apoyar a la comunidad).

- **Idioma:** Español (México).
- **Público:** Familias del colegio.
- **Modelo:** Donación / toma libre. No hay carrito, pago ni login. *"Puedes llevarte prendas sin que dones."*

---

## 2. Stack técnico

| Pieza | Detalle |
|---|---|
| Tipo | HTML + CSS + JavaScript **vanilla**, todo en un solo archivo (`index.html`). |
| Dependencias | **Ninguna** — no hay framework, build, npm ni node_modules. |
| Datos | Google Sheet leído en vivo vía endpoint `gviz` (CSV). |
| Proxy CORS | `corsproxy.io` (necesario para que el navegador pueda leer el Sheet). |
| Analytics | Google Analytics 4 (gtag.js), ID `G-MCLHFG33FF`. |
| Imágenes | Carpeta local `imágenes/` (nombre con acento — ojo al desplegar). |

No hay paso de compilación: **se edita el HTML y se sube tal cual.**

---

## 3. Estructura de archivos

```
ccm uniformes/
├── index.html          ← TODO el sitio (HTML + CSS + JS en un archivo)
├── PRODUCT_SPECS.md     ← este documento
└── imágenes/
    ├── Logo ccm horizontal blanco.png   ← logo de la barra superior
    ├── logo-colegio-ciudad-de-mexico-polanco-opt1.webp  ← (sin usar actualmente)
    ├── X.png                            ← imagen fallback si una foto no carga
    ├── Chaleco-todos.png
    ├── Chamarra-deportiva.png
    ├── Chamarra-invierno.png
    ├── Hoodie.png
    ├── Jumper.png
    ├── Pantalon-deportivo.png
    ├── Pantalon-vestir.png
    ├── Playera-cuelloV.png
    ├── Polo.png                         ← usada por "Playera" y "Playera polo"
    ├── Polo-manga-larga.png
    ├── Shorts-todos.png
    ├── Sueter-abierto.png
    ├── Sueter-cerrado.png
    ├── Sueter-universitario.png
    ├── bata.jpg                         ← usada por "Bata de laboratorio"
    └── bata.webp                        ← (sin usar actualmente)
```

---

## 4. De dónde vienen los datos (lo más importante)

El inventario **NO está en el código**. Vive en un Google Sheet y se actualiza en vivo.

- **Spreadsheet ID:** `1tzWm-bLqIZKHxMF6pgcpankSBhZBqIO10RtaGyfiV1o`
- **Hoja (tab):** `INVENTORY_CURRENT`
- **URL que consume la página:**
  ```
  https://corsproxy.io/?url=<encoded>
    https://docs.google.com/spreadsheets/d/1tzWm-bLqIZKHxMF6pgcpankSBhZBqIO10RtaGyfiV1o/gviz/tq?tqx=out:csv&sheet=INVENTORY_CURRENT
  ```
- El Sheet debe estar **compartido como "cualquiera con el enlace puede ver"** para que el endpoint público funcione.
- La página **refresca el inventario cada 30 segundos** (`REFRESH_MS`) sin recargar la pantalla.

### Columnas que espera el Sheet
La fila 1 debe tener encabezados (no importan mayúsculas/espacios). Se buscan por nombre:

| Columna | Uso |
|---|---|
| `tipo`  | Slug del uniforme. **Debe coincidir exactamente** con una llave del `CATALOG` (ver §6). |
| `talla` | Talla de la prenda. Si está vacía, se muestra como **"Única"**. |
| `stock` | Cantidad (entero). Sólo se muestran filas con stock > 0. |

- Cada fila = una talla de un tipo de prenda. Filas con el mismo `tipo`+`talla` se suman.
- Un `tipo` que no exista en `CATALOG` se **ignora** (no aparece). Por eso, agregar un uniforme nuevo requiere editar tanto el Sheet **como** el `CATALOG` del código.

---

## 5. Qué muestra la página (secciones)

1. **Barra superior** — logo del colegio + título "Bolsa de uniformes".
2. **Hero** — mensaje de segunda vida + banner "¡Puedes llevarte prendas sin que dones!" + pills de valores.
3. **Stats** — *Piezas disponibles* y *Tallas con stock* (calculadas en vivo del inventario).
4. **Controles** — buscador de texto + filtros + dropdown de talla.
5. **Grid de tarjetas** — una tarjeta por tipo de uniforme con stock; muestra foto, badge "Donado",
   estado (Disponible / Últimas piezas), tallas disponibles con cantidad, y total.
6. **Economía circular** — flujo de 4 pasos (Donas → Clasificamos → Transformamos → Apoyamos),
   4 tarjetas explicativas y un enlace al **catálogo de muñecas** (Issuu).
7. **Footer** — "Plantel Polanco" + contador de "prendas rescatadas del desecho" (= total de piezas).

### Filtros disponibles
| Filtro | Lógica |
|---|---|
| Todos | Todo. |
| Varios en stock | total > 2. |
| Últimas piezas | total ≤ 2 (`LOW_STOCK`). |
| Kinder/Primaria | nivel `pri` o `todos`. |
| Sec/Prepa | nivel `sec` o `todos`. |

**Umbral de "últimas piezas":** constante `LOW_STOCK = 2`. Cambiarla afecta tanto el filtro como el badge naranja de las tarjetas.

---

## 6. Catálogo de uniformes (en el código)

Definido en el objeto `CATALOG` dentro de `index.html`. Mapea el slug del Sheet → nombre visible, imagen, nivel y subtítulo.

| Slug (`tipo` en el Sheet) | Nombre | Imagen | Nivel |
|---|---|---|---|
| `CHALECO-TODOS` | Chaleco | Chaleco-todos.png | todos |
| `CHAMARRA/DEPORTIVA-TODOS` | Chamarra deportiva | Chamarra-deportiva.png | todos |
| `CHAMARRA/INVIERNO-TODOS` | Chamarra de invierno | Chamarra-invierno.png | todos |
| `HOODIE-SEC/PREP` | Hoodie | Hoodie.png | sec |
| `JUMPER-PRI` | Jumper | Jumper.png | pri |
| `PANTALON/DEPORTIVO-TODOS` | Pantalón deportivo | Pantalon-deportivo.png | todos |
| `PANTALON/VESTIR-PRI` | Pantalón de vestir | Pantalon-vestir.png | pri |
| `PLAYERA-SEC/PREP` | Playera | Playera-cuelloV.png | sec |
| `PLAYERAV-SEC/PREP` | Playera V | Playera-cuelloV.png | sec |
| `PLAYERA/POLO` | Playera polo | Polo.png | sec |
| `POLO/LARGA-TODOS` | Polo manga larga | Polo-manga-larga.png | todos |
| `SHORT-TODOS` | Short | Shorts-todos.png | todos |
| `SUETER/ABIERTO-PRI` | Suéter abierto | Sueter-abierto.png | pri |
| `SUETER/CERRADO-PRI` | Suéter cerrado | Sueter-cerrado.png | pri |
| `SUETER/DEPORTIVO-SEC/PREP` | Suéter universitario | Sueter-universitario.png | sec |
| `BATA/LABORATORIO-SEC/PREP` | Bata de laboratorio | bata.jpg | sec |

**Nivel** (`level`) sólo se usa para los filtros: `pri` = Kinder/Primaria, `sec` = Sec/Prepa, `todos` = aparece en ambos.

### Orden de tallas
`SIZE_ORDER = ["CH","M","G","XG"]`. Primero las tallas numéricas (ascendente), luego las letras en ese orden, y "Única" al final.

---

## 7. Tareas comunes de mantenimiento

**Cambiar el inventario / cantidades / tallas**
→ Editar el Google Sheet (`INVENTORY_CURRENT`). No se toca código. El sitio se actualiza solo en ≤30 s.

**Agregar un tipo de uniforme nuevo**
1. Agregar la imagen en `imágenes/`.
2. Agregar una entrada al objeto `CATALOG` en `index.html` (slug, name, img, level, sub).
3. Usar ese mismo slug en la columna `tipo` del Sheet con su stock.

**Cambiar una foto**
→ Reemplazar el archivo en `imágenes/` con el mismo nombre, o cambiar el valor `img` en `CATALOG`.
Si una foto falla al cargar, se muestra `X.png` automáticamente.

**Cambiar textos del hero / economía circular**
→ Editar directamente el HTML (secciones `.hero` y `.circular`).

**Cambiar colores**
→ Variables CSS en `:root` al inicio del `<style>` (`--navy`, `--green`, etc.).

---

## 8. Enlaces externos

- **Catálogo de muñecas (Issuu):** https://issuu.com/ccm_polanco/docs/uniformes_peluches_y_mun_ecas
- **Google Analytics:** propiedad `G-MCLHFG33FF`.

---

## 9. Riesgos y puntos frágiles a vigilar

- **Dependencia de `corsproxy.io`:** es un proxy de terceros gratuito. Si se cae o cambia su política,
  el inventario deja de cargar. La página muestra un mensaje de error y reintenta cada 30 s.
  *Alternativa más robusta a futuro:* publicar el Sheet como CSV y servirlo desde el mismo dominio, o un backend propio.
- **Permisos del Sheet:** si alguien cambia el compartido a privado, el inventario deja de cargar.
- **Slugs:** un typo en la columna `tipo` del Sheet hace que esa fila se ignore silenciosamente.
- **Carpeta `imágenes/` con acento:** algunos servidores/hosts manejan mal los acentos en rutas. Verificar tras desplegar.
- **Todo el estado es público y de sólo lectura** — no hay datos sensibles ni autenticación.

---

## 10. Despliegue

Sitio estático: subir `index.html` + carpeta `imágenes/` a cualquier hosting estático
(GitHub Pages, Netlify, servidor del colegio, etc.). No requiere build ni servidor de aplicación.
El repositorio git ya lleva el historial de cambios.
