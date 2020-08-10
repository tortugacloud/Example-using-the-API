#API v1の現在のバージョン

>すべての例はNode用に書かれています。しかし、あなたは問題なくwebアプリケーションでそれを使用することができます  

##♪トルトゥーガクラウドにログインしよう


```javascript
const crypto = require('crypto');
const axios = require('axios');

////いいパスワードを使用することを目的と攻撃者が傍受で 
//こんなにハッシュパスワードでログイン

const hashPasword = crypto.createHash('md5').update("enterYourPassword").digest('hex');

axios.post('https://tortugacloud.com/api/v1/user/login', {
    email: "enter@your.email",
    password: hashPasword
})
.then(res => console.log('res --> ', res.data))
.catch(error => console.log('error --> ', error.response.data))
```

成功した場合は、APIメソッドにアクセスするトークンを取得します

```javascript
{
  token: '********************.********************.********************'
}
```

### それでは、アカウント情報を取得しましょう

> このようにして、費用と収入を管理できます

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

成功すると、アカウントに関する情報を含むJSONオブジェクトを受け取ります（この例では、アカウントの役割はパートナーです）

```javascript
//オブジェクトは、アカウントの役割によって異なる場合があります
//ドキュメントの最後のテーブルでidUserStatusとidUserGroupの意味を確認できます
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

##ファイルシステムの操作

###ストレージの使用済みスペースに関する情報をリクエストする

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

成功すると、アカウントに関する情報を含むJSONオブジェクトを受け取ります（この例では、アカウントの役割はパートナーです）
```javascript
//オブジェクトは、アカウントの役割によって異なる場合があります
//測定単位-メガバイト
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### ルートディレクトリの構造に関する情報を要求する

> ルートディレクトリの内容を取得するには、IDの代わりに値0を渡します
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

成功すると、アカウントに関する情報を含むJSONオブジェクトを受け取ります

```javascript
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

###ディレクトリを作成

> idParentは現在のidFolderと等しい

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

成功した場合、応答を受け取ります

```text
Created
```

また、ルート構造のリクエストを繰り返すと

```javascript
axios.get('https://tortugacloud.com/api/v1/file-system/folder/0', {
    headers: {
        Authorization: `JWT ${token}`
    }
})
// ...
//結果は次のようになります

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

###ディレクトリを削除する

> **注意、ディレクトリの削除は再帰的に実行されます**

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

>削除が成功すると、空の応答本文が返されます

```text

```

###ディレクトリの名前を変更する

> ディレクトリidをidParentと混同しないでください!!
> idParentは、ファイルマネージャーが機能するために必要です

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

>たとえば、「新しいフォルダ」の名前をID 10に変更します。

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
> ルートディレクトリに関する情報を呼び出した結果、

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

###ファイルのアップロード

> Node.js FormData実装を使用してファイルをアップロードする

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

> idが8のディレクトリに関する情報を呼び出した結果、
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

###ファイルの名前を変更する

> 読み込んだindex.jsの名前をapp.jsに変更しましょう
> **ファイルの名前には拡張子が含まれており、ファイル名を変更してもMIMEタイプは変更されません**

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
//実行結果
{ status: 200, data: 'OK' }

```

＃＃＃ ダウンロードファイル

>ファイルをダウンロードするには、次の代わりにファイルのobjectNameを渡します：link
>  https://tortugacloud.com/api/v1/file-system/file/:link
> ファイルへの簡潔で短いリンクを取得するには、 [リンクショートナー](https://tortugacloud.com/shorter)


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
//サーバー応答の例

{
  status: 200,
  filename: 'attachment; filename="key.ppk"',
  data: 'ファイルのコンテンツはデモ用に非表示になっています',
}
```

###ファイルを削除する

>ファイルを削除するには、ファイルのIDと、ファイルが配置されているフォルダーのIDが必要です

```javascript
// app.jsを削除しましょう

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

//これを行うには、リクエストを実行します

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

>サーバーの応答

```javascript
{ status: 200, data: 'OK' }
```

### idUserStatusとは

| idUserStatus  | 値        |
| :-----------: |-----------------|
|      1        |アクティブなアカウント |
|      2        |ロックされたアカウント |



### idUserGroupとは

| idUserGroup  | 値        |
| :-----------: |-----------------|
|      1        |クライアントアカウント |
|      2        |アフィリエイトアカウント |
|      3        |メンバーアカウント（コンテンツ利用者） |
