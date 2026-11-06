# Total Trading Limited — sitio web

Sitio de una sola página para **Total Trading Limited** (Jamaica), un servicio de
pedidos especiales de vehículos: el cliente dice qué auto quiere, año y presupuesto,
y la empresa lo busca en subastas y redes de distribuidores de Japón, Europa y
Norteamérica, y lo entrega en Jamaica.

Además del pedido especial, el negocio ahora también va a tener un pequeño inventario
propio de autos ya en Jamaica, listos para venta inmediata — el sitio está creciendo
para cubrir ambos caminos: "búscamelo" y "cómpralo ya".

## Estructura

Sitio estático, sin build ni dependencias. `dist/` es el sitio completo.

```
dist/
  index.html    página completa: HTML, CSS y JS en un archivo
  hero.mp4      video de portada (entra 5 s después de la pantalla de carga)
  seal.png      sello de marca, panel de pedido especial
  seal-sm.png   sello reducido, barra superior / pie / favicon
  og.png        imagen de vista previa al compartir el enlace
  _headers      cabeceras de seguridad y caché (Cloudflare lo lee de aquí)
netlify.toml    resto del hosting anterior; Cloudflare NO lo lee
```

Todo el JavaScript es propio, sin librerías externas. Las únicas peticiones de red
son las tipografías de Google Fonts (Archivo y Plus Jakarta Sans).

## Qué hace

- **Pantalla de carga** de 3 s con un auto girando, dibujado en canvas
- **Portada** con foto de póster y el video entrando con fundido a los 5 s
- **Globo 3D interactivo** (canvas, arrastrable, con inercia) que muestra las rutas
  de búsqueda mundial convergiendo en Kingston
- **Proceso de 4 pasos** en scroll horizontal en escritorio, apilado en móvil
- **Inventario** (`#inventario`): parrilla de autos en stock con foto, precio, estado y
  una **ficha de specs con etiquetas** (transmisión, kilometraje, combustible y
  tracción), **leída en vivo desde la base de datos** (Supabase). Los precios siguen el
  selector USD/JMD igual que la sección de depósito
- **Cuentas de cliente**: registro, inicio de sesión, cierre de sesión y recuperación
  de contraseña. La sesión sobrevive a recargas y se renueva sola
- **Autos guardados**: con sesión iniciada, el corazón de cada auto lo guarda en la
  cuenta, y un filtro deja ver solo los guardados
- **Formulario de solicitud**: marca, modelo, año y presupuesto
- **Sección de depósito** con tres vías de pago y un campo donde el visitante escribe
  el monto acordado y lo ve convertido a la otra moneda. La página **abre en JMD**, que
  es como cotiza el negocio; el botón USD/JMD de la barra convierte todo a dólares
  cuando hace falta
- **Bilingüe** inglés / español, y **modo día / noche** que respeta el ajuste del
  sistema y se puede forzar con el botón
- Respeta `prefers-reduced-motion`: sin animaciones automáticas si el visitante
  las tiene desactivadas

## Despliegue

**Cloudflare Pages** → https://total-trading-limited.pages.dev

El proyecto está conectado a GitHub: **cada push a la rama `claude/cloudflare-migracion`
se publica solo**. No hay comando que correr ni carpeta que arrastrar.

> ⚠️ **La rama de producción es `claude/cloudflare-migracion`, no `main`.** Un cambio
> que se quede en `main` no sale al aire.

### ⚠️ Las compilaciones son un recurso compartido

Cloudflare da **500 compilaciones al mes para todos los sitios de OmniaLink juntos**, y
cada push a una rama de producción gasta una. Por eso:

- Junta los cambios y súbelos en **un solo push**, no de a uno.
- Verifica antes de subir (abre el sitio, mira capturas, corre las pruebas).
- Nunca subas commits de prueba "a ver si sale".

Si se agota el límite los sitios no se caen, pero no se publica nada nuevo hasta el mes
siguiente.

### Qué cambió al salir de Netlify (5 nov 2026)

Netlify restringió toda la cuenta por falta de créditos del plan gratis y bajó los seis
sitios de golpe. La dirección `.netlify.app` **ya no funciona**.

Dos cosas que no se heredan y hay que recordar:

- **Cloudflare no lee `netlify.toml`.** Las cabeceras van en `dist/_headers` y las
  redirecciones irían en un `dist/_redirects`. El `_headers` ya cubre todo lo que tenía
  el `netlify.toml` (la caché de `hero.mp4`) y además añade las de seguridad y las PNG.
  El `netlify.toml` se queda solo porque el proyecto viejo de Netlify sigue de respaldo.
