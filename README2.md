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
* 어노테이션을 통해 문서 작성
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

### JavaScript code를 통한 API 문서 작성
여기서 js-doc를 통한 주석 작성이나 yaml, json 파일 작성은 그에 따른 장점도 있으나, 작성에 소비되는 시간과 노력이 크다는 단점이 있다.

그에 따라 JavaScript code를 통해서 규격에 맞는 문서를 작성하고 json으로 반환하여 문서화하는 설정.

## 크게 3가지로 각 역할을 구분
* Express : 서버 초기화. 사용 할 url에 swaggerUI를 생성, 초기화 한다.
* ApiDoc : JavaScript를 통해 API 문서를 작성해 나눈 파일을 모으고 전달한다.
* Swagger : Swagger option과 문서를 관리하고 설정한다.

```javascript
// express.js
const express = require("express");
const ApiDocs = require("../../docs/index");

module.exports = () => {
  const app = express();
  initializeSwagger(app);

  return app;
};

function initializeSwagger(app) {
  // production 환경인 경우
  if (process.env.NODE_ENV === "production") return;

  const apiDocs = new ApiDocs();
  apiDocs.init();

  const { swaggerUI, specs, setUpOption } = apiDocs.getSwaggerOption();
  app.use("/api-docs", swaggerUI.serve, swaggerUI.setup(specs, setUpOption));
}
```

```javascript
// swagger.js
const swaggerProtocols = ["http", "https"];

const swaggerMediaTypes = ["application/json"];

const swaggerPorts = ["", "3000", "3001", "3002", "3003"];

/** @type {import("swagger-jsdoc").OAS3Definition["servers"]} */
const swaggerServers = [
  {
    url: "{protocol}://localhost:{port}",
    description: "LOCAL 서버",
    variables: {
      port: { enum: swaggerPorts, default: "3002" },
      protocol: { enum: swaggerProtocols, default: "http" },
    },
  },
  {
    url: "{protocol}://58.233.207.36:{port}",
    description: "DEV 서버",
    variables: {
      port: { enum: swaggerPorts, default: "3002" },
      protocol: { enum: swaggerProtocols, default: "http" },
    },
  },
  {
    url: "{protocol}://{enviroment}.xitecloud.io:{port}",
    description: "AWS, DEV URL",
    variables: {
      port: { enum: swaggerPorts, default: "3002" },
      protocol: { enum: swaggerProtocols, default: "http" },
      enviroment: { enum: ["dev", "core"], default: "dev" },
    },
  },
];

const swaggerSecurityDefinitions = {
  ApiKeyAuth: {
    type: "apiKey",
    name: "Authorization",
    in: "header",
  },
};

/**
 * @type {import("swagger-jsdoc").Components["securitySchemes"]}
 */
const swaggerSecurityScheme = {
  BearerAuth: {
    type: "http",
    scheme: "bearer",
    bearerFormat: "JWT",
    name: "Authorization",
    description: "인증 토큰 값을 넣어주세요.",
    in: "header",
  },
};

/** @type {import("swagger-jsdoc").Components["schemas"]} */
const swaggerComponents = {
  AUTH_ERROR: {
    description: "Auth Error",
    type: "object",
    properties: {
      403: {
        type: "You don't have permission to access / on this server",
      },
    },
  },
  SERVER_ERROR: {
    description: "Server Error",
    type: "object",
    properties: {
      500: {
        type: "Internal Error",
        code: 500,
      },
      400: {
        type: "Required Error",
        code: 400,
      },
    },
  },
  DB_ERROR: {
    description: "DB Error",
    type: "object",
    properties: {
      500: {
        type: "DB Error",
        code: 500,
      },
    },
  },
};

/**
 * Swagger docs
 * @link {https://swagger.io/docs/specification/about/}
 */
class Swagger {
  static _uniqueSwaggerInstance;
  /**
   * doc 문서 배열
   * @type {import("swagger-jsdoc").Paths}
   */
  _paths = [];
  /**
   * api option
   * @type {import("swagger-jsdoc").OAS3Options}
   */
  _option = {};
  /**
   * ui option
   * @type {import("swagger-ui-express").SwaggerUiOptions}
   */
  _setUpOption = {};

  /**
   * singleton
   * @returns {Swagger}
   */
  constructor() {
    if (!Swagger._uniqueSwaggerInstance) {
      this.init();
      Swagger._uniqueSwaggerInstance = this;
    }

    return Swagger._uniqueSwaggerInstance;
  }

  init = () => {
    this._option = {
      definition: {
        openapi: "3.0.0",
        /**
         * @description
         * update 2023-07-05
         * @version 1.0.1v
         */
        info: {
          version: "1.0.1",
          title: "XiteCore",
          description: "XiteCore RestFul API",
        },
        // 서버 설정
        servers: swaggerServers,
        // http, https
        schemes: swaggerProtocols,

        /* open api 3.0.0 version option */
        produces: swaggerMediaTypes,
        // 작성한 문서를 넣어줄 프로퍼티.
        paths: {},

        // security
        securityDefinitions: swaggerSecurityDefinitions,
        components: {
          responses: {
            UnauthorizedError: {
              description: "tokenError",
            },
          },
          securitySchemes: swaggerSecurityScheme,
          schemas: swaggerComponents,
        },
      },
      // Swagger 파일 연동
      apis: ["**/routes/*.routes.js"],
    };
    this._setUpOption = {
      customCss: ".swagger-ui .topbar { display: none }",
      customSiteTitle: "XiteCore API",
      swaggerOptions: {
        // 요청 시간 display
        displayRequestDuration: true,
        // 모든 tag default close
        docExpansion: "none",
      },
    };
  };

  /** @param {import("swagger-jsdoc").Paths} api */
  addAPI = (api) => {
    this._paths.push(api);
  };

  getOption = () => {
    const resultPath = {};

    // 객체화
    this._paths.forEach((path) => {
      for (const [key, value] of Object.entries(path)) {
        resultPath[key] = value;
      }
    });

    this._option.definition.paths = { ...this._option.definition.paths, ...resultPath };

    return {
      apiOption: this._option,
      setUpOption: this._setUpOption,
    };
  };
}

module.exports = Swagger;
```

