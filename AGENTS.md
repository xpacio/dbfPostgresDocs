# Instrucciones — Repositorio de documentación

> Este archivo gobierna **este** repositorio. Su gemelo (el repo padre, donde vive el
> código) está en `dbfPostgres/AGENTS.md`.

---

## ⚠️ Regla OBLIGATORIA: la documentación sigue al código

**Este repositorio es público.** Publica la documentación de **uso** de la plataforma, para
que las áreas de la empresa la consuman. **No** debe revelar cómo funciona por dentro.

### Cuando cambia el código, esta doc se actualiza **en el mismo momento**

El repo padre (`dbfPostgres`) tiene esta carpeta como **submódulo**. El flujo correcto:

```bash
# 1) Cambió el comportamiento de la API → actualizar la doc pública ACÁ
cd /root/dbfPostgresDocs        # o dbfPostgres/doc_public/, es el mismo repo
#    editar guia-consumo.md / plataforma.md / probar.html
git commit -am "docs: <qué cambió>" && git push

# 2) Que el repo padre registre QUÉ versión de la doc le corresponde a ese código
cd /var/www/dbfPostgres
git add doc_public
git commit -m "docs: bump doc pública a <hash corto>"
```

**Nunca** dejes los dos repos desincronizados: si un commit del padre cambia el
comportamiento de la API, el mismo día tiene que existir el commit correspondiente aquí.

### Qué obliga a actualizar qué

| Cambio en el código | Actualizar aquí |
|---|---|
| Nueva versión de la API, o cambio en la autenticación | `guia-consumo.md` (§2) |
| Cambio en cuotas, caché o límites | `guia-consumo.md` (§3) |
| Cambio en códigos de error | `guia-consumo.md` (§3) |
| Nuevo beneficio, nueva capacidad o nuevo alcance | `plataforma.md` |
| Cambio en cómo se prueba la credencial | `probar.html` |
| Cualquier otra cosa | Preguntarse: *¿esto cambia lo que un consumidor necesita saber?* |

---

## Reglas de contenido (es un repo PÚBLICO)

**Verificado con `grep` antes de cada push.** Lo que **NO** puede aparecer:

| Prohibido | Ejemplo |
|---|---|
| Nombres de tablas o del modelo de datos | ✗ `nota_impor`, `core_canota`, `_suc` |
| Nombres de columnas internas | ✗ cualquier campo de la base |
| Detalles del mecanismo | ✗ cómo se compara, cómo se transfiere, cómo se almacena, particiones |
| Nombres de tecnología o infraestructura | ✗ el motor de base de datos, el servidor web, el lenguaje |
| Credenciales, direcciones internas o rutas | ✗ cualquier `key`, IP privada o ruta de servidor |
| Nombres de personas o áreas específicas | ✗ (salvo que sea un contacto oficial ya publicado) |
| Procesos de operación interna | ✗ despliegues, respaldos, incidencias |

**Lo que SÍ puede aparecer:**

- Las **direcciones de la API** (`/api/v2/...`) y los **nombres de los indicadores** — son
  la **interfaz**, no el mecanismo. Sin ellos la guía de consumo no sirve.
- Los **códigos de error** (`401`, `429`…) y qué hacer con cada uno.
- Las **capacidades** de la plataforma: qué se puede consultar y para qué.

> **La línea es esta:** decir *"puedes pedir las notas de una tienda por rango de fechas"*
> ✔. Decir *"se consulta la tabla de notas comparando un identificador por registro"* ✗.

### Verificación antes de cada push

```bash
# Desde la raíz de este repo, debe salir VACÍO:
grep -rinE 'DBF|ETL|core_|partici|postgres|endpoin|totp|rbfid|_suc|xxh3|\bSQL\b|COPY ' . \
  --include='*.md' --include='*.html' | grep -v '^\./\.git/'
```

---

## Estructura

```
.
├── README.md          entrada: por dónde empezar
├── plataforma.md      qué es y qué aporta (dirección + sistemas)
├── guia-consumo.md    cómo consumir la API (para quien desarrolla)
└── probar.html        página autocontenida para validar la credencial
```

---

## Qué NO es este repositorio

- **No** es el repo del código. Ese es privado y vive aparte.
- **No** es la documentación técnica profunda. Esa también vive en el repo padre, en `docs/`.
- **No** lleva credenciales. Nunca. Ni de ejemplo.

---

*Si algo de esta documentación no coincide con lo que la API responde, es un error que hay
que corregir: la documentación se verifica contra el servicio real.*
