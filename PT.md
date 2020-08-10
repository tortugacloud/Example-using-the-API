# Versão atual da API v1

> Todos os exemplos são escritos para Node.js, mas você pode usá-los em seu aplicativo da web sem problemas

### Vamos entrar no Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

// Não queremos saber sua senha ou permitir que um invasor a intercepte
// Então, hash sua senha antes da autorização

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

Se for bem-sucedido, você receberá um token para acessar os métodos da API

```javascript
{
  token: '********************.********************.********************'
}
```

### Ótimo, vamos obter as informações da sua conta

> Desta forma, você pode controlar suas despesas e receitas

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

Se for bem-sucedido, você receberá um objeto JSON com informações sobre sua conta (neste exemplo, a função da conta é parceiro)

```javascript
// O objeto pode ser diferente dependendo da função de sua conta
// Você pode ver o que idUserStatus e idUserGroup significam nas tabelas no final do documento
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

## Trabalhando com o sistema de arquivos

### Solicite informações sobre o espaço usado em seu armazenamento

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

Se for bem-sucedido, você receberá um objeto JSON com informações sobre sua conta (neste exemplo, a função da conta é parceiro)

```javascript
// O objeto pode ser diferente dependendo da função de sua conta
// Unidades de medida - megabytes
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Solicitar informações sobre a estrutura do diretório raiz

> Para obter o conteúdo do diretório raiz, passe o valor 0 em vez de id
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

Se for bem-sucedido, você receberá um objeto JSON com informações sobre sua conta

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Criar diretório

> idParent é igual ao idFolder atual

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

Se for bem sucedido, você receberá uma resposta

```text
Created
```

Além disso, se você repetir a solicitação para a estrutura raiz

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// O resultado será o seguinte

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

### Excluindo um diretório

> **Atenção, a exclusão do diretório é realizada recursivamente**

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

> Na exclusão bem-sucedida, você receberá um corpo de resposta vazio

```text

```

### Renomeando um diretório

> Não confunda o id do diretório com idParent !!
> idParent é necessário para o gerenciador de arquivos funcionar

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

> Por exemplo, renomeie a 'Nova Pasta' com id 10

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
> Como resultado da chamada de informações sobre o diretório raiz, obtemos

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

### Carregando arquivos

> Use a implementação Node.js FormData para fazer upload do arquivo

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

> Como resultado da chamada de informações sobre o diretório com id igual a 8, obtemos
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

### Renomear um arquivo

> Vamos renomear o index.js que acabamos de carregar para app.js
> **O nome do arquivo contém sua extensão, alterá-lo não altera o tipo MIME**

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
// Resultado da execução
{ status: 200, data: 'OK' }

```

### ⇬ Fazer download do arquivo

> Para baixar um arquivo, passe o nome do objeto do arquivo em vez de: link  
> https://tortugacloud.com/api/v1/file-system/file/:link
> Para obter um link agradável e curto para o arquivo, use nosso [link shortener] (https://tortugacloud.com/shorter)


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
  data: 'O conteúdo do arquivo está oculto para demonstração',
}
```

### Excluindo um arquivo

> Para deletar um arquivo, precisamos do seu id e do id da pasta em que está localizado

```javascript
// Vamos remover nosso app.js

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

// Para fazer isso, execute a solicitação

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

> Resposta do servidor

```javascript
{ status: 200, data: 'OK' }
```

### O que é idUserStatus

| idUserStatus | Valor |
| : -----------: | ----------------- |
| 1 | conta ativa |
| 2 | conta bloqueada |



### O que é idUserGroup

| idUserGroup | Tipo de conta |
| : -----------: | ----------------- |
| 1 | Conta do cliente |
| 2 | Conta de afiliado |
| 3 | Conta de membro (consumidores de conteúdo) |
