# XiteCore API 문서화 With Swagger

## Library
* [swagger-ui-express](https://github.com/scottie1984/swagger-ui-express) : Express에서 생성된 [swagger-ui](https://swagger.io/tools/swagger-ui/) API 문서 제공을 쉽게 도와준다.
* [swagger-jsdoc](https://github.com/Surnet/swagger-jsdoc) : jsdoc 주석으로 router를 마크업하고 swagger yml을 생성하며 이 모듈로 문서를 생성할 수 있다.

[swagger-ui](https://swagger.io/tools/swagger-ui/) : API 정의를 작성하고 시각화하여 클라우드에서 호스팅되는 UI를 생성하여 내부 개발자, 외부 소비자 등에게 API 문서에 대한 액세스를 제공할 수 있다.

```bash
"devDependencies": {
  "swagger-jsdoc": "^6.2.8",
  "swagger-ui-express": "^4.6.0"
}
```

## Usage

일반적인 사용법
Express setup
```javascript
// 모듈 추가
const express = require('express');
const swaggerJsdoc = require("swagger-jsdoc");
const swaggerUi = require("swagger-ui-express");

const app = express();
// Swagger-ui 옵션
const options = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "제목",
      version: "0.0.1",
      description: "설명",
      license: {
        // 라이센스
      },
    },
    servers: [
      // api 서버
    ],
  },
  apis: // router 파일 연동,
};

const specs = swaggerJsdoc(options);
// swagger api router, swagger-ui 설정 및 생성
app.use("/api-docs",
  swaggerUi.serve,
  swaggerUi.setup(specs)
);

module.exports = app;
```

문서 작성
[swagger-docs](https://swagger.io/docs/specification/about/)
```yaml
paths:
  /users{id}:
    get:
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: array
            items:
              type: integer
            minItems: 1
          style: matrix
          explode: true
        - in: query
          name: metadata
          schema:
            type: boolean
      responses:
        '200':
          description: A list of users
```

or 

```javascript
/** 
* @swagger
*     components:
*         schemas:
*             Book:
*                 type: object
*                 required:
*                     - title
*                     - author
*                     - finished
*                 properties:
*                     id:
*                         type: integer
*                         description: The auto-generated id of the book.
*                     title:
*                         type: string
*                         description: The title of your book.
*                     author:
*                         type: string
*                         description: Who wrote the book?
*                     finished:
*                         type: boolean
*                         description: Have you finished reading it?
*                     createdAt:
*                         type: string
*                         format: date
*                         description: The date of the record creation.
*                     example:
*                         title: The Pragmatic Programmer
*                         author: Andy Hunt / Dave Thomas
*                         finished: true
*/
```
or

```json
// 주소
"/pet/{petId}": {
      // 통신 메서드
      "get": {
        // 그룹명
        "tags": [
          "pet"
        ],
        // 주소 옆에 설명
        "summary": "Find pet by ID",
        "description": "Returns a single pet",
        "operationId": "getPetById",
        "parameters": [
          {
            "name": "petId",
            "in": "path",
            "description": "ID of pet to return",
            "required": true,
            "schema": {
              "type": "integer",
              "format": "int64"
            }
          }
        ],
        // 응답에 대한 설정
        "responses": {
          "200": {
            "description": "successful operation",
            "content": {
              "application/xml": {
                "schema": {
                  "$ref": "#/components/schemas/Pet"
                }
              },
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/Pet"
                }
              }
            }
          },
          "400": {
            "description": "Invalid ID supplied",
            "content": {}
          },
          "404": {
            "description": "Pet not found",
            "content": {}
          }
        },
        // 선언한 전역 객체로 제한 사항을 걸고자 하면 여기에 기재
        "security": [
          {
            "api_key": []
          }
        ]
      },
    }
```

서버를 열고 http://`<app_host>`:`<app_port>`/api-docs url 을 통해 접근하면 swagger-ui로 만들어진 ui를 통해 api 문서 엑세스가 가능해진다.

### JavaScript code 작성을 통한 API 문서 작성
여기서 js-doc를 통한 주석 작성이나 yaml, json 파일 작성은 그에 따른 장점도 있으나, 작성에 소비되는 시간과 노력이 크다.
그에 따라 JavaScript code를 통해서 규격에 맞는 문서를 작성하고 json으로 반환하여 문서화하는 설정.

### [swagger-jsdoc](https://www.npmjs.com/package/swagger-jsdoc)

If you are using swagger-jsdoc simply pass the swaggerSpec into the setup function:

```javascript
// Initialize swagger-jsdoc -> returns validated swagger spec in json format
const swaggerSpec = swaggerJSDoc(options);

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

### Swagger Explorer

By default the Swagger Explorer bar is hidden, to display it pass true as the 'explorer' property of the options to the setup function:

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  explorer: true
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

### Custom swagger options

To pass custom options e.g. validatorUrl, to the SwaggerUi client pass an object as the 'swaggerOptions' property of the options to the setup function:

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  swaggerOptions: {
    validatorUrl: null
  }
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

For all the available options, refer to [Swagger UI Configuration](https://github.com/swagger-api/swagger-ui/blob/master/docs/usage/configuration.md)

### Custom CSS styles

To customize the style of the swagger page, you can pass custom CSS as the 'customCss' property of the options to the setup function.

E.g. to hide the swagger header:

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customCss: '.swagger-ui .topbar { display: none }'
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

### Custom CSS styles from Url

You can also pass the url to a custom css file, the value must be the public url of the file and can be relative or absolute to the swagger path.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customCssUrl: '/custom.css'
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

You can also pass an array of css urls to load multiple css files.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customCssUrl: [
    '/custom.css',
    'https://example.com/other-custom.css'
  ]
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

### Custom JS

If you would like to have full control over your HTML you can provide your own javascript file, value accepts absolute or relative path. Value must be the public url of the js file.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customJs: '/custom.js'
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

You can also pass an array of js urls to load multiple js files.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customJs: [
    '/custom.js',
    'https://example.com/other-custom.js'
  ]
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

It is also possible to add inline javascript, either as string or array of string.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customJsStr: 'console.log("Hello World")'
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
  customJsStr: [
    'console.log("Hello World")',
    `
    var x = 1
    console.log(x)
    `
  ]
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

### Load swagger from url

To load your swagger from a url instead of injecting the document, pass `null` as the first parameter, and pass the relative or absolute URL as the 'url' property to 'swaggerOptions' in the setup function.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');

var options = {
  swaggerOptions: {
    url: 'http://petstore.swagger.io/v2/swagger.json'
  }
}

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(null, options));
```

To load multiple swagger documents from urls as a dropdown in the explorer bar, pass an array of object with `name` and `url` to 'urls' property to 'swaggerOptions' in the setup function.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');

var options = {
  explorer: true,
  swaggerOptions: {
    urls: [
      {
        url: 'http://petstore.swagger.io/v2/swagger.json',
        name: 'Spec1'
      },
      {
        url: 'http://petstore.swagger.io/v2/swagger.json',
        name: 'Spec2'
      }
    ]
  }
}

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(null, options));
```

Make sure 'explorer' option is set to 'true' in your setup options for the dropdown to be visible.


### Load swagger from yaml file

To load your swagger specification yaml file you need to use a module able to convert yaml to json; for instance `yaml`.

    npm install yaml

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const fs = require("fs")
const YAML = require('yaml')

const file  = fs.readFileSync('./swagger.yaml', 'utf8')
const swaggerDocument = YAML.parse(file)

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));
```


### Modify swagger file on the fly before load

To dynamically set the host, or any other content, in the swagger file based on the incoming request object you may pass the json via the req object; to achieve this just do not pass the the swagger json to the setup function and it will look for `swaggerDoc` in the `req` object.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {}

app.use('/api-docs', function(req, res, next){
    swaggerDocument.host = req.get('host');
    req.swaggerDoc = swaggerDocument;
    next();
}, swaggerUi.serveFiles(swaggerDocument, options), swaggerUi.setup());
```

### Two swagger documents

To run 2 swagger ui instances with different swagger documents, use the serveFiles function instead of the serve function. The serveFiles function has the same signature as the setup function.

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocumentOne = require('./swagger-one.json');
const swaggerDocumentTwo = require('./swagger-two.json');

var options = {}

app.use('/api-docs-one', swaggerUi.serveFiles(swaggerDocumentOne, options), swaggerUi.setup(swaggerDocumentOne));

app.use('/api-docs-two', swaggerUi.serveFiles(swaggerDocumentTwo, options), swaggerUi.setup(swaggerDocumentTwo));

app.use('/api-docs-dynamic', function(req, res, next){
  req.swaggerDoc = swaggerDocument;
  next();
}, swaggerUi.serveFiles(), swaggerUi.setup());
```

### Link to Swagger document

To render a link to the swagger document for downloading within the swagger ui - then serve the swagger doc as an endpoint and use the url option to point to it:

```javascript
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

var options = {
    swaggerOptions: {
        url: "/api-docs/swagger.json",
    },
}
app.get("/api-docs/swagger.json", (req, res) => res.json(swaggerDocument));
app.use('/api-docs', swaggerUi.serveFiles(null, options), swaggerUi.setup(null, options));
```


## Requirements

* Node v0.10.32 or above
* Express 4 or above

## Testing

* Install phantom
* `npm install`
* `npm test`
