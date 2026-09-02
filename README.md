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
- Lee del XML lo que cambia el resultado: **método de pago** (un PPD no acredita hasta que llega el complemento de pago), **tipo de comprobante** (una nota de crédito resta; un complemento de pago libera los PPD que referencia; nómina y traslado se ignoran), **moneda y tipo de cambio** (un CFDI en dólares se convierte a pesos) y el **UUID**, que es lo que identifica de verdad una factura y evita duplicados.
- Calcula el ISR del mes con la tabla del art. 113-E y lo compara contra la retención del 1.25%.
- Arrastra el IVA a favor de un mes al siguiente, como corresponde.
- Acumula las **deducciones personales del año** (médicos, seguros, colegiaturas, retiro), aplica el tope y estima cuánto valen contra tu ISR de sueldos con la tarifa anual del art. 152.
- Muestra el **año mes por mes**: facturado, IVA a cargo, acreditado, pagado y saldo a favor.
- Lleva el **libro de saldos a favor**: cuánto IVA tienes disponible para acreditar, cuánto ya aplicaste y cuánto ISR te han retenido de más.
- Exporta CSV del mes y respaldo completo en JSON.

## El cierre del mes

El panel de cierre es un espejo del portal del SAT: lista los campos que te va a pedir, con el nombre que usa el SAT y en el orden en que aparecen, agrupados por obligación. Cada uno se copia con un toque.

- **Calcula tu fecha límite real.** No todos declaramos el día 17: se le suman días hábiles según el sexto dígito numérico de tu RFC (Decreto del 26-dic-2013, art. 5.1, que la RMF recoge cada año). Descuenta fines de semana, los días festivos de ley y jueves y viernes santo. El SAT puede publicar días inhábiles extra que aquí no se conocen.
- **Avisa de lo que falta resolver** antes de declarar, con el IVA que está en juego.
- **Compara contra el portal.** Al cerrar puedes capturar lo que te mostró la pantalla del SAT. Si no cuadra con lo calculado, casi siempre es una factura que no tienes cargada; la diferencia queda anotada en el mes.
- **Congela el mes.** Una vez cerrado, un CFDI que llegue después con fecha de ese mes se marca como **tardía** y no reescribe en silencio lo que ya enviaste. Puedes incluirla a la fuerza (implica complementaria) o reabrir el mes completo.

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
2. Sube estos archivos a la raíz del repo: `index.html`, `manifest.webmanifest`, `icon.svg`, `README.md`.
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

## Próximos pasos

Ordenados por lo que más resuelve con menos trabajo. La regla del proyecto es que siga siendo un solo archivo que se abre y se entiende en diez segundos.

### 1. Descarga masiva del SAT
Hoy arrastras los XML a mano. El SAT tiene un servicio oficial y gratuito de descarga masiva de CFDI, autenticado con e.firma, que entrega todo lo emitido y recibido a tu RFC.

No se puede llamar desde el navegador: necesita SOAP y tu e.firma, que no debe salir de tu equipo. Sería un script de Node que corres localmente y que deja los XML en una carpeta; la app los lee de ahí. Es la mejora que más trabajo manual elimina, y también la más grande.

### 2. Verificar el estatus contra el SAT
Un proveedor puede cancelar una factura después de emitirla y hoy la seguirías acreditando. El SAT publica un servicio de consulta de estatus por UUID. No se puede llamar desde el navegador por CORS, así que encaja con el mismo script local del punto anterior.

### 3. Separar los topes propios de la deducción personal
Las colegiaturas y las aportaciones complementarias de retiro llevan límites propios que hoy no se distinguen del tope general. Con un subtipo por gasto el cálculo anual quedaría fino.

### 4. Que el clasificador aprenda de la descripción
Hoy aprende por RFC del emisor. Aprender también por palabras de la descripción cubriría a proveedores que facturan cosas distintas — un mismo RFC que a veces vende software y a veces comida.

### 5. Días inhábiles que publica el SAT
La fecha límite ya descuenta fines de semana, festivos de ley y semana santa, pero el SAT publica periodos de vacaciones generales y días inhábiles adicionales en su portal. Hoy no se conocen; sería una lista corta y editable a mano.

### Lo que conviene NO hacer

- **OCR de fotos de tickets.** Suena atractivo pero un ticket no genera IVA acreditable: solo un CFDI válido lo hace. El OCR agregaría un modelo de IA, una llave de API y una fuente de errores para producir datos que de todas formas hay que confirmar contra el XML.
- **Robots que facturen solos en portales de comercios.** Cada portal es distinto, muchos tienen CAPTCHA y romperlo no está sobre la mesa. Es trabajo de mantenimiento infinito para la parte más chica del problema.
- **Backend, cuentas de usuario, base de datos.** Nada de eso hace falta para una persona con un RFC, y cada pieza agrega costo, superficie de ataque y algo que se puede caer.

## Cifras que hay que actualizar cada año

Viven al inicio del `<script>` y son las únicas que caducan:

- `UMA_ANUAL` — valor anual de la UMA que publica el INEGI en enero.
- `TARIFA_ANUAL` — tarifa del art. 152 LISR (Anexo 8 de la RMF, DOF de finales de diciembre).
- `tasaISR()` — los cinco tramos del art. 113-E del RESICO.