```javascript
// index.js
const swaggerUI = require("swagger-ui-express");
const swaggerJsDoc = require("swagger-jsdoc");
const Swagger = require("../swagger/Swagger");

const common = require("../docs/api/common/index");
const dashboard = require("../docs/api/dashboard/index");
const manage = require("../docs/api/manage/index");
const project = require("../docs/api/project/index");

class ApiDocs {
  /**
   * api 문서 배열
   * @type {import("swagger-jsdoc").Paths[]}
   */
  _apiDocOption;
  _swagger;

  constructor() {
    // api 문서 배열 초기화 
    this._apiDocOption = [...common, ...dashboard, ...manage, ...project];
    // swagger 생성
    this._swagger = new Swagger();
  }

  init = () => {
    this._apiDocOption.forEach((x) => this._swagger.addAPI(x));
  };

  getSwaggerOption = () => {
    const { apiOption, setUpOption } = this._swagger.getOption();

    const specs = swaggerJsDoc(apiOption);
    return {
      swaggerUI,
      specs,
      setUpOption,
    };
  };
}

module.exports = ApiDocs;
```

이와 같이 javascript로 작성할 수 있도록 설정한 후 

Swagger open api 문서 규격에 맞는 데이터를 반환하도록 코드를 작성하면 된다.

