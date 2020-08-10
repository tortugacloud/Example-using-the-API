# Текущая версия API v1

> Все примеры написаны для Node.js, но вы сможете без проблем использовать в своём веб приложении  

### Давайте авторизируемся в Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

// Мы не хотим знать ваш пароль или давать возможность злоумышленику перехватить его
// Поэтому захешируйте свой пароль перед авторизацией

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

При успешном выполнении вы получите токен для доступа к методам API

```javascript
{
  token: '********************.********************.********************'
}
```

### Отлично, давайте получим информацию о вашей учётной записи  

> Так вы сможете контролировать ваши расходы и доходы 

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

При успешном выполнении вы получите JSON объект с информацией о вашей учётной записи (В данном примере роль учётной записи - партнёр)

```javascript
// Объект может отличаться в зависимости от роли вашей учётной записи 
// Что означают idUserStatus и idUserGroup вы можете в таблицах в конце документа
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

## Работа с файловой системой 

### Запрос информации об израсходованном месте в вашем хранилище

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

При успешном выполнении вы получите JSON объект с информацией о вашей учётной записи (В данном примере роль учётной записи - партнёр)

```javascript
// Объект может отличаться в зависимости от роли вашей учётной записи 
// Единицы измерения - мегабайты
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Запрос информации о структуре корневой директории 

>  Для получения содержимого корневой директории передайте значение 0 вместо id
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

При успешном выполнении вы получите JSON объект с информацией о вашей учётной записи

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Создание директории  

>  idParent равняется текущему idFolder

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

При успешном выполнении вы получите ответ

```text
Created
```

При этом если повторить запрос о корневой структуре 

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// Результат будет следующим 

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

### Удаление директории

> **Внимание, удалении директории выполняется рекурсивно**

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

> При успешном удалении вы получите пустое тело ответа

```text

```

### Переименование директории

> Не путайте id директории с idParent !!  
> idParent нужен для работы файлового менеджера

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

> Для примера переименуем папку 'New Folder' с id 10

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
> В результате вызова информации о корневой директории получим

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

### Загрузка файлов 

> Для загрузки файлом воспользуйтесь Node.js реализацией FormData

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

> В результате вызова информации о директории c id равным 8 получим  
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

### Переименование файла 

> Давайте переименуем только что загруженный index.js в app.js  
> **Имя файла содержит его расширение, меняя его, вы не меняете MIME-тип**

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
// Результат выполнения 
{ status: 200, data: 'OK' }

```

### Скачивание файла

>  Для скачивания файла передайте objectName файла вместо :link
>  https://tortugacloud.com/api/v1/file-system/file/:link
> Для получения красивой и короткой ссылки на файл воспользуйтесь нашим [укорачивателем ссылок](https://tortugacloud.com/shorter)


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
  data: 'Содержимое файла скрыто для демонстрации',
}
```

### Удаление файла

> Для удаление файла нам понадобится его id и id папки в которой он находится

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

// Для этого выполним запрос

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

> Ответ сервера

```javascript
{ status: 200, data: 'OK' }
```

### Что такое idUserStatus 

| idUserStatus  | Значение        |
| :-----------: |-----------------|
|      1        |активная учётная  запись |
|      2        |заблокированная учётная запись |



### Что такое idUserGroup

| idUserGroup  | Тип учётной записи        |
| :-----------: |-----------------|
|      1        |Клиентская учётная запись |
|      2        |Партнёрская учётная запись |
|      3        |Членская учётная запись (Потребители контента) |