- **Netlify Forms no existe aquí.** El formulario de pedido ya no envía a ningún
  servidor: abre WhatsApp con el pedido escrito. Ver la sección del formulario.

## Base de datos (Supabase)

Proyecto: `ronaldfdez Dealer MJ` → https://uampqldgiditxueqmtlj.supabase.co

### Cómo agregar o editar autos (sin tocar código)

1. Entra a https://supabase.com → tu proyecto → **Table Editor** → tabla `cars`.
2. **Insert row** y llena los campos:

   | Campo | Qué poner |
   |---|---|
   | `make` | Marca: Toyota, BMW… (obligatorio) |
   | `model` | Modelo: Land Cruiser, X5… (obligatorio) |
   | `year` | Año (obligatorio) |
   | `price` | El importe, sin símbolo ni comas |
   | `price_currency` | `JMD` o `USD` — la moneda en que cotizas. El sitio muestra esa cifra
     exacta cuando el visitante mira en esa moneda, y la convierte marcada con `≈` en la otra |
   | `fuel` | Petrol, Diesel, Hybrid… |
   | `transmission` | Automatic / Manual |
   | `drivetrain` | 4WD, AWD, 2WD… |
   | `mileage` | Kilometraje, solo el número |

   Estos cuatro (más `mileage_unit`) son los que salen en la ficha de specs de la
   tarjeta, cada uno con su etiqueta. **Un campo que dejes vacío no desaparece: sale
   con una raya (—).** Es a propósito — un hueco se lee como "no se sabe", y esconderlo
   haría que un auto sin datos pareciera mejor documentado que otro.
   | `mileage_unit` | `km` por defecto. Solo ponlo en `mi` si el odómetro viene en millas |
   | `status` | `in_stock`, `reserved` o `sold` |
   | `photos` | **Lista** de enlaces a las fotos, en el orden en que se muestran. La
     primera es la de portada. Vacío = sale el ícono con "Foto próximamente". Con dos o
     más, la tarjeta las pasa sola cada 3,8 s y aparecen puntitos para saltar entre ellas |
   | `published` | **Déjalo en `false` mientras lo preparas.** Ponlo en `true` para que aparezca en el sitio |

3. Los cambios salen en la web al recargar la página. No hay que desplegar nada.

### Cómo subir las fotos de un auto

1. En Supabase → **Storage** → bucket **`car-photos`** → **Upload file**.
2. Sube las fotos del vehículo. Conviene un nombre que se entienda: `hilux-2023-front.jpg`.
3. En cada archivo → **Get URL** (o los tres puntos → *Copy URL*). Copia ese enlace.
4. Pega los enlaces en la columna `photos` de la tabla `cars`, **en el orden en que
   quieres que se vean**. La primera es la que sale de portada en la tarjeta.

**Antes de subir:** si la foto pesa más de ~1 MB, redúcela. Una foto de 2 MB por auto
hace que la página cargue lento en el celular de un comprador. Con 1200 px de ancho y
calidad media sobra: las del Hilux pesan ~150 KB cada una y se ven nítidas.

El bucket acepta solo imágenes (JPEG, PNG, WebP, AVIF) y hasta 10 MB por archivo.
Cualquiera puede **ver** las fotos —para eso están— pero **nadie puede subir ni borrar**
con la llave pública del sitio: eso solo se hace desde el panel, con la cuenta del dueño.

**`published` es el interruptor de publicación**: un auto con `published = false` es
invisible para el sitio, aunque exista en la tabla. Sirve para preparar una ficha con
calma sin que salga a medio llenar.

### ⚠️ El plan gratuito pausa la base tras ~7 días sin actividad

**Ya pasó tres veces**: el 18 de septiembre, el 6 de octubre y el 6 de noviembre. Las
tres veces el proyecto apareció como `INACTIVE` y **el inventario del sitio en vivo dejó de cargar** (mostraba
el mensaje de error a los visitantes). No se pierde nada — los datos siguen ahí y
vuelven al reactivarlo — pero mientras está pausado, cualquiera que entre a la web ve
un error en lugar de los autos.

Reactivarlo: panel de Supabase → botón **Restore project**.

**Cuánto tarda, de verdad:** la primera vez fueron un par de minutos. La segunda tardó
cerca de **media hora**. No hay que alarmarse ni tocar nada mientras tanto.

El estado pasa por tres fases, y conviene saber leerlas:

