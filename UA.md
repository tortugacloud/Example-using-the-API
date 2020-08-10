# Поточна версія API v1

> Все приклади написані для Node.js, але ви зможете без проблем використовувати в своєму веб додатку

### Давайте авторізіруемся в Tortuga Cloud


```javascript
const crypto = require ('crypto');
const axios = require ('axios');

// Ми не хочемо знати ваш пароль або давати можливість зловмисник перехопити його
// Тому захешіруйте свій пароль перед авторизацією

const hashPasword = crypto.createHash ('md5'). update ("enterYourPassword"). digest ('hex');

axios.post ('https://tortugacloud.com/api/v1/user/login', {
     email: "enter@your.email",
     password: hashPasword
})
.then (res => console.log ('res ->', res.data))
.catch (error => console.log ('error ->', error.response.data))
```

При успішному виконанні ви отримаєте токен для доступу до методів API

```javascript
{
  token: '********************.********************.********************'
}
```

### Відмінно, давайте отримаємо інформацію про ваш обліковий запис

> Так ви зможете контролювати ваші витрати і доходи

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

При успішному виконанні ви отримаєте JSON об'єкт з інформацією про ваш обліковий запис (В даному прикладі роль облікового запису - партнер)

```javascript
// Об'єкт може відрізнятися в залежності від ролі вашого облікового запису
// Що означають idUserStatus і idUserGroup ви можете в таблицях в кінці документа
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

## Робота з файловою системою

### Запит інформації про витрачені місці в вашому сховищі

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

При успішному виконанні ви отримаєте JSON об'єкт з інформацією про ваш обліковий запис (В даному прикладі роль облікового запису - партнер)

```javascript
// Об'єкт може відрізнятися в залежності від ролі вашого облікового запису
// Одиниці виміру - мегабайти
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Запит інформації про структуру кореневої директорії

> Для отримання вмісту кореневої директорії передайте значення 0 замість id
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

При успішному виконанні ви отримаєте JSON об'єкт з інформацією про ваш обліковий запис

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Створення директорії

> IdParent дорівнює поточному idFolder

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

При успішному виконанні ви отримаєте відповідь

```text
Created
```

При цьому якщо повторити запит про кореневої структурі

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// Результат буде наступним

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

### Видалення директорії

> **Увага, видаленні директорії виконується рекурсивно**

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

> При успішному видаленні ви отримаєте порожній тіло відповіді

```text

```

### Перейменування директорії

> Не плутайте id директорії з idParent !!
> idParent потрібен для роботи файлового менеджера

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

> Для прикладу перейменуємо папку 'New Folder' з id 10

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
> В результаті виклику інформації про кореневій директорії отримаємо

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

### Завантаження файлів

> Для завантаження файлом скористайтеся Node.js реалізацією FormData

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

> В результаті виклику інформації про директорії c id рівним 8 отримаємо
>
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

### Перейменування файлу

> Давайте перейменуємо тільки що завантажений index.js в app.js
> **Файл містить його розширення, змінюючи його, ви не міняєте MIME-тип**

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

### Завантаження файлу

> Для скачування файлу передайте objectName файлу замість:link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Для отримання красивою і короткою посилання на файл скористайтеся нашим [укорачівателем посилань](https://tortugacloud.com/shorter)


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
  data: 'Вміст файлу приховано для демонстрації',
}
```

### Видалення файлу

> Для видалення файлу нам знадобиться його id і id папки в якій він знаходиться

```javascript
// Давайте видалимо наш app.js

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

// Для цього виконаємо запит

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

> Відповідь сервера

```javascript
{ status: 200, data: 'OK' }
```

### Що таке idUserStatus

| idUserStatus  | Значение        |
| :-----------: |-----------------|
|      1        |активна обліковий запис |
|      2        |заблокована обліковий запис |



### Що таке idUserGroup

| idUserGroup  | Тип учётной записи        |
| :-----------: |-----------------|
|      1        |Клієнтська обліковий запис |
|      2        |Партнерська обліковий запис |
|      3        |Членська обліковий запис (Споживачі контенту) |
