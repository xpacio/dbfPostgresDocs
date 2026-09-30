# Guía de consumo de la API

> Para quien va a **construir** sobre los datos: tableros, scripts, automatizaciones o
> asistentes de IA.
> Si solo quieres entender qué aporta la plataforma, lee [`plataforma.md`](plataforma.md).

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

## 3. Preguntas frecuentes (las que de verdad se hacen)

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

## 4. Buenas prácticas

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

## 5. Cómo saber si algo cambió

Cada respuesta trae la fecha y hora en que se calculó el dato, y si vino del cálculo o de un
resultado ya guardado. Cuando publiquemos cambios en la API, se avisan en este repositorio.

---

## 6. ¿Algo no coincide?

Si la API responde distinto a lo que dice esta guía, **es un error que hay que corregir**.
Abre un *issue* con:

- La consulta exacta (sin tu credencial).
- Lo que esperabas.
- Lo que recibiste.

---

*Guía de consumo. Para el detalle de qué es la plataforma y qué aporta, ver
[`plataforma.md`](plataforma.md).*
