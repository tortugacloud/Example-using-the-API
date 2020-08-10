# Versione API corrente v1

> Tutti gli esempi sono scritti per Node.js, ma puoi usarli nella tua applicazione web senza problemi

### Accediamo a Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

// Non vogliamo conoscere la tua password o consentire a un utente malintenzionato di intercettarla
// Quindi, hash la tua password prima dell'autorizzazione

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

In caso di successo, riceverai un token per accedere ai metodi API

```javascript
{
  token: '********************.********************.********************'
}
```

### Ottimo, otteniamo le informazioni sul tuo account

> In questo modo puoi controllare le tue spese e le tue entrate

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

In caso di successo, riceverai un oggetto JSON con informazioni sul tuo account (in questo esempio, il ruolo dell'account è partner)

```javascript
// L'oggetto può differire a seconda del ruolo del tuo account
// Puoi vedere cosa significano idUserStatus e idUserGroup nelle tabelle alla fine del documento
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

## Lavorare con il file system

### Richiedi informazioni sullo spazio utilizzato nel tuo spazio di archiviazione

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

In caso di successo, riceverai un oggetto JSON con informazioni sul tuo account (in questo esempio, il ruolo dell'account è partner)

```javascript
// L'oggetto può differire a seconda del ruolo del tuo account
// Unità di misura: megabyte
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Richiedi informazioni sulla struttura della directory principale

> Passa 0 invece di id per ottenere il contenuto della directory principale
>  https://tortugacloud.com/api/v1/file-system/folder/:id

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

In caso di successo, riceverai un oggetto JSON con le informazioni sul tuo account

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Crea directory

> idParent è uguale a idFolder corrente

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

In caso di esito positivo, riceverai una risposta

```text
Created
```

Inoltre, se ripeti la richiesta per la struttura radice

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// Il risultato sarà il seguente

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

### Eliminazione di una directory

> **Attenzione, l'eliminazione della directory viene eseguita in modo ricorsivo**

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

> In caso di eliminazione riuscita, otterrai un corpo di risposta vuoto

```text

```

### Rinominare una directory

> Non confondere l'id della directory con idParent !!
> idParent è necessario affinché il file manager funzioni

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

> Ad esempio, rinomina la "Nuova cartella" con ID 10

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
> Come risultato della chiamata di informazioni sulla directory root, otteniamo

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

### Caricamento di file

> Usa l'implementazione di Node.js FormData per caricare il file

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

> Come risultato della chiamata di informazioni sulla directory con id uguale a 8, otteniamo
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

### Rinomina un file

> Rinominiamo index.js che abbiamo appena caricato in app.js
> **Il nome del file contiene la sua estensione, cambiarlo non cambia il tipo MIME**

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
// Risultato dell'esecuzione
{ status: 200, data: 'OK' }

```

### Download file

> Per scaricare un file, passare il objectName del file invece di: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Per un collegamento breve e piacevole al file, usa il nostro [link shortener] (https://tortugacloud.com/shorter)


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
// Esempio di risposta del server

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'Il contenuto del file è nascosto per dimostrazione',
}
```

### Eliminazione di un file

> Per eliminare un file, abbiamo bisogno del suo ID e dell'ID della cartella in cui si trova

```javascript
// Rimuoviamo il nostro app.js

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

// Per fare ciò, eseguire la richiesta

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

> Risposta del server

```javascript
{ status: 200, data: 'OK' }
```

### Cos'è idUserStatus

| idUserStatus | Valore |
| : -----------: | ----------------- |
| 1 | account attivo |
| 2 | account bloccato |



### Cos'è idUserGroup

| idUserGroup | Tipo di conto |
| : -----------: | ----------------- |
| 1 | Account cliente |
| 2 | Account affiliato |
| 3 | Account di iscrizione (consumatori di contenuti) |
