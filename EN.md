# Current version of the API v1

> All examples are written for Node.js, but you will be able to use it in your web application without any problems  

### Let's log in to Tortuga Cloud


```javascript 
const crypto = require('crypto'); 
const axios = require ('axios'); 

// We don't want to know your password or allow a malicious hacker to intercept it 
// So hash your password before logging in 

const hashPasword = crypto.createHash ('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword 
}) 
.then(res => console.log('res --> ', res. data)) 
.catch(error => console.log('error --> ', error.response.data)) 
``` 

If successful, you will get a token to access the API methods

```javascript 
{ 
 token: '********************.********************.********************' 
} 
``` 

### Great, let's get your account information  

> This way you can control your expenses and income 

```javascript
const axios = require('axios');

const token = "***.***.***"

axios.get('https://tortugacloud.com/api/v1/user/', {
    headers: {
        Authorization: `JWT ${token}`
    } 
}) 
.then(res => console.log('res --> ', res. data)) 
.catch(error => console.log('error --> ', error.response.data)) 
```

If successful, you will get a JSON object with information about your account (in this example, the account role is partner) 

```javascript 
// The object may differ depending on the role of your account 
// You can see what idUserStatus and idUserGroup mean in the tables at the end of the document 
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

## Working with the file system  

### Request information about the spent space in your storage  

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

If successful, you will get a JSON object with information about your account (in this example, the account role is partner) 

```javascript 
/ / The object may differ depending on the role of your account 
// Units of measurement - megabytes 
{ 
 used: 0, 
    total: '10240000.000' 
} 
```

### Request information about the root directory structure 

> To get the contents of the root directory, pass the value 0 instead of id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```javascript

const axios = require('axios');

axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    } 
}) 
.then(res => console.log('res --> ', res. data)) 
.catch(error => console.log('error --> ', error.response.data)) 

``` 

If successful you will get a JSON object with information about your account 

```javascript 
{ 
 idFolder: 8, 
    folders: [], 
    files: [] 
} 
```

### Creating a directory 

> idParent equals the current idFolder

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
.then(res => console.log('res --> ', res. data)) 
.catch(error => console.log('error --> ', error.response.data)) 

``` 

If successful, you will receive a response 

```text
Created 
```

However if you repeat the request for the root structure 

```javascript 
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// The result will be as follows 

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

### Deleting a directory

> **Note that directory deletion is performed recursively** 

```javascript 

axios. delete('https://tortugacloud.com/api/v1/file-system/folder', {
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

> If you delete it successfully, you will get an empty response body

```text 

``` 

### Renaming the directory

> Don't confuse directory id with idParent !!  
> idParent is needed for the file Manager to work

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

> For example, rename the folder 'New Folder' with id 10 

```javascript 

axios.put('https://tortugacloud.com/api/v1/file-system/folder', {
    idFolder: 10,
    name: 'test'
}, {
    headers: {
        Authorization: `JWT ${token}`
    }, 
}) 
.then(res => console.log('res --> ', res. data)) 
.catch(error => console.log('error --> ', error.response.data)) 

``` 
> As a result of calling the root directory information, we get 

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

### Uploading files 

> Use Node to upload the file.js implementation of FormData 

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
.then(res => console.log('res --> ', res. data)) 
.catch(error => console.log('error --> ', error.response.data)) 
``` 

> As a result of calling directory information with id equal to 8, we get  
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

### Rename a file 

> Let's rename the newly loaded index.js in app.js  
> **The file name contains its extension, changing it does not change the MIME type** 

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
// Execution result 
{ status: 200, data: 'OK' }

```

### Downloading a file 

> To download the file, pass the objectName of the file instead of: link  
>  https://tortugacloud.com/api/v1/file-system/file/:link  
> To get a nice and short link to a file, use our [link shortener] (https://tortugacloud.com/shorter)  


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
// An example of server response 

{ 
 status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'File contents are hidden for demonstration', 
} 
```

### Deleting a file 

> To delete a file we need its id and the ID of the folder where it is located 

```javascript 
// Let's delete our app.js 

{ 
... 
 folders: [ ... ], 
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

// To do this, run the query 

axios. delete('https://tortugacloud.com/api/v1/file-system/file',
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
    data: res. data, 
})) 

``` 

> Server response

```javascript
{ status: 200, data: 'OK' } 
``` 

### What is idUserStatus 

| idUserStatus | Value | 
| :-----------: |-----------------|
| 1 | active account |
| 2 |blocked account | 


### What is idUserGroup?

| idUserGroup | account Type |
| :-----------: |-----------------|
| 1 | Client account |
| 2 | Partner account |
| 3 | Member account (content Consumers) |
