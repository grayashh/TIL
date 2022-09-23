# **Express.js**

## Express.js란?

Node.js를 위한 프레임워크로 일부 필수적인 작업과 신경 쓰고 싶지 않은 세부 내용을 외부에 맡길 수 있도록 도와주고 일련의 규칙과 클린코드를 작성하고 핵심적인 일에 집중하게 도와주는 유틸리티 함수를 다수 제공한다

## 미들웨어(Middleware)란?

미들웨어는 express의 핵심이다. `요청과 응답의 중간(미들(middle))에 위치하여 미들웨어라고 부른다.`
미들웨어는 `app.user()`와 함께 사용된다.

<br>

`app.use()`에 매개변수가 req, res, next인 함수를 넣으면 된다. 미들웨어는 위에서부터 아래로 순서대로 실행되면서 요청과 응답 사이에 특별한 기능을 추가할 수 있다.
`next라는 세번째 매개변수를 사용했는데, 다음 미들웨어로 넘어가는 함수이다.` next를 실행하지 않으면 다음 미들웨어가 실행되지 않는다.

<br>

### 미들웨어가 실행되는 경우

- app.use(미들웨어) : 모든 요청에서 미들웨어 실행
- app.use('/abc', 미들웨어) : abc로 시작하는 요청에서 미들웨어 실행
- app.post('/abc', 미들웨어) : abc로 시작하는 POST 요청에서 미들웨어 실행

`app.use()나 app.get() 같은 라우터에 미들웨어를 여러개 장착할 수 있다.`

<br>

## 에러 처리 미들웨어

현재 app.get('/')에 두 번째 미들웨어에서 에러가 발생하고, 이 에러는 그 아래에 있는 에러 처리 미들웨어에 전달된다.
`에러 처리 미들웨어는` 매개변수가 `err, req, res, next`로 4개이다.

<br>

그리고 `app.js`를 아래와 같이 바꾼 후에 `npm start`를 한 후에 `http://localhost:3000`으로 접속해보자.

```javascript
import express from "express";

const app = express();

app.use((req, res, next) => {
  console.log("모든 요청에 다 실행된다.");
  next();
});

app.get(
  "/",
  (req, res, next) => {
    console.log("GET / 요청에서만 실행된다.");
    next();
  },
  (req, res) => {
    throw new Error("에러는 에러 처리 미들웨어로 간다.");
  }
);

app.use((err, req, res, next) => {
  console.error(err);
  res.staus(500).send(err.message);
});

app.listen(3000);
```

콘솔 값을 확인해보면 `위에서 부터 아래로 하나씩 실행`이 된 것도 볼 수 있고 브라우저에도 에러 응답 메세지를 볼 수 있다.

![2](https://user-images.githubusercontent.com/45676906/99909597-e3e24580-2d2c-11eb-9a55-eab06cd08f3e.png)

## POST 요청 처리

```javascript
import express from "express";
import bodyParser from "body-parser";

const app = express();

app.use(bodyParser.urlencoded());

app.post("/add-product", (req, res, next) => {
  res.send(
    "<form action='/product' method='POST'><input type='text' name='title'><button type='submit'>Add Product</button></form>"
  );
});

app.use("/product", (req, res, next) => {
  console.log(req.body);
  res.redirect("/");
});

app.use("/", (req, res, next) => {
  res.send("<h1>Hello from Express!</h1>");
});

app.listen(3000);
```

bodyParser.urlencoded()를 사용하면 POST 요청의 본문을 해석해서 req.body 객체로 만들어준다.

<br>

## 라우터 분리

```javascript
// routes/admin.js
import express from "express";

const router = express.Router();

router.get("/add-product", (req, res, next) => {
  res.send(
    "<form action='/add-product' method='POST'><input type='text' name='title'><button type='submit'>Add Product</button></form>"
  );
});

router.post("/add-product", (req, res, next) => {
  console.log(req.body);
  res.redirect("/");
});

export default router;
```

## **`메서드가 get과 post로 다르기 때문에 경로를 반복해도 된다`**

**여기서 이미 메서드가 다르면 같은 경로를 사용할 수 있다는 중요한 사실을 알 수 있습니다**

```javascript
// routes/shop.js
import express from "express";

const router = express.Router();

router.get("/", (req, res, next) => {
  res.send("<h1>Hello from Express!</h1>");
});

export default router;
```

```javascript
// app.js
import express from "express";
import bodyParser from "body-parser";
import adminRoutes from "./routes/admin.js";
import shopRoutes from "./routes/shop.js";

const app = express();

app.use(bodyParser.urlencoded());

app.use(adminRoutes);
app.use(shopRoutes);

app.listen(3000);
```

express의 `Router()`를 사용해 라우팅을 다른 파일에 위탁하는 편리한 방법을 제공한다.

<br>

## 페이지를 찾을 수 없을 때

처리할 수 없는 요청이 들어왔을 때 오류가 발생한다.

일반적으로 이런 경우에는 404 error 페이지가 나타나는데
`미들웨어 또는 Express가 미들웨어를 사용해 요청을 처리하는 방법을 활용하면 된다.`

```javascript
app.use((req, res, next) => {
  res.status(404).send("<h1>Page not found</h1>");
});
```

전송 전에 항상 status나 setHeader을 호출할 수 있다.

`모든 메서드 호출을 연계할 수 있고 마지막이 send이기만 하면 된다.`

<br>

## HTML 페이지 렌더링

```javascript
import express from "express";
import path from "path";

const __dirname = path.resolve();

const router = express.Router();

router.get("/", (req, res, next) => {
  res.sendFile(path.join(__dirname, "./", "views", "shop.html"));
});

export default router;
```

path.join은 실행 중인 운영체제에 따라 자동으로 올바른 경로를 생성한다.

dirname은 자신이 사용된 파일의 경로를 알려주는데

`__dirname is not defined 에러가 뜬다면 `

```javascript
const __dirname = path.resolve();
```

위 구문을 추가해주면 된다.

<br>

## helper functions

```javascript
// util/path.js
import path from "path";

export default path.dirname(process.mainModule.filename);
```

helper 함수의 도움을 받아 상위 디렉터리를 얻을 수 있고 routes 폴더에서 사용할 수 있는 방법중 하나이다.

`이 코드는 현재 실행되는 파일의 경로를 알려주고 파일명을 dirname에 입력해서 그 디렉터리의 경로를 알아낼 수 있다.`

<br>

## 정적 파일 제공

```javascript
app.use(express.static(path.join(__dirname, "public")));
```

`express.static`을 정적 파일을 제공하는 미들웨어로 등록해주고 static의 인자로 전달되는 'public' 이라는 디렉터리 밑에 있는 데이터들을 웹브라우저의 요청에 따라 제공해줄 수 있다.
