# ChatBot EcoMarket

Chatbot retail conversacional para EcoMarket, evolucionado desde una PoC local a un MVP comercial con PostgreSQL, carrito multiarticulo, checkout, logging persistente y soporte documental con RAG.

Version actual recomendada: `ChatBot_2.2.0.ipynb`

## Estado del proyecto

El proyecto ya no esta en fase de prueba conceptual. La version `2.2.0` funciona como un MVP comercial con estas capacidades:

- consulta de productos por categoria o producto;
- deteccion de disponibilidad y control de stock;
- compra multiarticulo en una sola conversacion;
- carrito de compras conversacional;
- checkout con captura de cliente;
- seleccion de metodo de pago;
- creacion de pedido contra PostgreSQL / Neon;
- promociones automaticas simples;
- sugerencia de productos alternativos;
- logging persistente en base de datos;
- RAG documental para pagos, promociones, devoluciones y FAQ.

## Evolucion funcional

### v1.0.1 - PoC

- datos en memoria Python;
- 6 productos hardcodeados;
- 3 pedidos simulados;
- logging a CSV local;
- sin base de datos real;
- sin carrito;
- la intencion `Compra de productos` existia, pero no cerraba ventas.

### v2.2.0 - MVP comercial

- PostgreSQL 18 en Neon como fuente operativa de verdad;
- `RetailService` como unica capa de acceso a SQL;
- queries parametrizadas;
- validacion de esquema al arrancar;
- mDeBERTa + reglas + fallback con Groq;
- RAG documental acotado a conocimiento textual;
- carrito multiarticulo;
- checkout con nombre, email y telefono;
- metodos de pago del MVP:
  - `tarjeta`
  - `efectivo`
  - `contra_entrega`
- promociones automaticas:
  - 5% por volumen desde 3 unidades del mismo producto;
  - envio gratis desde 50 EUR;
  - 10% adicional desde 75 EUR;
- sugerencia de alternativas cuando el producto no existe exactamente o esta agotado;
- logging en tabla `logs_chatbot`.

## Arquitectura

La arquitectura actual separa claramente conversacion, logica de negocio y acceso a datos.

```text
Usuario
  ->
Reglas fuertes
  ->
mDeBERTa zero-shot
  ->
Groq fallback / extraccion estructurada
  ->
Motor de respuestas
  ->
RetailService
  ->
PostgreSQL / Neon
```

### Componentes clave

- `ChatBot_2.2.0.ipynb`
  Notebook principal del MVP.

- `RetailService`
  Capa unica de lectura y escritura sobre PostgreSQL.

- `rag_knowledge/`
  Base documental para pagos, promociones, devoluciones y preguntas frecuentes.

- `logs_chatbot`
  Tabla de trazabilidad para registrar preguntas, intenciones, confianza, origen y respuesta.

## Estructura del repositorio

```text
ChatBot_EcoMarket/
|
|- Bot/
|  |- ChatBot_1.0.1.ipynb
|  |- ...
|  |- ChatBot_2.2.0.ipynb
|  |- .env
|
|- data/
|  |- rag_knowledge/
|  |  |- devoluciones_y_cambios.md
|  |  |- pagos_y_checkout.md
|  |  |- promociones_y_cupones.md
|  |  |- faq_atencion_cliente.md
|  |
|  |- inventario_productos.xlsx
|  |- pedidos.xlsx
|
|- docs/
|  |- Avance_ChatBot_EcoMarket_v2_2_0.docx
|
|- README.md
|- requirements.txt
```

## Flujo conversacional del MVP

### 1. Exploracion

El usuario puede consultar por:

- categoria:
  `que productos de la categoria lacteos tienes disponible`
- producto:
  `tienes manzanas`

### 2. Construccion de carrito

Ejemplo:

```text
hola, deseo comprar 4 manzanas, 3 leche entera y 2 huevos
```

El bot:

- extrae varios productos;
- valida si existen;
- valida stock;
- agrega al mismo carrito;
- evita crear pedidos separados por cada mensaje.

### 3. Control comercial

Si hay stock suficiente:

- no muestra el stock exacto;
- confirma que puede agregar el producto al carrito.

Si el cliente pide mas de lo disponible:

- informa la cantidad realmente disponible;
- propone una contrapropuesta para no perder la venta.

Si el producto no existe exactamente o esta agotado:

- sugiere productos alternativos.

### 4. Checkout

El flujo pide:

1. nombre completo;
2. email;
3. telefono;
4. metodo de pago.

Metodos de pago del MVP:

- `tarjeta`
- `efectivo`
- `contra_entrega`

Regla operativa:

- `tarjeta` y `efectivo`: el bot indica acercarse a tienda para completar el pago;
- `contra_entrega`: el cobro se realiza al entregar el pedido.

### 5. Pedido final

Cuando el usuario confirma:

- se crea o actualiza el cliente;
- se genera un pedido unico;
- se registran lineas de detalle;
- se registran movimientos de inventario;
- se guarda la interaccion en `logs_chatbot`.

## Promociones del MVP

La version `2.2.0` aplica promociones automaticas simples y explicables:

- si el cliente compra 3 o mas unidades del mismo producto:
  `5%` de descuento por volumen en esa linea;
- si el total del carrito supera `50 EUR`:
  envio gratis;
- si el total promocional del carrito alcanza `75 EUR` o mas:
  `10%` de descuento adicional sobre el total.

Estas promociones estan pensadas para el MVP:

- sin cupones personalizados;
- sin reglas por cliente;
- sin acumulaciones complejas.

## RAG documental

El RAG se usa solo para conocimiento textual. No reemplaza la logica transaccional.

### Si usa RAG

- devoluciones y cambios;
- promociones y FAQ documental;
- metodos de pago;
- soporte textual.

### No usa RAG

- stock;
- carrito;
- clientes;
- pedidos;
- movimientos de inventario;
- validaciones operativas.

## Variables de entorno

El notebook busca un archivo `.env` y tambien puede leer variables del sistema.

Variables esperadas:

```env
DB_HOST=
DB_PORT=5432
DB_NAME=ChatBot_Ecomarket
DB_USER=
DB_PASSWORD=
DB_SSLMODE=require

GROQ_API_KEY=
GROQ_API_BASE=https://api.groq.com/openai/v1/chat/completions
GROQ_MODEL=llama-3.3-70b-versatile
GROQ_TIMEOUT_SECONDS=20
```

Importante:

- no subir `.env` al repositorio;
- mantener la API key y credenciales fuera del notebook.

## Dependencias

En `requirements.txt` actualmente figuran dependencias NLP base:

- `transformers`
- `torch`
- `sentencepiece`

Para ejecutar correctamente la version `2.2.0`, ademas se usan librerias como:

- `psycopg2-binary`
- `python-dotenv`

Si el entorno no las tiene instaladas, habra que agregarlas manualmente.

## Como ejecutar

### Opcion 1 - Jupyter Notebook

Abrir:

```text
Bot/ChatBot_2.2.0.ipynb
```

Ejecutar las secciones en orden:

1. Configuracion
2. Conexion PostgreSQL
3. RetailService
4. Utilidades NLP
5. RAG documental
6. Reglas de intencion
7. Generacion de respuestas y carrito
8. Logging del chatbot
9. Tests conversacionales
10. Chat interactivo

### Opcion 2 - Demo conversacional dentro del notebook

Ejemplos utiles para demo:

```text
que productos de la categoria lacteos tienes disponible
```

```text
hola, deseo comprar 4 manzanas, 3 leche entera y 2 huevos
```

```text
ver carrito
```

```text
finalizar compra
```

```text
Esteban Orozco, esteban@ecomarket.test, 600123456
```

```text
contra entrega
```

```text
si
```

```text
quiero yogur
```

```text
que promociones tienen activas
```

## Limitaciones actuales

- sigue siendo un notebook, no una aplicacion desplegada con API y frontend separados;
- Groq puede fallar si la API key no tiene permisos o cuota activa;
- la experiencia depende de la calidad del catalogo, aliases y stock cargado;
- las promociones actuales son generales y no contemplan campañas avanzadas;
- no hay autenticacion de cliente ni sesion persistente entre canales.

## Siguiente evolucion natural

Sin salir del enfoque MVP, los siguientes pasos mas utiles serian:

- exponer `RetailService` via FastAPI;
- medir conversion y abandono del checkout;
- formalizar mejor el motor de sustituciones;
- completar `requirements.txt` con dependencias reales del notebook;
- mover la logica principal desde notebook a modulos `.py`;
- crear una interfaz web o integracion con canal conversacional real.

## Equipo

Proyecto desarrollado por el equipo de analitica / data science para el caso EcoMarket, con foco en evolucionar desde una PoC de NLP hacia un MVP comercial conversacional usable y trazable.