| Estado | Qué significa |
|---|---|
| `INACTIVE` | Pausado. El sitio muestra el error. |
| `COMING_UP` | Levantando la máquina. **El esquema reporta vacío.** |
| `RESTORING` | Devolviendo los datos a su sitio. El esquema sigue incompleto. |
| `ACTIVE_HEALTHY` | Listo. Recién aquí tiene sentido consultar. |

> Durante `COMING_UP` y `RESTORING` el proyecto reporta **el esquema vacío, 0 tablas y
> 0 usuarios**. **Es un espejismo, no pérdida de datos.** No hay que reconstruir nada
> ni volver a correr las migraciones — eso sí haría daño. Hay que esperar a
> `ACTIVE_HEALTHY` y volver a consultar.

### Hay una tarea programada que la mantiene despierta

Desde el 6 de octubre hay una Routine (`trig_01QknDnrRY7BtoNzwdcUdoVY`,
"Mantener despierta la base de Total Trading") que **lunes y jueves a las 9 AM de
Jamaica** hace una petición real al API REST, igual que la haría un visitante. Esa
petición cuenta como actividad y reinicia el contador de los 7 días.

Dos veces por semana y no una porque el margen de 7 días es demasiado justo: si el
único aviso semanal falla una vez, la base se cae igual. Con lunes y jueves nunca
pasan más de 4 días entre toques.

#### ⚠️ Estuvo un mes sin funcionar, y por qué

La base se pausó igual el 6 de noviembre, con la tarea corriendo desde el 6 de octubre.
La causa estaba en cómo la escribí: **las sesiones que dispara una Routine arrancan sin
el repositorio** (se ve en sus etiquetas, `config:no-git-repo`), y el texto de la tarea
les mandaba sacar la llave de `dist/index.html`. Sin repositorio no hay llave, sin
llave no hay petición válida, y el ping nunca llegó a hacerse. Un mes de ejecuciones
que terminaban "bien" sin tocar la base.

Arreglado el 6 de noviembre: **la llave publicable va escrita dentro de la propia
tarea**, que ya no depende de ningún archivo. Se puede hacer porque es la llave
publicable — la misma que está en el HTML a la vista de cualquiera — y lo que protege
los datos es el row-level security, no el secreto de esa llave.

La lección, por si vuelve a montarse algo así: **una tarea programada no hereda lo que
tiene la sesión donde se creó.** Ni el repositorio, ni los conectores. Todo lo que
necesite tiene que estar en su propio texto, y hay que probarla disparándola a mano y
mirando lo que de verdad hizo, no solo que diga `SUCCEEDED`.

**Lo que esa tarea NO puede hacer:** reactivar la base si ya se cayó. Las sesiones que
dispara no llevan las herramientas de Supabase, solo `curl`. Si encuentra el API caído
**manda un aviso al teléfono** y hay que entrar al panel a pulsar *Restore project* a
mano. Si algún día se quiere que lo haga sola, hay que añadirle el conector de Supabase
desde la pantalla de Routines en claude.ai.

Cuando todo está bien la tarea no avisa nada. El silencio es la señal buena — pero
`SUCCEEDED` en el historial **no** quiere decir que el ping se hiciera; eso fue
justamente lo que escondió el fallo durante un mes.

### Esto sigue siendo un parche

La tarea programada evita el síntoma, no la causa, y ya falló un mes entero sin que se
notara. El arreglo de verdad es **pasar al plan de pago de Supabase**, que no pausa:
son ~US$25/mes contra el cliente entrando a su propia web y encontrándola rota. Van
tres caídas en dos meses.

### ⚠️ Falta configurar el envío de correos antes de abrir el registro al público

Supabase trae un servicio de correo incluido, pero **solo envía a direcciones
preautorizadas** (las del equipo del proyecto), tiene un límite bajo por hora y su
propia documentación dice que **no es para producción**.

Traducido: un cliente real que se registre **no recibirá** el correo de confirmación
ni el de recuperar contraseña. Funciona para probar con tu propio correo, nada más.

Para abrirlo al público hay que conectar un SMTP propio en
**Authentication → SMTP Settings**. Hay opciones gratuitas suficientes para este
volumen (Brevo, Resend, Mailgun). Es una configuración de una sola vez, ~15 minutos.

También conviene revisar en **Authentication → URL Configuration** que el *Site URL*
apunte a https://total-trading-limited.pages.dev, porque de ahí salen los enlaces
de confirmación y de recuperación de contraseña.

### Seguridad

La llave que va en `dist/index.html` es la **llave publicable**, hecha para ser pública.
Lo que realmente protege los datos es el row-level security de las tablas:

