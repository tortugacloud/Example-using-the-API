# Aktuelle API-Version v1

> Alle Beispiele sind für Node.js geschrieben, aber Sie können sie problemlos in Ihrer Webanwendung verwenden

### Melden wir uns bei Tortuga Cloud an


```javascript
const crypto = require('crypto');
const axios = require('axios');

// Wir möchten Ihr Passwort nicht kennen oder einem Angreifer erlauben, es abzufangen
// Hash also dein Passwort vor der Autorisierung

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```
Bei Erfolg erhalten Sie ein Token für den Zugriff auf die API-Methoden

```javascript
{
  token: '********************.********************.********************'
}
```

### Großartig, lassen Sie uns Ihre Kontoinformationen abrufen

> Auf diese Weise können Sie Ihre Ausgaben und Einnahmen kontrollieren

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

Bei Erfolg erhalten Sie ein JSON-Objekt mit Informationen zu Ihrem Konto (in diesem Beispiel ist die Kontorolle Partner).

```javascript
// Das Objekt kann je nach Rolle Ihres Kontos unterschiedlich sein
// Sie können in den Tabellen am Ende des Dokuments sehen, was idUserStatus und idUserGroup bedeuten
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

## Arbeiten mit dem Dateisystem

### Fordern Sie Informationen zum verwendeten Speicherplatz in Ihrem Speicher an

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

Bei Erfolg erhalten Sie ein JSON-Objekt mit Informationen zu Ihrem Konto (in diesem Beispiel ist die Kontorolle Partner).

```javascript
// Das Objekt kann je nach Rolle Ihres Kontos unterschiedlich sein
// Maßeinheiten - Megabyte
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Informationen zur Struktur des Stammverzeichnisses anfordern

> Übergeben Sie 0 anstelle von id, um den Inhalt des Stammverzeichnisses abzurufen
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

Bei Erfolg erhalten Sie ein JSON-Objekt mit Informationen zu Ihrem Konto

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Verzeichnis erstellen

> idParent entspricht dem aktuellen idFolder

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

Bei Erfolg erhalten Sie eine Antwort

```text
Created
```

Darüber hinaus, wenn Sie die Anforderung für die Stammstruktur wiederholen

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// Das Ergebnis ist wie folgt

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

### Ein Verzeichnis löschen

> ** Achtung, das Löschen eines Verzeichnisses erfolgt rekursiv **

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

> Nach erfolgreichem Löschen erhalten Sie einen leeren Antworttext

```text

```

### Umbenennen eines Verzeichnisses

> Verwechseln Sie die Verzeichnis-ID nicht mit idParent !!
> idParent wird benötigt, damit der Dateimanager funktioniert

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

> Benennen Sie beispielsweise den 'Neuen Ordner' mit der ID 10 um

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
> Als Ergebnis des Aufrufs von Informationen über das Stammverzeichnis erhalten wir

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

### Dateien hochladen

> Verwenden Sie die FormData-Implementierung von Node.j, um eine Datei hochzuladen

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

> Als Ergebnis des Aufrufs von Informationen über das Verzeichnis mit einer ID von 8 erhalten wir
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

### Benennen Sie eine Datei um

> Benennen wir die gerade geladenen index.js in app.js um
> ** Der Name der Datei enthält ihre Erweiterung. Durch Ändern wird der MIME-Typ nicht geändert. **

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
// Ausführungsergebnis
{ status: 200, data: 'OK' }

```

### Download-Datei

> Um eine Datei herunterzuladen, übergeben Sie den Objektnamen der Datei anstelle von: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Verwenden Sie für einen schönen und kurzen Link zur Datei unseren [Link Shortener] (https://tortugacloud.com/shorter).

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
// Beispiel für eine Serverantwort

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'Der Dateiinhalt wird zur Demonstration ausgeblendet',
}
```

### Eine Datei löschen

> Um eine Datei zu löschen, benötigen wir ihre ID und die ID des Ordners, in dem sie sich befindet

```javascript
// Lass uns unsere app.js entfernen

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

// Führen Sie dazu die Anfrage aus

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

> Serverantwort

```javascript
{ status: 200, data: 'OK' }
```

### Was ist idUserStatus?

| idUserStatus | Wert |
| : -----------: | ----------------- |
| 1 | aktives Konto |
| 2 | gesperrtes Konto |


### Was ist idUserGroup?

| idUserGroup | Kontotyp |
| : -----------: | ----------------- |
| 1 | Kundenkonto |
| 2 | Affiliate-Konto |
| 3 | Mitgliedskonto (Inhaltskonsumenten) |
