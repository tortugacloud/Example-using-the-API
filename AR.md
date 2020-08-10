# إصدار API الحالي v1

> جميع الأمثلة مكتوبة لـ Node.js ، ولكن يمكنك استخدامها بسهولة في تطبيق الويب الخاص بك

### دعنا نسجل الدخول إلى Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

// لا نريد معرفة كلمة المرور الخاصة بك أو السماح للمهاجمين باعتراضها
// لذا ، قم بتجزئة كلمة المرور قبل الإذن

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

إذا نجحت ، ستتلقى رمزًا مميزًا للوصول إلى طرق API

```javascript
{
  token: '********************.********************.********************'
}
```

### رائع ، دعنا نحصل على معلومات حسابك

> بهذه الطريقة يمكنك التحكم في نفقاتك ودخلك

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

في حالة النجاح ، ستتلقى كائن JSON به معلومات حول حسابك (في هذا المثال ، دور الحساب هو الشريك)

```javascript
// قد يختلف الكائن حسب دور حسابك
// يمكنك رؤية ما تعنيه idUserStatus و idUserGroup في الجداول الموجودة في نهاية المستند
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

## العمل مع نظام الملفات

### اطلب معلومات حول المساحة المستخدمة في التخزين لديك

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

في حالة النجاح ، ستتلقى كائن JSON به معلومات حول حسابك (في هذا المثال ، دور الحساب هو الشريك)

```javascript
// قد يختلف الكائن حسب دور حسابك
// وحدات القياس - ميغا بايت
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### طلب معلومات حول بنية الدليل الجذر

> مرر 0 بدلاً من id للحصول على محتويات الدليل الجذر
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

إذا نجحت ، فسوف تتلقى كائن JSON مع معلومات حول حسابك

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### إنشاء دليل

> idParent يساوي idFolder الحالي

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

إذا نجحت ، ستتلقى ردًا

```text
Created
```

علاوة على ذلك ، إذا كررت طلب بنية الجذر

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

### حذف دليل

> **تنبيه ، يتم حذف دليل بشكل متكرر**

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

> عند الحذف بنجاح ، ستحصل على نص استجابة فارغ

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

> على سبيل المثال ، أعد تسمية "مجلد جديد" بالمعرف 10

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
> نتيجة لاستدعاء معلومات حول الدليل الجذر ، نحصل على

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

### تحميل الملفات

> استخدام تطبيق Node.js FormData لتحميل الملف

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

> نتيجة لاستدعاء معلومات حول الدليل مع معرف يساوي 8 ، نحصل على
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

### إعادة تسمية ملف

> دعنا نعيد تسمية index.js الذي حمّلناه للتو إلى app.js
> **اسم الملف يحتوي على امتداده ، وتغييره لا يغير نوع MIME** 

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
// نتيجة التنفيذ
{ status: 200, data: 'OK' }

```

### تحميل الملف

> لتنزيل ملف ، قم بتمرير اسم الكائن للملف بدلاً من: link
>  https://tortugacloud.com/api/v1/file-system/file/:link
> للحصول على رابط لطيف وقصير للملف ، استخدم [link shortener] (https://tortugacloud.com/shorter)


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
// مثال على استجابة الخادم

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'محتوى الملف مخفي للشرح',
}
```

### حذف ملف

> لحذف ملف ، نحتاج إلى معرفه ومعرف المجلد الذي يوجد فيه

```javascript
// فلنقم بإزالة app.js

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

// للقيام بذلك ، قم بتنفيذ الطلب

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

> استجابة الخادم

```javascript
{ status: 200, data: 'OK' }
```

### ما هو idUserStatus

| idUserStatus  | القيمة        |
| :-----------: |-----------------|
|      1        |حساب نشط |
|      2        |حساب مغلق |



### ما هو idUserGroup

| idUserGroup  |القيمة        |
| :-----------: |-----------------|
|      1        |حساب العميل |
|      2        |حساب الأفلييت |
|      3        |حساب العضوية (مستهلكي المحتوى) |
