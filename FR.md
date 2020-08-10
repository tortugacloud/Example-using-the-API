# Version actuelle de l'API v1

> Tous les exemples sont écrits pour Node.js, mais vous pouvez les utiliser dans votre application web sans aucun problème

### Connectons-nous à Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

// Nous ne voulons pas connaître votre mot de passe ou permettre à un attaquant de l'intercepter
// Alors, hachez votre mot de passe avant l'autorisation

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

En cas de succès, vous recevrez un jeton pour accéder aux méthodes de l'API

```javascript
{
  token: '********************.********************.********************'
}
```

### Super, obtenons les informations de votre compte

> De cette façon, vous pouvez contrôler vos dépenses et vos revenus

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

En cas de succès, vous recevrez un objet JSON avec des informations sur votre compte (dans cet exemple, le rôle du compte est partenaire)
```javascript
// L'objet peut différer selon le rôle de votre compte
// Vous pouvez voir ce que signifient idUserStatus et idUserGroup dans les tableaux à la fin du document
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

## Travailler avec le système de fichiers

### Demander des informations sur l'espace utilisé dans votre stockage

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

En cas de succès, vous recevrez un objet JSON avec des informations sur votre compte (dans cet exemple, le rôle du compte est partenaire)
```javascript
// L'objet peut différer selon le rôle de votre compte
// Unités de mesure - mégaoctets
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Demander des informations sur la structure du répertoire racine

> Passez 0 au lieu de id pour obtenir le contenu du répertoire racine
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

En cas de succès, vous recevrez un objet JSON avec des informations sur votre compte

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Créer un répertoire

> idParent est égal à l'idFolder actuel

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

En cas de succès, vous recevrez une réponse

```text
Created
```

De plus, si vous répétez la requête pour la structure racine

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// Le résultat sera le suivant

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

### Suppression d'un répertoire

> ** Attention, la suppression d'un répertoire est effectuée de manière récursive **

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

> En cas de suppression réussie, vous obtiendrez un corps de réponse vide

```text

```

### Renommer un répertoire

> Ne confondez pas l'identifiant de répertoire avec idParent !!
> idParent est nécessaire pour que le gestionnaire de fichiers fonctionne

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

> Par exemple, renommez le 'Nouveau dossier' avec l'ID 10

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
> À la suite de l'appel d'informations sur le répertoire racine, nous obtenons

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

### Téléchargement de fichiers

> Utilisez l'implémentation FormData de Node.js pour télécharger le fichier

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

> À la suite de l'appel d'informations sur le répertoire avec un id égal à 8, nous obtenons
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

### Renommer un fichier

> Renommons l'index.js que nous venons de charger en app.js
> **Le nom du fichier contient son extension, sa modification ne change pas le type MIME**

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
// Résultat de l'exécution
{ status: 200, data: 'OK' }

```

### Télécharger un fichier

> Pour télécharger un fichier, passez le nom d'objet du fichier au lieu de: lien
> https://tortugacloud.com/api/v1/file-system/file/:link
> Pour un lien agréable et court vers le fichier, utilisez notre [raccourcisseur de lien] (https://tortugacloud.com/shorter)

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
// Пример ответа сервера 

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'Le contenu du fichier est masqué pour la démonstration',
}
```

### Suppression d'un fichier

> Pour supprimer un fichier, nous avons besoin de son identifiant et de l'identifiant du dossier dans lequel il se trouve

```javascript
// Supprimons notre app.js

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

// Pour ce faire, exécutez la requête

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

> Réponse du serveur

```javascript
{ status: 200, data: 'OK' }
```

### Qu'est-ce que idUserStatus

| idUserStatus | Valeur |
| : -----------: | ----------------- |
| 1 | compte actif |
| 2 | compte bloqué |


### Qu'est-ce que idUserGroup

| idUserGroup | Type de compte |
| : -----------: | ----------------- |
| 1 | Compte client |
| 2 | Compte affilié |
| 3 | Compte de membre (consommateurs de contenu) |
