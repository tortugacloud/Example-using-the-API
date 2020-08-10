＃当前API版本v1

>所有示例都是针对Node.js编写的，但是您可以在Web应用程序中轻松使用它们

###让我们登录Tortuga Cloud


```javascript
const crypto = require('crypto');
const axios = require('axios');

//我们不想知道您的密码或让攻击者截取它
//因此，请在授权之前对您的密码进行哈希处理

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

如果成功，您将收到一个令牌以访问API方法

```javascript
{
  token: '********************.********************.********************'
}
```

###太好了，让我们获取您的帐户信息

>这样您就可以控制您的支出和收入

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

如果成功，您将收到一个JSON对象，其中包含有关您帐户的信息（在此示例中，帐户角色是合作伙伴）

```javascript
//根据您帐户的角色，对象可能有所不同
//您可以在文档末尾的表格中看到idUserStatus和idUserGroup的含义
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

## 使用文件系统

### 请求有关存储空间中已用空间的信息

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

如果成功，您将收到一个JSON对象，其中包含有关您帐户的信息（在此示例中，帐户角色是合作伙伴）

```javascript
//根据您帐户的角色，对象可能有所不同
//度量单位-兆字节

{ 
    used: 0, 
    total: '10240000.000' 
}
```

###请求有关根目录结构的信息

> 要获取根目录的内容，请传递值0而不是id
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

如果成功，您将收到一个JSON对象，其中包含有关您帐户的信息

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

###创建目录

> idParent 等于当前的 idFolder

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

如果成功，您将收到回复

```text
Created
```

而且，如果重复对根结构的请求

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
//结果如下

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

###删除目录

> **注意，目录删除是递归执行的**

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

>成功删除后，您将获得一个空的响应正文

```text

```

### 重命名目录

> 不要将目录ID与idParent混淆！
> 文件管理器需要idParent才能工作

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

> 例如，将ID为10的“新文件夹”重命名

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
> 通过调用有关根目录的信息，我们得到

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

###上传文件

>使用Node.js FormData实现上传文件

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

>由于调用有关ID等于8的目录的信息，我们得到 
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

###重命名文件

>重命名刚刚加载到app.js的index.js
> **文件名包含其扩展名，更改它不会更改MIME类型**
>
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
//执行结果
{ status: 200, data: 'OK' }

```

### 下载文件

>要下载文件，请传递文件的objectName而不是：link
>  https://tortugacloud.com/api/v1/file-system/file/:link
>要获得指向该文件的简短链接，请使用我们的[链接缩短器]（https://tortugacloud.com/shorter）

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
//服务器响应示例

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: '隐藏文件内容以进行演示',
}
```

###删除文件

>要删除文件，我们需要它的ID和它所在的文件夹的ID
>
```javascript
//让我们删除app.js

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

//为此，执行请求

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

>服务器响应

```javascript
{ status: 200, data: 'OK' }
```

###什么是idUserStatus

| idUserStatus  | 值        |
| :-----------: |-----------------|
|      1        |活跃账户 |
|      2        |锁定帐户 |



###什么是idUserGroup

| idUserGroup  | 值        |
| :-----------: |-----------------|
|      1        |客户账户 |
|      2        |会员帐号 |
|      3        |会员账户（内容消费者） |
