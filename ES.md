# Versión actual de API v1

> Todos los ejemplos están escritos para Node.js, pero puedes usarlos en tu aplicación web sin ningún problema

### Inicie sesión en Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

// No queremos saber su contraseña ni permitir que un atacante la intercepte
// Entonces, hash tu contraseña antes de la autorización

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

Si tiene éxito, recibirá un token para acceder a los métodos de la API

```javascript
{
  token: '********************.********************.********************'
}
```

### Genial, obtengamos la información de tu cuenta

> De esta forma puedes controlar tus gastos e ingresos

```javascript
const axios = require('axios');

const token = "***.***.***"

axios.get('https://tortugacloud.com/api/v1/user/', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

Si tiene éxito, recibirá un objeto JSON con información sobre su cuenta (en este ejemplo, el rol de la cuenta es socio)

```javascript
// El objeto puede diferir según la función de su cuenta
// Puede ver lo que significan idUserStatus e idUserGroup en las tablas al final del documento
{
  id: 9,
  idUserStatus: 1,
  idUserGroup: 2,
  idTariff: 1,
  fullname: 'Name Lastname',
  email: 'enter@your.email',
  promoCode: 'ed40cc80',
  balance: 0
}
```

## Trabajando con el sistema de archivos

### Solicita información sobre el espacio utilizado en tu almacenamiento

```javascript

const axios = require('axios');

axios.get('https://tortugacloud.com/api/v1/file-system', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))

```

Si tiene éxito, recibirá un objeto JSON con información sobre su cuenta (en este ejemplo, el rol de la cuenta es socio)

```javascript
// El objeto puede diferir según la función de su cuenta
// Unidades de medida - megabytes
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Solicita información sobre la estructura del directorio raíz

> Para obtener el contenido del directorio raíz, pase el valor 0 en lugar de id
> https://tortugacloud.com/api/v1/file-system/folder/:id

```javascript

const axios = require('axios');

axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))

```

Si tiene éxito, recibirá un objeto JSON con información sobre su cuenta

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Crear el directorio

> idParent es igual al idFolder actual

```javascript

const axios = require('axios');

axios.post('https://tortugacloud.com/api/v1/file-system/folder', 
{
    idParent: 8,
    name: "New folder"
},
{
    headers: {
        Authorization: `JWT ${token}`
    }
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))

```

Si tiene éxito, recibirá una respuesta

```text
Created
```

Además, si repite la solicitud de la estructura raíz

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// El resultado será el siguiente

{
  idFolder: 8,
  folders: [
    {
      id: 9,
      idParent: 8,
      name: 'New folder',
      bucketName: '******************'
    }
  ],
  files: []
}

```

### Eliminando un directorio

> **Atención, la eliminación de un directorio se realiza de forma recursiva**

```javascript

axios.delete('https://tortugacloud.com/api/v1/file-system/folder', {
    headers: {
        Authorization: `JWT ${token}`
    },
    data: {
        idFolder: 9
    }
})
.then(res => console.log('res --> ', res.status))
.catch(error => console.log('error --> ', error.response.data))

```

> Si se elimina correctamente, obtendrá un cuerpo de respuesta vacío

```text

```

### Cambiar el nombre de un directorio

> ¡No confunda la identificación del directorio con idParent!
> idParent es necesario para que el administrador de archivos funcione

```javascript
{
...
  folders: [
    {
      id: 10,
      idParent: 8,
      name: 'New folder',
      bucketName: '****************'
    },
    ...
  ],
...
}
```

> Por ejemplo, cambie el nombre de la 'Nueva carpeta' con id 10

```javascript

axios.put('https://tortugacloud.com/api/v1/file-system/folder', {
    idFolder: 10,
    name: 'test'
}, {
    headers: {
        Authorization: `JWT ${token}`
    },
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))

```
> Como resultado de llamar a información sobre el directorio raíz, obtenemos

```javascript
{
...
  folders: [
    {
      id: 10,
      idParent: 8,
      name: 'test',
      bucketName: '****************'
    },
    ...
  ],
...
}
```

### Subiendo archivos

> Use la implementación FormData de Node.js para cargar el archivo

```javascript
const fs = require('fs');
const FormData = require('form-data');

let form = new FormData();
form.append('files',fs.readFileSync(__filename), 'index.js');

axios.post('https://tortugacloud.com/api/v1/file-system/file/8',
form.getBuffer(),
{
    headers: {
        ...form.getHeaders(),
        Authorization: `JWT ${token}`,
    }
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

> Como resultado de llamar a información sobre el directorio con id igual a 8, obtenemos

```javascript
{
...
    folders: [...],
    files: [
      {
        id: 1,
        name: 'index.js',
        sizeInKB: 1.138,
        FolderId: 8,
        objectName: '******'
      }
    ]
}
```

### Cambiar el nombre de un archivo

> Cambiemos el nombre del index.js que acabamos de cargar en app.js
> **El nombre del archivo contiene su extensión, cambiarlo no cambia el tipo MIME**

```javascript
axios.put('https://tortugacloud.com/api/v1/file-system/file',
{
    idFile: 1,
    name: "app.js"
}, 
{
    headers: {
        Authorization: `JWT ${token}`
    }
})
...
// Resultado de la ejecución
{ status: 200, data: 'OK' }

```

### Descargar archivo

> Para descargar un archivo, pase el objectName del archivo en lugar de: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Para obtener un enlace bonito y breve al archivo, utilice nuestro [acortador de enlaces] (https://tortugacloud.com/shorter)

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/file/****',
{
    headers: {
        Authorization: `JWT ${token}`
    }
})
.then(res => console.log('res --> ', {
    status:res.status,
    filename:res.headers['content-disposition'],
    data:res.data,
}))
// Ejemplo de respuesta del servidor

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'El contenido del archivo está oculto para demostración',
}
```

### Eliminando un archivo

> Para eliminar un archivo, necesitamos su id y el id de la carpeta en la que se encuentra

```javascript
// Давайте удалим наш app.js 

{
...
    folders: [...],
    files: [
      {
        id: 1,
        name: 'app.js',
        sizeInKB: 1.138,
        FolderId: 8,
        objectName: '******'
      }
    ]
}

// Para hacer esto, ejecuta la solicitud

axios.delete('https://tortugacloud.com/api/v1/file-system/file',
{
    headers: {
        Authorization: `JWT ${token}`
    },
    data: {
        idFile: 1,
        idFolder: 8
    }
})
.then(res => console.log('res --> ', {
    status:res.status,
    data:res.data,
}))

```

> Respuesta del servidor

```javascript
{ status: 200, data: 'OK' }
```
### ¿Qué es idUserStatus?

| idUserStatus | Valor |
| : -----------: | ----------------- |
| 1 | cuenta activa |
| 2 | cuenta bloqueada |



### ¿Qué es idUserGroup?

| idUserGroup | Tipo de cuenta |
| : -----------: | ----------------- |
| 1 | Cuenta de cliente |
| 2 | Cuenta de afiliado |
| 3 | Cuenta de membresía (consumidores de contenido) |
