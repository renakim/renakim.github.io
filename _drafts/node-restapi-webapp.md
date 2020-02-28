



//* Link: https://docs.microsoft.com/ko-kr/windows/uwp/get-started/get-started-tutorial-fullstack-web-app

https://github.com/uglide/azure-content/blob/master/articles/app-service-api/app-service-api-nodejs-api-app.md


## 패키지 설치

```
$ npm install -g yo
$ npm install -g generator-swaggerize
```

## Download sample code

```
git clone https://github.com/Azure-Samples/app-service-api-node-contact-list.git
cd app-service-api-node-contact-list
```


## Swaggerize 실행
```
cd start
yo swaggerize

(log)
Swaggerize Generator
Tell us a bit about your application
? Path (or URL) to swagger document: api.json
? Framework: express
? What would you like to call this project: ContactList
? Your name: rena
? Your github user name: renakim
? Your email: rena@wiznet.io
   create .eslintignore
   create .eslintrc
   create .npmignore
   create package.json
   create README.md
   create server.js
   create tests\contacts.js
   create tests\contacts\{id}.js
   create handlers\contacts.js
   create handlers\contacts\{id}.js
   create config\swagger.json
   create data\mockgen.js
   create data\contacts.js
   create data\contacts\{id}.js

```


## Scaffolded code
cd ContactList
//npm install (이전 단계에서 수행됨)
npm install --save jsonpath swaggerize-ui


## Customize the code

lib/ 디렉토리를 생성된 ContactList로 복사

### handlers/contacts.js 수정

```javascript
"use strict";

var repository = require("../lib/contactRepository");

module.exports = {
  get: function contacts_get(req, res) {
    res.json(repository.all());
  }
};
```


### handlers/contacts/{id}.js 수정

```javascript
"use strict";

var repository = require("../../lib/contactRepository");

module.exports = {
  get: function contacts_get(req, res) {
    res.json(repository.get(req.params["id"]));
  }
};
```

### server.js 수정

```javascript
"use strict";

var port = process.env.PORT || 8000; // first change

var http = require("http");
var express = require("express");
var bodyParser = require("body-parser");
var swaggerize = require("swaggerize-express");
var swaggerUi = require("swaggerize-ui"); // second change
var path = require("path");

var app = express();

var server = http.createServer(app);

app.use(bodyParser.json());

app.use(
  swaggerize({
    api: path.resolve("./config/api.json"), // third change
    handlers: path.resolve("./handlers"),
    docspath: "/swagger" // fourth change
  })
);

// change four
app.use(
  "/docs",
  swaggerUi({
    docs: "/swagger"
  })
);

server.listen(port, function() {
  // fifth and final change
});
```


## Local Test

api.json을 ContactList/config/ 으로 복사

```
node server.js
```




# Azure App service Web app

## Cloud shell 사용

### 1. App service Plan 생성

App service에서 호스팅되는 앱에 대한 리소스 집한 정의

```
az appservice plan create --name <app service plan name> --resource-group <your resource group name> --sku FREE

az appservice plan create --name wizfitest-ASP --resource-group DefaultResourceGroup-EA --sku FREE
```

### 2. App service Plan에서 웹앱 프로비전

* --deployment-local-git: 로컬에서 Git을 사용해 웹앱 코드를 업로드하고 배포 할 수 있음

```
az webapp create -n <your web app name> -g <your resource group name> -p <your app service plan name> --deployment-local-git

az webapp create -n wizfi -g DefaultResourceGroup-EA -p wizfitest-ASP --deployment-local-git
```

### 3. 환경변수 설정 (생략 가능)

따옴표 사용하지 않음.


```
az webapp config appsettings set -n <your web app name> -g <your resource group name> --settings IotHubConnectionString=<your IoT hub connection string>
```

### 4. App 옵션 설정 (생략 가능)

웹소켓, HTTPS 요청만 허용 설정

```
	az webapp config set -n <your web app name> -g <your resource group name> --web-sockets-enabled true
az webapp update -n <your web app name> -g <your resource group name> --https-only true
	
	az webapp config set -n wizfi -g DefaultResourceGroup-EA --web-sockets-enabled true
	
	az webapp update -n wizfi -g DefaultResourceGroup-EA --https-only true
```

### 5. 사용자 수준 배포 자격 증명 설정

App service에 코드 배포 시 사용함.

설정 시 Azure 계정의 모든 구독에 있는 모든 App service 앱에서 사용 가능.


```
az webapp deployment user set --user-name <your deployment user name>
	
az webapp deployment user set --user-name wizfi
```

### 6. Git URL Get

코드를 App Service에 push 할 때 사용할 Git URL 획득

```
az webapp deployment source config-local-git -n <your web app name> -g <your resource group name>
	
az webapp deployment source config-local-git -n wizfi -g DefaultResourceGroup-EA
```

### 7. 코드 배포 (로컬에서 실행)

```
git remote add webapp <Git clone URL>
git remote add webapp https://wizfi@wizfi.scm.azurewebsites.net/wizfi.git

git push webapp master:master
```


### 8. App 상태 확인

```
az webapp show -n <your web app name> -g <your resource group name> --query state
	
az webapp show -n wizfi -g DefaultResourceGroup-EA --query state
```