- **`cars`**: con la llave publicable solo se pueden **leer** los autos publicados.
  Insertar, modificar y borrar están prohibidos. Editar solo se puede desde el panel
  de Supabase, con la cuenta del dueño.
- **`favourites`**: cada política está atada a `auth.uid()`, así que un cliente con
  sesión solo alcanza **sus propias** filas. Comprobado contra la base: un usuario no
  ve los guardados de otro, no puede guardar en su nombre ni borrarle nada, y alguien
  sin sesión no ve ninguno.

> La llave publicable viaja **solo en la cabecera `apikey`**. Supabase intenta leer la
> cabecera `Authorization: Bearer` como un JWT y rechaza la petición con "Invalid JWT"
> si le llega ahí. En `Authorization` solo va el token de sesión del propio usuario.

> La llave `service_role` / `sb_secret_...` **nunca** debe ir en el sitio ni compartirse.

## Próxima fase: pago en línea

1. ~~Inventario navegable conectado a base de datos~~ — **hecho**.
2. ~~Cuentas de cliente~~ — **hecho**, pendiente solo el SMTP (ver aviso arriba).
3. **Pago en línea** — bloqueado hasta decidir con el cliente qué procesador va a usar
   en Jamaica (WiPay, First Atlantic Commerce, Stripe, PayPal Business…). Sin cuenta de
   comercio no hay nada real que conectar. Cuando exista, el cobro debe autorizarse en
   un servidor (una Cloudflare Function), nunca desde el navegador.

## Pendiente antes de salir a producción

Esto es una **vista previa de diseño**. La página lo dice abiertamente en varios
sitios, y esos avisos deben quitarse solo cuando lo de abajo esté resuelto:

- [ ] **Números de cuenta bancaria** — NCB, Scotiabank y JN Bank dicen "pendiente"
- [x] **Montos de depósito** — resuelto: el dueño confirmó que **no hay montos fijos**,
      el depósito se acuerda vehículo por vehículo. Los cuatro botones inventados
      (US$500 / 1.000 / 2.500 / 5.000) se quitaron y en su lugar hay un campo donde el
      visitante escribe el monto que se le cotizó y lo ve en las dos monedas
- [ ] **Pasarela de pago** — hace falta cuenta de comercio (First Atlantic Commerce
      o WiPay) y un servidor que autorice cada cobro. La página **no pide datos de
      tarjeta en ningún punto**, y así debe seguir hasta que exista ese servidor.
- [x] **Formulario de solicitud** — **abre WhatsApp** (+1 876 404 3325) con el pedido ya
      escrito, en el idioma que tenga puesto el visitante. No manda nada solo: el cliente
      ve el mensaje y lo envía él. Antes usaba Netlify Forms, que no existe en Cloudflare
- [ ] **Tasa de cambio** — fija en 161 JMD/USD, etiquetada como indicativa. En producción
      debería venir de un servicio de tasas. Se cambia en la constante `USD_TO_JMD`
      dentro de `dist/index.html`.
- [ ] **Video de portada** — es vertical (540×960); en escritorio se recorta y se ve
      suave. Una versión horizontal mejoraría bastante.
- [ ] **Fotos propias** — la portada usa una imagen de referencia; conviene sustituirla
      por fotos de entregas reales con derechos de la empresa.
- [x] **Autos reales en el inventario** — los ficticios se borraron. Hoy hay un auto
      real (Toyota Hilux 2023). Los huecos de la fila se completan solos con casillas
      "SOON" hasta que entre más stock.
- [x] **Foto del Hilux** — publicada en `dist/cars/`, servida por Cloudflare. El dueño
      autorizó el uso de estas imágenes; conviene tener esa autorización por escrito
      **de la casa de subasta (USS/NAA)**, que es quien retiene el copyright según sus
      propios avisos — no del dueño del vehículo, que no posee las fotos que otro tomó.
- [x] **Almacenamiento de fotos** — bucket `car-photos` creado en Supabase Storage.
      Las del Hilux siguen en el repositorio (se subieron antes de montarlo); moverlas
      es opcional y solo cambia el valor de la fila.
- [ ] **SMTP para las cuentas** — el registro y la recuperación de contraseña ya
      funcionan, pero los correos solo llegan a direcciones preautorizadas hasta
      conectar un SMTP propio. Ver el aviso en la sección de la base de datos.
- [ ] **Pago en línea** — todavía no existe; depende de que se decida el procesador.
      Ver "Próxima fase".
