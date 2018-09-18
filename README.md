# Activar CORS (Cross-Origin Resource Sharing)

## A nivel de servidor

Se necesita establecer una serie de [cabeceras](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) a enviar en cada respuesta:


La más importante es:

- [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)

Para permitir el acceso a los recursos que expone una api desde cualquier origen, esta cabecera debe estar establecida a '*'. No obstante, el empleo de * está desaconsejado, y se puede sustituir por una lista de los origin (hosts) desde los que se pueden realizar peticiones. 

En un ejemplo en nodejs:

```js

app.use(function(req, res, next){

	//ponemos el header al valor correcto
    res.header("Access-Control-Allow-Origin", "https://foo1.org, https://foo2.org");

	next();

})

```

Mediante este mecanismo, ya se puede realizar peticiones empleando los verbos:

- GET
- POST
- HEAD

Estas son las llamadas [simple requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Simple_requests).

El problema que se produce es que las peticiones de este tipo tienen [limitaciones](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Simple_requests).

Siendo las principales:

- Solo se permiten tres tipos de Content-Type:

  * application/x-www-form-urlenconded
  * multipart/form-data
  * text/plain

- Las cabeceras que se pueden enviar también son [limitadas](https://fetch.spec.whatwg.org/#cors-safelisted-request-header), necesitando realizar algunos cambios para poder, por ejemplo, enviar credenciales.  


### Preflight Requests

Si se pretende enviar una cabecera Authorization (para credenciales) se necesitará el empleo de [Preflight Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Preflighted_requests). 


Lo característico de este tipo de peticiones es que el navegador realiza una petición con OPTIONS *previa* a la verdadera petición, su razón es la de asegurarse que realmente es "seguro" enviar la credencial a ese sitio. 

Para habilitar las preflighted requests es necesario modificar un poco la lógica de respuesta del servidor. 

Se necesita discernir qué tipo de petición se está recibiendo:

* En el caso de métodos que no sean OPTIONS:
  * Devolver Access-Control-Allow-Origin (con '*' si se quiere habilitar para todos o una lista de los hosts permitidos)
  * Devolver Access-Control-Allow-Methods (con todos los métodos que se quieren permitir que no sean OPTIONS (GET,PUT,POST,DELETE...))
  * Devolver Access-Control-Allow-Headers (Con las cabeceras que se quieren permitir (principalmente 'Content-Type' y 'Authorization')=
* En el caso del método OPTIONS:
  * Devolver un 200 y no procesar la peticion (se puede devolver un 204 (no content))

En un ejemplo en nodejs,


```js

app.use(function(req, res, next){

  res.header('Access-Control-Allow-Origin', '*'); // o una lista de origins
  res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  //interceptamos el método OPTIONS y devolvemos sin procesar
  if(req.method === 'OPTIONS'){
     res.send(200);
  }
  else{
	//procesamos la request
    next();
  }

})


```













