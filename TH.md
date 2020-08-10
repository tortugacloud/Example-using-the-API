# API ปัจจุบันเวอร์ชัน v1.0

> ตัวอย่างทั้งหมดเขียนขึ้นสำหรับ Node.js แต่คุณสามารถใช้ในเว็บแอปพลิเคชันของคุณได้โดยไม่มีปัญหาใด ๆ

### เข้าสู่ระบบ Tortuga Cloud กันเถอะ


```javascript
const crypto = require('crypto');
const axios = require('axios');

// เราไม่ต้องการทราบรหัสผ่านของคุณหรืออนุญาตให้ผู้โจมตีสกัดกั้น  
// ดังนั้นแฮชรหัสผ่านก่อนการอนุญาต  

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

หากสำเร็จคุณจะได้รับโทเค็นเพื่อเข้าถึงเมธอด API

```javascript
{
  token: '********************.********************.********************'
}
```

### เยี่ยมมากขอข้อมูลบัญชีของคุณ

> วิธีนี้คุณสามารถควบคุมค่าใช้จ่ายและรายได้ของคุณ

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

หากสำเร็จคุณจะได้รับออบเจ็กต์ JSON พร้อมข้อมูลเกี่ยวกับบัญชีของคุณ (ในตัวอย่างนี้บทบาทบัญชีคือคู่ค้า)  

```javascript
// วัตถุอาจแตกต่างกันไปขึ้นอยู่กับบทบาทของบัญชีของคุณ
// คุณสามารถดูความหมายของ idUserStatus และ idUserGroup ได้ในตารางท้ายเอกสาร
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

## การทำงานกับระบบไฟล์

### ขอข้อมูลเกี่ยวกับพื้นที่ที่ใช้ในการจัดเก็บของคุณ

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

หากสำเร็จคุณจะได้รับออบเจ็กต์ JSON พร้อมข้อมูลเกี่ยวกับบัญชีของคุณ (ในตัวอย่างนี้บทบาทบัญชีคือคู่ค้า)  

```javascript
// วัตถุอาจแตกต่างกันไปขึ้นอยู่กับบทบาทของบัญชีของคุณ
// หน่วยวัด - เมกะไบต์
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### ขอข้อมูลเกี่ยวกับโครงสร้างของไดเรกทอรีราก

> ในการรับเนื้อหาของไดเร็กทอรีรูทให้ส่งค่า 0 แทน id  
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

หากสำเร็จคุณจะได้รับออบเจ็กต์ JSON พร้อมข้อมูลเกี่ยวกับบัญชีของคุณ  

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### สร้างไดเร็กทอรี

> idParent เท่ากับ idFolder ปัจจุบัน

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

หากสำเร็จคุณจะได้รับการตอบสนอง

```text
Created
```

ยิ่งไปกว่านั้นหากคุณทำซ้ำการร้องขอโครงสร้างรูท

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
// ผลลัพธ์จะเป็นดังนี้

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

### การลบไดเร็กทอรี

> **โปรดทราบการลบไดเร็กทอรีจะดำเนินการซ้ำ**

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

> เมื่อลบสำเร็จคุณจะได้รับเนื้อหาตอบกลับที่ว่างเปล่า

```text

```

### การเปลี่ยนชื่อไดเรกทอรี

> อย่าสับสนระหว่าง id ไดเรกทอรีกับ idParent !!
> idParent จำเป็นสำหรับตัวจัดการไฟล์ในการทำงาน

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

> ตัวอย่างเช่นเปลี่ยนชื่อ 'โฟลเดอร์ใหม่' ด้วย id 10

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
> จากการเรียกข้อมูลเกี่ยวกับไดเร็กทอรีรากเราจึงได้รับ

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

### การอัปโหลดไฟล์

> ใช้ Node.js การใช้งาน FormData เพื่ออัปโหลดไฟล์

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

> เนื่องจากการเรียกข้อมูลเกี่ยวกับไดเร็กทอรีที่มี id เท่ากับ 8 เราจึงได้รับ
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

### เปลี่ยนชื่อไฟล์

> มาเปลี่ยนชื่อ index.js ที่เราเพิ่งโหลดไปที่ app.js  
> **ชื่อไฟล์มีนามสกุลการเปลี่ยนไม่เปลี่ยนประเภท MIME**

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
// ผลการดำเนินการ
{ status: 200, data: 'OK' }

```

### ดาวน์โหลดไฟล์

> ในการดาวน์โหลดไฟล์ให้ส่ง objectName ของไฟล์แทน: link  
> https://tortugacloud.com/api/v1/file-system/file/:link  
> สำหรับลิงก์ที่ดีและสั้นไปยังไฟล์ให้ใช้ [link shortener] ของเรา (https://tortugacloud.com/shorter)

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
// ตัวอย่างการตอบสนองของเซิร์ฟเวอร์

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'เนื้อหาไฟล์ถูกซ่อนไว้เพื่อการสาธิต',
}
```

### การลบไฟล์

> ในการลบไฟล์เราต้องมี id และ id ของโฟลเดอร์ที่ไฟล์นั้นอยู่

```javascript
// เอา app.js ของเราออก

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

// ในการดำเนินการนี้ให้ดำเนินการตามคำขอ

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

> การตอบสนองของเซิร์ฟเวอร์

```javascript
{ status: 200, data: 'OK' }
```

### idUserStatus คืออะไร

| idUserStatus | มูลค่า |
| : -----------: | ----------------- |
| 1 | บัญชีที่ใช้งานอยู่ |
| 2 | บัญชีที่ถูกบล็อก |


### idUserGroup คืออะไร

| idUserGroup | ประเภทบัญชี |
| : -----------: | ----------------- |
| 1 | บัญชีลูกค้า |
| 2 | บัญชีพันธมิตร |
| 3 | บัญชีสมาชิก (ผู้บริโภคเนื้อหา) |
