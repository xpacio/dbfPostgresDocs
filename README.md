# Documentación de la Plataforma — dbfPostgres

Bienvenido. Este repositorio reúne la documentación **de uso** de la plataforma de
consolidación de información: qué es, qué aporta y cómo consumir sus datos desde tus
propios desarrollos.

> **Este repositorio es para consumir, no para operar.** No contiene código, ni
> credenciales, ni la operación interna. Si necesitas una credencial, pídela al área que
> administra la plataforma.

---

## Por dónde empezar

| Si quieres… | Lee |
|---|---|
| **Entender qué es y qué aporta** (para dirección o para tu área) | [`plataforma.md`](plataforma.md) |
| **Ver las preguntas que ya se pueden responder** | [`plataforma.md`](plataforma.md#preguntas-que-cualquier-persona-puede-hacerle) — cada una enlaza a su receta |
| **Consumir la API desde tu desarrollo** | [`guia-consumo.md`](guia-consumo.md) |
| **Copiar una consulta que ya funciona** | [`guia-consumo.md`](guia-consumo.md#3-recetas-las-preguntas-de-negocio-resueltas) — 8 recetas listas |
| **Probar que tu credencial funciona**, sin instalar nada | [`probar.html`](probar.html) |

---

## Lo esencial, en 30 segundos

**La plataforma convierte cada punto de venta en una fuente de consulta casi inmediata.**
En lugar de esperar el reporte del día siguiente, consultas el dato cuando lo necesitas, y
todas las áreas ven la misma cifra.

Tienes dos formas de consumirla:

| Vía | Para quién | Cómo |
|---|---|---|
| **Directa desde una página web** | Tableros, páginas de escritorio | Una sola cabecera de autenticación; se monta y funciona |
| **Desde un programa en servidor** | Python, C#, automatizaciones, asistentes de IA | Credenciales que nunca salen de tu servidor |

El detalle está en [`guia-consumo.md`](guia-consumo.md).

---

## Prueba rápida

Abre [`probar.html`](probar.html) (o descárgalo y ábrelo con doble clic) y pega tu
credencial. Verifica en unos segundos que puedes consultar y que los permisos de tu perfil
son los que esperas.

---

## Cómo está organizado

```
.
├── README.md          ← este archivo
├── plataforma.md      ← qué es y qué aporta (audiencia mixta: dirección y sistemas)
├── guia-consumo.md    ← cómo consumir la API (para quien desarrolla)
└── probar.html        ← página de prueba, autocontenida
```

---

## ¿Falta algo o algo no coincide?

Abre un *issue* en este repositorio. Si detectas que la documentación no coincide con lo que
la API responde, **es un error que hay que corregir**: la documentación se verifica contra
el servicio real.

---

*Documentación de uso. No incluye detalles de implementación interna.*
