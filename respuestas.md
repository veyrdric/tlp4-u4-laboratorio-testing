# Respuestas del laboratorio

## 2. Conocer la aplicacion

**Que archivo contiene la regla de calculo?**
`src/services/precios.ts`, en la funcion `calcularPrecioFinal`.

**Que archivo define la ruta HTTP?**
`src/app.ts`.

**En que se diferencian sus responsabilidades?**
`precios.ts` solo hace el calculo y no sabe nada de HTTP. `app.ts` recibe la peticion, saca `precio` y `descuento` del body, llama al servicio y arma la respuesta con el codigo de estado.

Nota: la guia dice `/healthy` pero en el codigo la ruta es `/salud`. `/healthy` devuelve 404 y `/salud` devuelve `{"estado":"ok"}`.

## 4. Interpretar la falla

**1. Que total devuelve un descuento del 150 % sobre $1.000?**
Devuelve `-500`. Lo probe con curl y la API respondio `{"total":-500}` con codigo 200.

**2. Por que ese resultado no cumple la regla de negocio?**
Porque un descuento no puede ser mayor al 100 % y un precio final no puede ser negativo, seria como pagarle al cliente.

**3. Por que la prueba falla si la funcion termino de ejecutarse?**
Porque la prueba espera que la funcion lance un `RangeError`. Que termine sin error es justamente lo que esta mal, el test no pide que funcione sino que rechace ese valor.

## 7. Prueba de integracion HTTP

**Por que es de integracion y no unitaria?**
Porque no llama a la funcion directamente. Manda una peticion HTTP a la app con Supertest y ahi participan varias partes juntas: express parsea el JSON, la ruta llama al servicio, el servicio lanza el `RangeError` y la ruta lo convierte en un 400. Si alguna de esas partes falla, el test falla.

## 8. Typecheck y pruebas

**Por que conviene usar tanto typecheck como npm test?**
Porque revisan cosas distintas. `typecheck` revisa los tipos sin ejecutar nada, por ejemplo si paso un string donde va un number. `npm test` ejecuta el codigo y revisa que haga lo que tiene que hacer. Un codigo puede tener los tipos bien y calcular mal, o al reves. Ademas Node ejecuta los `.ts` sin revisar los tipos, entonces sin `typecheck` esos errores no se ven.

## 10. Registro de resultados

| Comprobacion | Resultado | Observacion |
| --- | --- | --- |
| Respuesta de `GET /healthy` | 404 | La ruta real es `/salud`, que devuelve `{"estado":"ok"}` |
| Primera ejecucion de `npm test` | 4 pasan, 1 falla | Falla `rechaza descuentos superiores al 100 %` porque no se lanza ningun error |
| Correccion del servicio | 5 de 5 pasan | Valide precio y descuento con `Number.isFinite` y `RangeError` |
| Nuevas pruebas unitarias | 7 de 7 pasan | Descuento 100 %, precio negativo, descuento negativo y `NaN` |
| Prueba de integracion HTTP 400 | 3 de 3 pasan | Con descuento 150 % la API responde 400 con la propiedad `error` |
| Verificacion `npm run typecheck` | Sin errores | |

**Desafio NaN:** si, la validacion lo rechaza porque `Number.isFinite(NaN)` da `false`. Antes de la correccion `calcularPrecioFinal(NaN, 10)` devolvia `NaN` sin avisar.

Tambien cambie el workflow de `npm install` a `npm ci` porque ahora `package-lock.json` esta en el repositorio.

## 11. Preguntas de cierre

**1. Que diferencia hay entre comprobar manualmente una respuesta y ejecutar una prueba automatizada?**
La manual la tengo que hacer yo cada vez, mirando la respuesta, y me puedo olvidar de algun caso. La automatizada queda escrita en codigo, se ejecuta en segundos y siempre revisa lo mismo.

**2. Cual fue la causa de la primera prueba fallida?**
`calcularPrecioFinal` no validaba los datos de entrada, entonces aceptaba un descuento de 150 % y devolvia -500 en vez de lanzar un `RangeError`.

**3. Por que es util incluir casos invalidos ademas de los habituales?**
Porque los errores suelen estar en esos casos. Con datos normales el codigo original andaba bien y el problema recien aparecia con un descuento mayor a 100 o un precio negativo.

**4. Que parte comprueba `tests/precios.test.ts` y que partes comprueba `tests/api.test.ts`?**
`precios.test.ts` comprueba solo la funcion `calcularPrecioFinal`. `api.test.ts` comprueba las rutas de express junto con el servicio, el codigo de estado y el JSON que devuelve.

**5. Que ventajas ofrece ejecutar las pruebas despues de cada modificacion?**
Si un cambio rompe algo que antes andaba me entero en el momento y se en que cambio fue. Asi se evitan las regresiones.

## Preguntas del README

**Que descubre el test que una prueba manual podria pasar por alto?**
Casos como el descuento de 150 %. Probando a mano lo normal es probar con 10 % y ver que da 900, y no se prueban los valores raros.

**Por que se vuelven a ejecutar todas las pruebas despues de corregir el codigo?**
Para confirmar que la correccion arreglo el error y que no rompio otra cosa, por ejemplo que el descuento de 10 % siga dando 900.

## Preguntas de repaso del material teorico

**1. Por que una API que devuelve HTTP 200 puede contener errores de negocio?**
Porque el 200 solo dice que la peticion se proceso. Antes de corregir, el descuento de 150 % respondia 200 con un total de -500.

**2. Que diferencia hay entre una prueba unitaria y una de integracion?**
La unitaria prueba una parte sola, como una funcion. La de integracion prueba varias partes trabajando juntas, como la ruta de express con el servicio.

**3. Por que `toThrow` recibe una funcion en lugar del resultado de ejecutarla?**
Porque si pongo la llamada directo, el error se lanza antes de llegar a `expect` y el test se corta. Pasandole una funcion, Vitest la ejecuta adentro y puede atrapar el error para revisarlo.

**4. Que diferencia existe entre `npm test` y `npm run typecheck`?**
`npm test` ejecuta las pruebas con Vitest y revisa el comportamiento. `npm run typecheck` ejecuta `tsc --noEmit`, que solo revisa los tipos y no ejecuta nada.

**5. Que aporta GitHub Actions al trabajo colaborativo?**
Ejecuta el typecheck y las pruebas automaticamente en cada push o pull request. Asi todo el equipo ve si un cambio rompe algo antes de juntarlo con el resto, sin depender de que cada uno se acuerde de correr las pruebas.