## 예시 작성 코드
```javascript
const { omit, getDefaultContent, defaultSecurity } = require("../../swaggerUtil");

const defaultProperties = {
  card_id: {
    type: "number",
    description: "카드 id",
  },
  project_id: {
    type: "number",
    description: "프로젝트 id",
  },
  dashboard_id: {
    type: "number",
    description: "대쉬보드 id",
  },
  card_type: {
    type: "string",
    description: "카드타입 - 기상, 인원, 차량 ...",
  },
  x: {
    type: "number",
    description: "카드 위치 - x 좌표",
  },
  y: {
    type: "number",
    description: "카드 위치 -y 좌표",
  },
  w: {
    type: "number",
    description: "카드 크기 - 넓이",
  },
  h: {
    type: "number",
    description: "카드 크기 - 높이",
  },
  static: {
    type: "number",
    description: "카드 고정 여부",
  },
  term: {
    type: "number",
    description: "카드 갱신 주기(초)",
  },
  visible: {
    type: "number",
    description: "카드 visible 설정 0, 1",
  },
  option: {
    type: "object",
    description: "카드별 옵션값(자동, 수동 등의 설정외 기타)",
  },
};

const defaultExample = {
  card_id: "28",
  project_id: 1,
  dashboard_id: "12",
  card_type: "sensor",
  x: 106,
  y: 0,
  w: 22,
  h: 54,
  static: 0,
  term: "10",
  visible: 1,
  option: {
    latlng: "37.15001929990244, 127.31949922648478",
    zoomLevel: 14,
    use_dynamite: false,
    st_dt_dynamite: "000000",
    ed_dt_dynamite: "110500",
  },
};

const defaultTags = ["Dashboard/Card"];

/** @type {import("swagger-jsdoc").Paths[]} */
module.exports = [
  {
    "/card": {
      get: {
        summary: "카드 정보 목록 조회",
        tags: defaultTags,
        ...defaultSecurity,
        parameters: [
          {
            in: "query",
            name: "card_id",
            schema: {
              type: "number",
            },
            required: false,
            style: "form",
            description: "카드 id",
            example: null,
          },
          {
            in: "query",
            name: "dashboard_id",
            schema: {
              type: "number",
            },
            required: false,
            style: "form",
            description: "대쉬보드 id",
            example: 12,
          },
          {
            in: "query",
            name: "project_id",
            schema: {
              type: "number",
            },
            required: false,
            style: "form",
            description: "프로젝트 id",
            example: 1,
          },
          {
            in: "query",
            name: "card_type",
            schema: {
              type: "string",
            },
            required: false,
            style: "form",
            description: "카드 타입",
            example: "map",
          },
        ],
        responses: {
          200: {
            description: "카드정보 목록",
            content: getDefaultContent(defaultProperties, [defaultExample]),
          },
        },
      },
      post: {
        summary: "카드 정보 생성",
        tags: defaultTags,
        ...defaultSecurity,
        requestBody: {
          description: "project_id, dashboard_id, card_type is required",
          content: getDefaultContent(
            omit(defaultProperties, "card_id"),
            omit(defaultExample, "card_id")
          ),
        },
        responses: {
          200: {
            description: "카드 정보 생성 후 목록",
            content: getDefaultContent(defaultProperties, defaultExample),
          },
        },
      },
    },
  },
  {
    "/card/xca": {
      post: {
        summary: "카드 정보 수정",
        tags: defaultTags,
        ...defaultSecurity,
        requestBody: {
          description: "card_id required",
          content: getDefaultContent(defaultProperties, defaultExample),
        },
        responses: {
          200: {
            description: "카드 정보 수정 후 목록",
            content: getDefaultContent(defaultProperties, defaultExample),
          },
        },
      },
    },
  },
  {
    "/card/xcb": {
      post: {
        summary: "카드 정보 삭제",
        tags: defaultTags,
        ...defaultSecurity,
        description: "✨ 테스트 시 데이터 삭제 주의!",
        requestBody: {
          description: "card_id, dashboard_id, project_id is required",
          content: {
            "application/json": {
              schema: {
                properties: {
                  card_id: {
                    type: "number",
                    description: "카드 id",
                    example: 450,
                  },
                  project_id: {
                    type: "number",
                    description: "프로젝트 id",
                    example: 1,
                  },
                  dashboard_id: {
                    type: "number",
                    description: "대쉬보드 id",
                    example: 12,
                  },
                },
              },
            },
          },
        },
        responses: {
          200: {
            description: "삭제된 카드정보 목록",
            content: getDefaultContent(defaultProperties, defaultExample),
          },
        },
      },
    },
  },
  {
    "/card/batch": {
      post: {
        summary: "카드 정보 일괄 저장",
        tags: defaultTags,
        ...defaultSecurity,
        requestBody: {
          description: "data is array",
          content: {
            "application/json": {
              schema: {
                properties: defaultProperties,
                example: [
                  defaultExample,
                  {
                    card_id: "28",
                    project_id: 1,
                    dashboard_id: "12",
                    card_type: "carbon",
                    x: 0,
                    y: 61,
                    w: 22,
                    h: 67,
                    static: 0,
                    term: "10",
                    visible: 1,
                    option: {
                      term: "7",
                      termUnit: "w",
                      fn: "FN01",
                      startDate: "20221207",
                      assets: ["고압분사전용장비", "콘크리트 펌프차", "로더"],
                    },
                  },
                ],
              },
            },
          },
        },
        responses: {
          200: {
            description: "카드 정보 일괄 저장 후 목록",
            content: {
              "application/json": {
                schema: {
                  type: "object",
                  properties: defaultProperties,
                  example: [
                    defaultExample,
                    {
                      card_id: "28",
                      project_id_t_project_mng: 1,
                      dashboard_id_t_dashboard_set: "12",
                      f_card_type: "carbon",
                      f_x: 0,
                      f_y: 61,
                      f_w: 22,
                      f_h: 67,
                      f_static: "N",
                      f_term: "10",
                      f_visible: "Y",
                      f_option: {
                        term: "7",
                        termUnit: "w",
                        fn: "FN01",
                        startDate: "20221207",
                        assets: ["고압분사전용장비", "콘크리트 펌프차", "로더"],
                      },
                    },
                  ],
                },
              },
            },
          },
        },
      },
    },
  },
  {
    "/card/set/{card_type}/{project_id}": {
      get: {
        summary: "카드 설정 정보 조회",
        tags: defaultTags,
        ...defaultSecurity,
        parameters: [
          {
            in: "path",
            name: "card_type",
            schema: {
              type: "string",
            },
            required: true,
            style: "form",
            description: "카드 타입",
            example: "map",
          },
          {
            in: "path",
            name: "project_id",
            schema: {
              type: "number",
            },
            required: true,
            style: "form",
            description: "프로젝트 id",
            example: 1,
          },
        ],
        responses: {
          200: {
            description: "카드 설정 정보",
            content: getDefaultContent(defaultProperties, defaultExample),
          },
        },
      },
    },
  },
  {
    "/card/{card_id}": {
      get: {
        summary: "카드 정보 조회",
        tags: defaultTags,
        ...defaultSecurity,
        parameters: [
          {
            in: "path",
            name: "card_id",
            schema: {
              type: "number",
            },
            required: true,
            style: "form",
            description: "카드 id",
            example: 29,
          },
        ],
        responses: {
          200: {
            description: "카드 정보",
            content: getDefaultContent(defaultProperties, defaultExample),
          },
        },
      },
    },
  },
  {
    "/cardData/{project_id}/{card_type}": {
      get: {
        summary: "대시보드 카드 데이터 조회",
        tags: defaultTags,
        ...defaultSecurity,
        parameters: [
          {
            in: "path",
            name: "card_type",
            schema: {
              type: "string",
            },
            required: true,
            style: "form",
            description: "카드 타입",
            example: "weather",
          },
          {
            in: "path",
            name: "project_id",
            schema: {
              type: "number",
            },
            required: true,
            style: "form",
            description: "프로젝트 id",
            example: 1,
          },
        ],
        responses: {
          200: {
            description: "대시보드 카드 데이터",
            content: {
              "application/json": {
                schema: {
                  type: "object",
                  example: {
                    options: "...",
                    weatherData: "...",
                  },
                },
              },
            },
          },
        },
      },
    },
  },
];

```
