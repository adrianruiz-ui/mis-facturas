# Mis facturas del mes

Herramienta personal para saber, mes a mes, **cuánto me falta facturar** y **cuánto saldría a pagar de IVA** bajo RESICO.

Es una sola página HTML. Sin backend, sin cuentas, sin servidor. Todo se procesa y se guarda en el navegador.

## Qué hace

- Arrastras los **XML** de tus CFDI y los clasifica solos:
  - Si el **emisor eres tú**, la factura fija la base del mes (subtotal, IVA trasladado, IVA retenido, ISR retenido).
  - Si es un **gasto**, lo lee y decide si aplica para IVA acreditable.
- Clasifica cada gasto en este orden:
  1. Validaciones duras: que la factura esté a tu nombre; que no sea efectivo por más de $2,000; que traiga IVA desglosado.
  2. La **clave de producto/servicio del SAT** (`ClaveProdServ`) de los conceptos.
  3. El nombre del proveedor y la descripción.
- Cuando corriges una clasificación, guarda la regla por RFC del proveedor y la aplica sola la próxima vez.
- Calcula el ISR del mes con la tabla del art. 113-E y lo compara contra la retención del 1.25%.
- Exporta CSV del mes y respaldo completo en JSON.

## Las reglas fiscales que implementa

| Situación | Efecto |
|---|---|
| Cualquier gasto | **No baja el ISR.** En RESICO el ISR se calcula sobre ingresos cobrados, sin deducciones (art. 113-E LISR). |
| Gasto indispensable para la actividad | IVA acreditable al 100% |
| Comida en restaurante local | IVA acreditable al 8.5% — es la proporción deducible para ISR (art. 28-XX LISR + art. 5-I LIVA) |
| Internet y celular | Proporción de uso, por defecto 50% |
| Efectivo por más de $2,000 | No acredita |
| Factura a nombre de otro | No acredita |
| Médicos, seguros, colegiaturas | No cuentan para IVA; son deducción personal en la anual, contra el ingreso por sueldos |

La meta mensual es `IVA trasladado − IVA retenido`, que con la retención de dos terceras partes equivale a un tercio de lo facturado.

> No es asesoría fiscal. Es una estimación para orientarte. Confirma con tu contador antes de declarar.

## Publicarlo en GitHub Pages

1. Crea un repositorio nuevo (puede ser **privado**; Pages funciona en repos privados con cuenta de pago, si no, usa público — el código no contiene datos tuyos).
2. Sube estos archivos a la raíz del repo: `index.html`, `manifest.webmanifest`, `icon.svg`, `sw.js`, `README.md`.
3. **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main` / `(root)` → Save.**
4. En un minuto queda en `https://<tu-usuario>.github.io/<repo>/`.

En Chrome puedes instalarla como app desde el icono de la barra de direcciones.

No usa service worker: cada carga trae la versión actual del sitio, sin cachés fantasma que sirvan una versión vieja.

## Privacidad

- El repositorio contiene **solo código**. Ningún dato fiscal.
- Tu RFC, tus facturas y tus reglas viven en el `localStorage` de tu navegador, en tu equipo.
- Los XML se leen en memoria; nunca se suben a ningún servidor.
- Si borras los datos del sitio o cambias de navegador, se pierde: baja el **Respaldo** de vez en cuando.

## Desarrollo

No hay build. Es HTML, CSS y JavaScript en un archivo.

```bash
python3 -m http.server 8080
# abre http://localhost:8080
```

Un cambio en `index.html` se ve en la siguiente carga. `sw.js` solo existe para desinstalar el service worker que tuvieron las primeras versiones; se puede borrar cuando ya nadie cargue el sitio desde ese caché.

## Licencia

MIT
