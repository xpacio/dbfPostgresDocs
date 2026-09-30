# Guía de consumo de la API

> Para quien va a **construir** sobre los datos: tableros, scripts, automatizaciones o
> asistentes de IA.
> Si solo quieres entender qué aporta la plataforma, lee [`plataforma.md`](plataforma.md).

**Esta guía es un punto de partida, no un techo.** La API cubre más de lo que aquí se
documenta: hay decenas de temas consultables y formas de filtrar, ordenar y agrupar que no
están todas en los ejemplos. Si encuentras un muro:

- **Algo no funciona como dice la guía** → repórtalo (ver [§8](#8-algo-no-coincide)). Un
  ejemplo roto es un error que hay que corregir, no algo con lo que tengas que lidiar.
- **Necesitas algo que la API no hace hoy** → pídelo. Filtros nuevos, un indicador que no
  existe, otra forma de agrupar, más rango: se solicita y se evalúa. **La API crece con lo
  que las áreas necesitan**, no al revés.
- **Necesitas más permisos o más cuota** → también se solicita; va con tu perfil.

No te quedes con un "no se puede" sin preguntar: casi siempre hay una vía, o se puede
construir.

---

## 1. Lo que necesitas para empezar

| Cosa | De dónde |
|---|---|
| **Una credencial** (una cadena secreta que identifica a tu área) | Te la entrega el área que administra la plataforma |
| **Los permisos de tu credencial** | Se definen al entregártela: qué información ves y de qué sucursales |
| **La dirección del servicio** | Viene con tu credencial |

⚠️ **Tu credencial es personal y secreta.** No la subas a GitHub, no la compartas por chat y
no la pongas en una página web pública. Si se filtra, se revoca y se emite otra al instante.

---

## 2. Dos formas de consumirla, elige la tuya

### Si tu desarrollo es una **página web** (tablero, dashboard, página de escritorio)

Usas la vía directa: cada consulta lleva tu credencial en una cabecera.

```js
const r = await fetch(
  'https://<servicio>/api/v2/kpis/auditoria_canota?plaza=penla&desde=2026-06-01&hasta=2026-08-30',
  { headers: { 'X-Api-Key': TU_CREDENCIAL } }
);
const datos = await r.json();
console.log(datos.rows.length, datos.meta.generated_at);
```

Listo: funciona desde un servidor propio, desde GitHub Pages, o incluso abriendo el archivo
con doble clic. La dirección responde el permiso de origen automáticamente.

**Si la información es sensible**, pon un pequeño intermediario en tu servidor que guarde
la credencial y le devuelva a tu página solo el resultado. Tu página nunca toca el secreto.

### Si tu desarrollo corre en un **servidor** (Python, C#, automatización, IA)

Usa la vía con sesión: primero pides una sesión con tu credencial, y luego consultas con esa
sesión (que caduca sola).

```python
import requests

token = requests.post(
    'https://<servicio>/api/v1/auth',
    headers={'X-Api-Key': TU_CREDENCIAL},
).json()['token']

# Cada consulta usa la sesión + un código que cambia cada 100 segundos (te lo damos con
# la credencial, junto con un ejemplo listo para tu lenguaje).
```

Esta vía es la correcta cuando la credencial no puede salir de tu servidor.

---

## 3. Recetas: las preguntas de negocio, resueltas

Cada receta es una consulta que **ya funciona**. Copia, pega, cambia las fechas y tendrás
datos de tu operación. Usan la vía directa (la credencial en la cabecera).

Todas asumen estas dos líneas al inicio:

```js
const BASE = 'https://dbfpostgres.servicios.care';
const KEY  = TU_CREDENCIAL;                        // no la escribas en el código
const pedir = (ruta) => fetch(BASE + ruta, { headers: { 'X-Api-Key': KEY } }).then((r) => r.json());
```

> **Ten en cuenta:** las fechas van en `YYYY-MM-DD`, y el rango por consulta tiene un
> máximo (te lo dice el catálogo al autenticarte). La **primera** consulta de un rango
> grande tarda unos segundos; **las siguientes del mismo rango son inmediatas**.

---

### 1. ¿Qué tienda vendió más ayer? ¿Y la que menos?

Trae el resumen del día y ordénalo por sucursal. La respuesta incluye un bloque
`por_sucursal` con el conteo de cada tienda.

```js
const ayer = new Date(Date.now() - 864e5).toISOString().slice(0, 10);
const d = await pedir(`/api/v2/canota?desde=${ayer}&hasta=${ayer}`);

const ranking = [...d.por_sucursal].sort((a, b) => b.registros - a.registros);
console.log('Más vendió:', ranking[0]);          // { sucursal, registros }
console.log('Menos vendió:', ranking.at(-1));
```

*A quién sirve: dirección · dirección, compras*
*Si quieres el importe y no solo el conteo, el total del rango está en `d.kpis`.*

---

### 2. ¿Cómo se compara una plaza contra otra en los últimos 90 días?

Las plazas son **`penla`, `bajac`, `xalap`, `hermo`, `nicar`**. Pides la misma consulta
cambiando `plaza`.

```js
const fin = new Date().toISOString().slice(0, 10);
const ini = new Date(Date.now() - 90 * 864e5).toISOString().slice(0, 10);

const penla = await pedir(`/api/v2/canota?desde=${ini}&hasta=${fin}&plaza=penla`);
const bajac = await pedir(`/api/v2/canota?desde=${ini}&hasta=${fin}&plaza=bajac`);

console.log('penla', penla.kpis.nota_impor, '|', penla.rbfids.length, 'sucursales');
console.log('bajac', bajac.kpis.nota_impor, '|', bajac.rbfids.length, 'sucursales');
```

*A quién sirve: dirección, compras*
*`plaza=` acepta varias separadas por coma (`plaza=penla,xalap`). Sin `plaza`, obtienes
toda la operación.*

---

### 3. ¿Qué notas de esta tienda tienen un monto inusualmente alto?

El indicador de auditoría devuelve el conteo y el impuesto **por tienda y por día**. Para
buscar los picos, ordénalo.

```js
const d = await pedir('/api/v2/kpis/auditoria_canota?plaza=penla&desde=2026-09-01&hasta=2026-09-29');

const picos = [...d.rows].sort((a, b) => b.total_impto - a.total_impto).slice(0, 5);
console.table(picos);   // { cplaza, ctienda, nota_fecha, notas, total_impto }
```

*A quién sirve: auditoría*
*Para el detalle de tickets (partidas), usa `auditoria_cunota` del mismo modo.*

---

### 4. ¿Qué producto lleva meses sin venderse en ninguna tienda?

La comparación es entre **lo vendido** y **el catálogo del cedis** (`lista`), que es la
fuente de productos y precios de la operación.

> **Cómo funciona `lista`:** trae los productos **según su fecha de modificación**, es decir
> los que el área de precios **tocó** dentro del rango que pidas. No es una foto del
> catálogo completo: es "qué cambió". Por eso, para esta pregunta, **usa un rango amplio**
> (hasta el máximo que te permita tu perfil).

```js
// 1) qué se vendió en los últimos 90 días
const d = await pedir('/api/v2/partvta/agregate?cols=clave_art&sumas=imppar&desde=2026-07-01&hasta=2026-09-29');
const vendidos = new Set(d.filas.map((f) => f.clave_art));

// 2) el catálogo del cedis (rango amplio: trae lo modificado en ese periodo)
const catalogo = await pedir('/api/v2/lista/query?cols=clave,prod_descr&desde=2026-07-01&hasta=2026-09-29&limite=500');
//    pagina con &offset=500, 1000, ... si necesitas más

const sinVenta = catalogo.registros
  .filter((r) => !vendidos.has(r.clave))
  .map((r) => ({ clave: r.clave, descripcion: r.prod_descr }));

console.log(`${sinVenta.length} productos del catálogo sin una sola venta en el periodo`);
console.table(sinVenta.slice(0, 20));
```

*A quién sirve: inventarios, compras*
*`lista` es **del cedis** (las tiendas tienen su propia copia, que hoy no está expuesta en
la API). El campo `prod_descr` es la descripción y `clave` el código del producto.
**La existencia no está aquí**: vive en el movimiento de inventario (receta 5).*

---

### 5. ¿Dónde hay existencia de más y dónde falta, para mover entre tiendas?

El movimiento de inventario da el total; para comparar **por zona**, pide una consulta por
plaza.

```js
// el total de la operación
const d = await pedir('/api/v2/movsinv?desde=2026-09-01&hasta=2026-09-29');
console.log('existencia total:', d.kpis.cantidad, '· valuada en', d.kpis.costo);

// comparación por plaza (una consulta por plaza)
for (const plaza of ['penla', 'bajac', 'xalap', 'hermo', 'nicar']) {
  const p = await pedir(`/api/v2/movsinv?desde=2026-09-01&hasta=2026-09-29&plaza=${plaza}`);
  console.log(plaza, '->', p.kpis.cantidad);
}
```

*A quién sirve: inventarios*
*El desglose **por tienda** se pide una vez por tienda con `&rbfids=<tienda>`. No hay una
consulta que agrupe por sucursal, así que empieza por plaza (5 llamadas) y baja al detalle
solo donde haga falta.*

---

### 6. ¿Cómo se comporta un producto por día en las últimas semanas?

Filtra por el artículo y pide la agrupación diaria.

```js
const d = await pedir(
  '/api/v2/partvta/agregate?sumas=imppar,cantidad&tipo=daily' +
  '&desde=2026-09-01&hasta=2026-09-10&rbfids=roton' +
  '&filtro=' + encodeURIComponent('clave_art = H023910')
);

d.filas.forEach((f) => console.log(f.periodo, f.sum_imppar, f.sum_cantidad));
```

*A quién sirve: compras*
*`tipo` acepta `daily`, `weekly` o `monthly`. Cambia `clave_art` por el que te interese.*

---

### 7. ¿Qué tienda se sale del patrón del resto esta semana?

No hay una consulta de "anomalías": se calcula sobre el ranking. Son cinco líneas.

```js
const d = await pedir('/api/v2/canota?desde=2026-09-22&hasta=2026-09-29');

const vals = d.por_sucursal.map((r) => Number(r.registros));
const avg  = vals.reduce((a, b) => a + b, 0) / vals.length;
const sd   = Math.sqrt(vals.reduce((a, v) => a + (v - avg) ** 2, 0) / vals.length);

const fuera = d.por_sucursal.filter((r) => Math.abs((r.registros - avg) / sd) >= 2);
console.log('fuera del patrón:', fuera);      // p.ej. roton, con z = 2.9
```

*A quién sirve: dirección, auditoría*
*Ajusta el umbral (`>= 2`) según qué tan estricto quieras ser.*

---

### 8. ¿Qué se vendió un día específico del mes pasado, en detalle?

El detalle de partidas está en `cunota`. Filtra por la nota o por el rango del día.

```js
// (a) una nota concreta
const nota = await pedir('/api/v2/cunota/query?cols=nota_folio,prod_clave,nota_canti,nota_preci'
  + '&filtro=' + encodeURIComponent('nota_folio = 150090') + '&limite=50');

// (b) todo lo vendido en una fecha
//     OJO: si no pasas desde/hasta, la consulta mira solo los últimos 7 días.
//     Pasa el rango que incluya tu fecha.
const dia = await pedir('/api/v2/canota/query?cols=nota_folio,nota_fecha,clie_clave,nota_impor'
  + '&desde=2026-09-15&hasta=2026-09-15'
  + '&filtro=' + encodeURIComponent('nota_fecha = 2026-09-15') + '&limite=500');

console.table(nota.registros);
console.table(dia.registros);
```

*A quién sirve: auditoría*
*`limite` llega hasta 500; para más, pagina con `offset`. **Regla general:** siempre pasa
`desde`/`hasta` explícitos, para no depender del rango por defecto.*

---

## 4. Filtrar, ordenar y agrupar

Estas tres piezas se combinan con cualquier consulta, y son las que convierten una consulta
en la respuesta que necesitas.

### 4.1 Filtrar (`filtro`)

Se usa en `query` y `agregate`. Son condiciones separadas por `;`, cada una
`campo operador valor`:

```
filtro=clave_art = H023910
filtro=nota_impor >= 1000;clie_clave <> ''
filtro=prod_descr LIKE %INTER%
```

| Operador | Significa |
|---|---|
| `=` | igual |
| `<>` o `!=` | distinto |
| `<` / `>` | menor / mayor |
| `<=` / `>=` | menor o igual / mayor o igual |
| `LIKE` | busca texto — **necesita los comodines `%`** |

⚠️ **Tres cuidados al filtrar:**

1. **`LIKE` con `%`.** Para "contiene", rodea el término: `LIKE %INTER%` (18 resultados).
   **Sin los `%` busca coincidencia exacta** y te devuelve 0 (`LIKE INTER` → 0). Usa
   `LIKE INTER%` para "empieza con" y `LIKE %INTER` para "termina con".
2. **Espacios siempre** (`campo = valor`). Si el operador está mal escrito, el servicio puede
   tomarlo como parte del valor y devolverte un resultado **silenciosamente equivocado**.
3. **El filtro de fecha sigue aplicando.** Toda consulta mira un rango (`desde`/`hasta`), así
   que un filtro por texto sobre un rango que no lo contiene devuelve `total: 0`. Si buscas
   por descripción y sale vacío, **amplía el rango** antes de dudar del filtro.

### 4.2 Ordenar y paginar (`query`)

| Parámetro | Para qué | Límite |
|---|---|---|
| `orden` | campo por el que ordenar | debe existir |
| `dir` | `asc` o `desc` | — |
| `limite` | cuántos registros traer | **máximo 500** |
| `offset` | desde cuál empezar (para paginar) | — |

### 4.3 Agrupar y sumar (`agregate`)

| Parámetro | Para qué | Límite |
|---|---|---|
| `cols` | campos por los que agrupar (csv) | deben existir |
| `sumas` | campos numéricos a sumar (csv) | **máximo 8** |
| `tipo` | además, agrupar por periodo: `daily`, `weekly`, `monthly` | — |

Devuelve `filas[]` con un grupo por fila y `resumen` con los totales.

### 4.4 Fechas: siempre explícitas

Todas las consultas aceptan `desde` y `hasta` (`YYYY-MM-DD`). **Pásalos siempre**:

- Si los omites, el rango por defecto son los **últimos 7 días**, y tu filtro por una fecha
  anterior devolverá `total: 0` sin explicar por qué.
- El rango máximo lo fija tu perfil (te lo dice el catálogo al autenticarte).

### 4.5 Qué puedes consultar (el catálogo)

La API **se describe a sí misma**. Con una sola consulta obtienes:

```js
const cat = await pedir('/api/v2/meta');
// cat.dominios → [{ domain, columna_fecha, ejemplo }, ...]
// cat.endpoints, cat.filtro_where, cat.cuota, cat.max_days
console.log(`${cat.dominios.length} temas disponibles`);
console.table(cat.dominios.slice(0, 10));
```

**Esto es lo primero que conviene mirar** al empezar: te dice qué temas hay, cuál es su
campo de fecha y un ejemplo de consulta. Y si un campo te da error, `explore` te lista los
campos reales de ese tema:

```js
const campos = await pedir('/api/v2/<tema>/explore');
console.log(campos.columnas.map((c) => c.name));
```

---

## 5. Preguntas frecuentes (las que de verdad se hacen)

| Situación | Qué hacer |
|---|---|
| **Me responde `401`** | Tu credencial no es válida o la sesión caducó. Vuelve a autenticarte |
| **Me responde `429`** | Superaste tu cuota de consultas. Espera lo que indica la respuesta. **Las consultas repetidas no consumen cuota**: si pides lo mismo enseguida, el sistema te devuelve el resultado ya calculado |
| **Me responde `403`** | Estás consultando algo fuera de tus permisos (una sucursal o un tema que no te toca). Pide ampliación si lo necesitas |
| **Me responde `400`** | El rango de fechas está mal, o pediste un campo que no existe |
| **Tarda mucho la primera vez** | La primera consulta de un rango grande calcula; **las siguientes del mismo rango son inmediatas** (el resultado queda disponible un rato) |
| **Quiero un rango muy largo** | Se consulta por ventanas. Divide el periodo y une los resultados |
| **No sé qué información hay disponible** | Consulta el catálogo del servicio: se describe a sí mismo (qué temas, qué campos y un ejemplo) |
| **No sé qué puede responder** | Mira las preguntas de negocio al final de [`plataforma.md`](plataforma.md): todas se pueden responder hoy |

---

## 6. Buenas prácticas

1. **No pidas todo.** Filtra por sucursal y por rango; cuanto más acotada la consulta, más
   rápido responde para todos.
2. **Aprovecha que se repite.** La misma consulta dentro de un rato **no** cuesta cuota.
3. **Serializa las consultas pesadas.** Si vas a pedir varias, mejor una tras otra que todas
   a la vez.
4. **Revisa la fecha del dato.** Cada respuesta indica cuándo se calculó, para que sepas si
   necesitas refrescar.
5. **No guardes la credencial en el código.** Usa variables de entorno.
6. **Guarda la fecha del dato junto a tus resultados.** Así, si alguien cuestiona una cifra,
   sabes de cuándo era.

---

## 7. Cómo saber si algo cambió

Cada respuesta trae la fecha y hora en que se calculó el dato, y si vino del cálculo o de un
resultado ya guardado. Cuando publiquemos cambios en la API, se avisan en este repositorio.

---

## 8. ¿Algo no coincide?

Si la API responde distinto a lo que dice esta guía, **es un error que hay que corregir**.
Abre un *issue* con:

- La consulta exacta (sin tu credencial).
- Lo que esperabas.
- Lo que recibiste.

---

*Guía de consumo. Para el detalle de qué es la plataforma y qué aporta, ver
[`plataforma.md`](plataforma.md).*
