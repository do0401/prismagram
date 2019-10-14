# PrismaGram
Instagram clone width Express + Prisma + React and React Native

## #0 Introduction
### #0.0 What are we building
- 인스타그램 클론 프로젝트에서는 node.js, react.js, react-native, prsima를 사용한다.

### #0.1 Requirements
- vs code / node / git / prisma 계정 / chrome / android studio

## #1 Set Up
### #1.0 Setting up the project
- prismagram 폴더를 생성한다.
- github에서 새 프로젝트를 만든다.(prismagram) gitignore는 Node 타입으로 추가한다.
- prismagram 폴더로 가서 아래 명령어를 실행한다

`yarn init`

- 대부분 default로 설정한다.
- 아래 dependency 모듈을 설치한다.

`yarn add graphql-yoga`
`yarn add nodemon -D`
`yarn add babel-node`
`yarn add babel-cli`

- 그리고 package.json에서 scripts에 아래 내용을 추가한다

```json
// package.json
"scripts": {
  "dev": "nodemon --exec babel-node src/server.js"
}
```

- nodemon.json 파일을 package.json과 같은 경로에 생성한다.
- 아래 내용을 추가한다.

```json
// nodemon.json
{
  "ext": "js graphql"
}
```
- nodemon이 감시해야 할 파일의 확장자들을 지정하는 것이다.
- nodemon은 파일을 저장할 때마다 서버를 재실행해준다.

### #1.1 Creating GraphQL Server
- prisma를 시작하기 전에 dotenv를 설치한다.

`yarn add dotenv`

- dotenv는 .env 파일을 읽는다.
- 이제 server.js를 작성한다.

```js
// server.js
require("dotenv").config();
import { GraphQLServer } from "graphql-yoga";

const PORT = process.env.PORT || 4000;

const typeDefs = `
    type Query{
        hello: String!
    }
`;

const resolvers = {
  Query: {
    hello: () => "Hi"
  }
};

const server = new GraphQLServer({ typeDefs, resolvers });

server.start({ port: PORT }, () =>
  console.log(`Server running on  http://localhost:${PORT}`)
);

// .babelrc
{
  "presets": ["@babel/preset-env"]
}
```

- package.json과 같은 경로에 .babelrc 파일을 생성하고 작성해줬다.
- 그리고 babel 관련 몇 가지 패키지를 설치한다.

`yarn add @babel/preset-env`
`yarn add @babel/node`
`yarn add @babel/core`

- 이제 graphql playground에서 hell 쿼리를 실행하면 hi 라는 응답이 올 것이다.

### #1.2 Setting Up the Server like the Pros
- morgan이라는 logger(로깅 전용 모듈) 미들웨어를 추가한다.

`yarn add morgan`

- 기본적으로 GraphQLServer에는 express가 내장되어있다.
- 그래서 우리는 express 서버에서 morgan 미들웨어를 사용할 것이다.

```js
// server.js
// 코드 추가
server.express.use(logger("dev"));
```

- 이제 server.js에 직접 typeDefs와 resolvers를 추가하지 않고, 더 프로페셔널한 방식으로 추가하는 방법을 적용해본다.
- 먼저 src 폴더에 api 폴더와 schema.js 파일을 생성한다.
- api 폴더에 들어가는 파일은 graphql 파일들과 resolver 파일들이다.
- 그리고 그것들을 모두 모아서 schema.js 파일로 밀어넣을 것이다.
- 그러면 server.js에는 schema.js 파일 하나만 import 시켜주면 되는 것이다.
- 이런 방식을 적용하려면 아래 패키지를 설치해야 한다.

`yarn add graphql-tools merge-graphql-schemas`

- 그리고 아래와 같이 코드를 작성해서 테스트한다.

```js
// src/api/Hello/sayHello/sayHello.js
export default {
  Query: {
    sayHello: () => "Hello"
  }
};

// src/api/Hello/sayHello/sayHello.graphql
type Query {
  sayHello: String!
}

// src/schema.js
import path from "path";
import {
  makeExecutableSchema
} from "graphql-tools";
import {
  fileLoader,
  mergeResolvers,
  mergeTypes
} from "merge-graphql-schemas";

const allTypes = fileLoader(path.join(__dirname, "/api/**/*.graphql"));
const allResolvers = fileLoader(path.join(__dirname, "/api/**/*.js"));

const schema = makeExecutableSchema({
  typeDefs: mergeTypes(allTypes),
  resolvers: mergeResolvers(allResolvers)
});

export default schema;
```

- 먼저 server.js에 넣었던 typeDefs와 resolvers 내용을 sayHello.graphql과 sayHello.js 파일에 각각 삽입해준다.
- 그리고 schema.js 파일에서 api 폴더 안에 있는 모든 graphql 파일과 js 파일을 allTypes와 allResolvers로 지정하고 makeExecutableSchema 함수를 통해 schema를 만들어준다.
- makeExecutableSchema 함수의 인자로 typeDefs와 resolvers를 입력하고 mergeTypes와 mergeResolvers로 위에서 입력했던 allTypes와 allResolvers를 merge해준다.
- 이 방식은 니콜라스가 graphql 프로젝트에서 사용하는 방식이며, api 폴더 밑에 모든 폴더 밑의 모든 graphql과 js 파일을 받아오는 것이다.
- server.js에는 기존 typeDefs와 resolvers 내용을 삭제하고 schema를 import한다.

```js
// server.js
// 코드 수정
require("dotenv").config();
import {
  GraphQLServer
} from "graphql-yoga";
import logger from "morgan";
import schema from "./schema";

const PORT = process.env.PORT || 4000;

const server = new GraphQLServer({
  schema
});

server.express.use(logger("dev"));

server.start({
  port: PORT
}, () => console.log(`✅ Server running on http://localhost:${PORT}`));
```

## #2 Setting Up Prisma
### #2.0 Introduction to Prisma
- Prisma는 ORM이다. Object-Relational Mapping(객체 관계 연결)
- Prisma는 데이터베이스 과련한 어려운 문제들을 해결해준다.
- Prisma는 어플리케이션에 필요한 모델을 graphql로 정의할 수 있다는 점에서 특별하다.