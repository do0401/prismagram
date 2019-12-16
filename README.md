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

- 이제 graphql playground에서 hello 쿼리를 실행하면 hi 라는 응답이 올 것이다.

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
- Prisma는 데이터베이스 관련한 어려운 문제들을 해결해준다.
- Prisma는 어플리케이션에 필요한 모델을 graphql로 정의할 수 있다는 점에서 특별하다.
- Prisma를 사용하기 위해서는 prisma 사이트에 가입해야 한다. 깃헙 계정으로도 로그인할 수 있다.

`npm install -g prisma`

- 먼저 prisma를 설치하고, prisma 사이트에 로그인하여 ADD A SERVICE 를 클릭한다.
- Log into Prisma CLI 밑에 나오는 코드를 복사해서 로그인을 시도한다.
- 로그인이 완료되면 성공 메시지와 함께 CREATE A NEW SERVICE 버튼이 보이며, 해당 버튼을 누르면 prisma를 시작하는 가이드 메시지를 확인할 수 있다.

`prisma init prismagram`

- prisma init 명령어로 서비스를 시작한다.
- 몇 가지 선택 항목이 나오는데 아래와 같이 선택한다.

`Demo server`
`~~~~/demo-us1`
`prismagram`
`dev`
`Prisma JavaScript Client`

- 그러면 코드가 생성된다. 생성된 파일은 깃헙에서 숨기고 싶으므로 .gitignore에 추가한다

```js
// .gitignore
generated
```

- 생성된 파일 중 datamodel.prisma 파일이 데이터 모델이다.
- 이 모델을 prisma에 추가하려면 아래 명령어를 입력한다.

`prisma deploy`

- prsima 계정에 모두 자동으로 업로드가 된다. 바뀐 점도 반영해준다.
- prisma 사이트에 가면 반영된 내용을 확인할 수 있다.

### #2.1 Datamodel with Prisma
- prisma에서 데이터 모델을 만들어보자.

```js
// datamodel.prisma
type User {
  id: ID! @id
  username: String! @unique
  email: String! @unique
  firstName: String @default(value: "")
  lastName: String
  bio: String
  followers: [User!]! @relation(name: "FollowRelation")
  following: [User!]! @relation(name: "FollowRelation")
  posts: [Post!]!
  likes: [Like!]!
  comment: [Comment!]!
}

type Post {
  id: ID! @id
  location: String
  caption: String!
  user: User!
  files: [File!]!
  likes: [Like!]!
  comment: [Comment!]!
}

type Like {
  id: ID! @id
  user: User!
  post: Post!
}

type Comment {
  id: ID! @id
  text: String!
  user: User!
  post: Post!
}

type File {
  id: ID! @id
  url: String!
  post: Post!
}
```

- ! 는 필수 값이라는 의미이다.
- @unique 는 값이 unique 함을 의미한다.
- @id 는 id 값을 의미한다. id를 추가하지 않으면 delete를 실행할 수 없다.
- @relation 을 통해 양방향으로 관계를 형셩해줄 수 있다.
- deploy 명령어를 아래와 같이 수정해주자.

```json
// package.json
// 코드 추가
"scripts": {
    "deploy": "prisma deploy",
```

- 이제 배포를 하면 prisma 관리 사이트에서 추가한 내용을 확인할 수 있다.
- 그리고 터미널창에 아래와 같이 admin 페이지 주소가 표시된다.

`https://us1.prisma.sh/keepitlow-e432a9/prismagram/dev/_admin`

- 위 주소에서 끝에 /_admin 만 삭제하고 브라우저에 입력하면 graphql playground를 확인할 수 있다.

### #2.2 Testing Prisma OMG
- datamodel.prisma 파일을 아래와 같이 완성한다.

```js
// datamodel.prisma
type User {
  id: ID! @id
  username: String! @unique
  email: String! @unique
  firstName: String @default(value: "")
  lastName: String
  bio: String
  followers: [User!]! @relation(name: "FollowRelation")
  following: [User!]! @relation(name: "FollowRelation")
  posts: [Post!]!
  likes: [Like!]!
  comment: [Comment!]!
  rooms: [Room!]!
  loginSecret: String!
}

type Post {
  id: ID! @id
  location: String
  caption: String!
  user: User!
  files: [File!]!
  likes: [Like!]!
  comment: [Comment!]!
}

type Like {
  id: ID! @id
  user: User!
  post: Post!
}

type Comment {
  id: ID! @id
  text: String!
  user: User!
  post: Post!
}

type File {
  id: ID! @id
  url: String!
  post: Post!
}

type Room {
  id: ID! @id
  participants: [User!]!
  messages: [Message!]!
}

type Message {
  id: ID! @id
  text: String!
  from: User! @relation(name: "From") // 같은 모델로 관계를 형성하려면 relation을 지정해주어야 한다.
  to: User! @relation(name: "To")
  room: Room!
}
```

- playground admin 페이지로 가서 사용자를 생성해보자.

```js
// playground admin (https://us1.prisma.sh/keepitlow-e432a9/prismagram/dev)
mutation {
  createUser(data: {username:"kdh", email:"kdh@test.com"}) {
    id
  }
}
```

- 위와 같이 사용자를 생성하고 admin 페이지로 이동해보면 사용자가 생성된 것을 확인할 수 있다.
- graphql 파일만 작성하면 resolver도 생성되고, 서버도 만들어지고 관리 패널까지도 만들어진다.
- 다른 사용자도 만들어본다.

```js
// playground admin (https://us1.prisma.sh/keepitlow-e432a9/prismagram/dev)
mutation {
  createUser(data: {username:"CHY", email:"CHY@test.com"}) {
    id
  }
}
```

- 이제 playground에서 방금 생성한 사용자 정보를 수정해본다.
- 만약 어떤 것을 실행해야 할지 찾고 싶으면 오른쪽 DOCS 탭에서 모든 것을 확인할 수 있다.
- 아래와 같이 입력해서 사용자를 업데이트한다.

```js
// playground admin (https://us1.prisma.sh/keepitlow-e432a9/prismagram/dev)
mutation {
  updateUser(data: { firstName: "HY", lastName:"C" } where: { id:" ck240sa59swuh0b0914rw17z8" }) {
    username
  }
}
```

- 그리고 사용자를 팔로잉하도록 한다.

```js
// playground admin (https://us1.prisma.sh/keepitlow-e432a9/prismagram/dev)
mutation {
  updateUser(
    data: { following: { connect: { id: "ck1smm6k444x40b403uwrojww" }}}
    where: { id: "ck240sa59swuh0b0914rw17z8" }
  ) {
    username
    firstName
    lastName
    following {
      id
    }
    followers {
      id
    }
  }
}
```

- 팔로잉하도록 설정되었고 실제로 반환된 내용을 보면 following에 팔로잉한 사용자의 id를 확인할 수 있다.
- 아래와 같이 팔로잉한 사용자를 조회하면 팔로워가 생긴 것을 확인할 수 있다.

```js
// playground admin (https://us1.prisma.sh/keepitlow-e432a9/prismagram/dev)
{
  user(where:{id:"ck1smm6k444x40b403uwrojww"}) {
    username
    followers {
      email
    }
    following {
      email
    }
    lastName
    firstName
  }
}
```

- 양방향 relation이 잘 작동하고 있다.
- admin 페이지에서도 모든 내용을 볼 수 있다.

### #2.3 Intergrating Prisma in our Server
- 실제 api에서 어떻게 prisma를 사용하는지 알아보자.
- 먼저 prisma.yml 파일을 .gitignore 파일에 추가한다.
- prisma.yml 에는 admin url이 있으므로 공개되서는 안된다.
- generated 폴더를 보면 prisma-client 폴더에 index와 prisma-schema 파일이 있다.
- 이 파일을 통해 prisma와 정보를 주고 받게 된다.
- prisma 서버와 정보를 주고 받으려면 prisma client를 다운받아야 한다.
- package.json에 generate 명령어와 prisma 명령어를 추가한다.

```json
// package.json
"scripts": {
  "deploy": "prisma deploy",
  "generate": "prisma generate",
  "prisma": "yarn run deploy && yarn run generate",
  "dev": "nodemon --exec babel-node src/server.js"
}
```

- yarn prisma 를 실행하면 deploy가 실행되고 변경사항들이 업로드 되고 나면 client가 generate 된다. prisma와 상호작용할 client는 javascript이다.
- 추가로 prisma-client-lib를 설치한다.

`yarn add prisma-client-lib`

- prisma는 자동으로 client를 만들어주며, 이 client는 사용자 정보를 체크할 수 있다.
- 사용자가 서버에 요청을 하면 서버가 prisma에 요청한다.

### #2.4 Resolvers with Prisma
- 우리는 기존 데이터모델 구조체를 복사해야 한다.

```js
// src/api/models.graphql
type User {
	id: ID!
	username: String!
	email: String!
	firstName: String
	lastName: String
	bio: String
	followers: [User!]!
	following: [User!]!
	posts: [Post!]!
	likes: [Like!]!
	comment: [Comment!]!
	rooms: [Room!]!
	loginSecret: String!
}

type Post {
	id: ID!
	location: String
	caption: String!
	user: User!
	files: [File!]!
	likes: [Like!]!
	comment: [Comment!]!
}

type Like {
	id: ID!
	user: User!
	post: Post!
}

type Comment {
	id: ID!
	text: String!
	user: User!
	post: Post!
}

type File {
	id: ID!
	url: String!
	post: Post!
}

type Room {
	id: ID!
	participants: [User!]!
	messages: [Message!]!
}

type Message {
	id: ID!
	text: String!
	from: User!
	to: User!
	room: Room!
}
```

- datamodel.prisma 파일 내용을 붙여넣기하고 directive 들을 모두 삭제한다. graphql 문법이 아니고 prisma 문법이기 때문이다. graphql은 prisma 문법을 이해하지 못한다.
- 코드를 복사하는 건 좋은 방법이 아니다. 나중에는 prisma 파일을 graphql로 인식시켜주는 모델이 나오길 바란다.
- 지금은 prisma 파일에 loginSecret 이라는 것만 추가해도 이것을 models.graphql에 똑같이 붙여넣어야 한다.
- 이제 Greeting 은 삭제하고 User 폴더를 만들고 userById 폴더와 allUsers 폴더를 추가한다.

```js
// src/User/userById/userById.graphql
type Query {
	userById(id: String!): User!
}

// src/User/userById/userById.js
import {
  prisma
} from "../../../../generated/prisma-client";
export default {
  Query: {
    userById: async (_, args) => {  // 첫번째 인자에서는 아무 것도 받지 않는다.
      const {
        id        // args 에서 id를 받는다.
      } = args;
      return await prisma.user({
        id        // id가 같은 user를 찾는다.
      });
    }
  }
}

// src/User/allUsers/allUsers.graphql
type Query {
	allUsers: [User!]!
}

// src/User/allUsers/allUsers.js
import {
  prisma
} from "../../../../generated/prisma-client";
export default {
  Query: {
    allUsers: async () => prisma.users()
  }
}
```

### #3.0 Planning the API

- [ ] Create account
- [ ] Request Secret
- [ ] Confirm Secret (Login)
- [ ] Like / Unlike a photo
- [ ] Comment on a photo
- [ ] Search by user
- [ ] Search by location
- [ ] See user profile
- [ ] Follow / Unfollow User
- [ ] See the full photo
- [ ] Edit my profile
- [ ] Upload a photo
- [ ] Edit the photo (Delete)
- [ ] See the feed

### #3.1 Create Account Resolver
- 먼저 admin 페이지에서 새 Post를 작성한다.
- caption, location, user 를 입력하고 Post를 생성한다.
- api 폴더 안에 Image/toggleLike 폴더를 생성한다. 이미 좋아요가 되어 있으면 좋아요 취소로 아이콘이 바뀔 것이다.
- 하지만 좋아요 작업을 하기 전에 로그인을 먼저해야 한다. 로그인 시스템을 먼저 만들자.
- User 폴더 안에 기존에 만들었던 allUsers와 userById 폴더를 삭제하고 createAccount 폴더를 생성한다.

- createAccount를 위해서 prisma를 확인하여, 사용자 생성을 위해 무엇이 필요한지 확인할 수 있다.(admin - playground에서 확인)

```js
// api/User/createAccount
// createAccount.graphql
type Mutation {
	createAccount(
    username: String!,
    email: String!,
    firstName: String,
    lastName: String,
    bio: String
  ): User!
}

type Query {
	something: String!
}
```

- graphql 파일에 mutation만 있으면 에러가 발생하므로 임의의 Query도 만들었다.

```js
// createAccount.js
import {
  prisma
} from "../../../../generated/prisma-client";
export default {
  Mutation: {
    createAccount: async (_, args) => {
      const {
        username,
        email,
        firstName = "",
        lastName = "",
        bio = ""
      } = args;
      const user = await prisma.createUser({
        username,
        email,
        firstName,
        lastName,
        bio
      });
      return user;
    }
  }
}
```

- firstName, lastName, bio는 비어 있을 수도 있으므로, 빈 문자열을 선언했다.
- playground에서 createAccount를 실행해보자.

```js
// playground
mutation {
  createAccount(username: "Keepitlow", email:"nmwh47@gmail.com", firstName: "Dohwan", lastName: "Kim"){
    id
  }
}
```

- admin 페이지에서 확인해 보면 사용자가 생성된 것을 확인할 수 있다.

### #3.2 requestSecret Resolver

- [x] Create account
- 이제 로그인을 해보자. 여기서 로그인은 비밀값을 요청하고 확인하는 과정을 의미한다.
- User 폴더 안에 requestSecret 폴더를 생성하고, 그 안에 requestSecret.js 파일과 requestSecret.graphql 파일도 만든다.

```js
// api/User/requestSecret
// reqeustSecret.graphql
type Mutation {
	requestSecret(email: String!): Boolean! // Boolean 타입을 리턴하고, 함수 인자로 String 타입인 email이 필요하다
}

// requestSecret.js
export default {
  Mutation: {
    requestSecret: async (_, args) => {
      const {
        email
      } = args;
    }
  }
}
```

- 비밀값을 생성할 함수를 만들기 위해 api폴더 바깥에 utill.js를 만든다.
- 비밀값을 만들기 위해서는 매우 많은 단어를 가져와야 한다.
- word generator에서 무작위 형용사 500개와 명사 500개를 가져온다.

```js
// src/words.js
export const adjectives = [
  // 가져온 형용사 500개 단어를 배열로 만든다.
]
export const nouns = [
  // 가져온 명사 500개 단어를 배열로 만든다.
]
```

- utills.js에는 randomNumber를 생성하는 generateSecret 함수를 만든다.

```js
// utill.js
import {
  adjectives,
  nouns
} from "./words"

export const generateSecret = () => {
  // 비밀값을 만드는 계산식
  const randomNumber = Math.floor(Math.random() * adjectives.length);
  // adjectives와 nouns를 리턴한다.
  return `${adjectives[randomNumber]} ${nouns[randomNumber]}`
};

// requestSecret.js
import { generateSecret } from "../../../utills";
import { prisma } from "../../../../generated/prisma-client";

export default {
  Mutation: {
    requestSecret: async (_, args) => {
      const { email } = args;
      const loginSecret = generateSecret();
      console.log(loginSecret);
      try {
        // 사용자를 찾아서 갱신 요청을 하려면 유니크한 특성(id, username, email)들로만 할 수 있다.
        // user의 email이 인자로 입력된 email과 같은 사용자를 where로 찾는다.
        await prisma.updateUser({data: {loginSecret}, where: {email}});
        return true;
      } catch {
        return false;
      }
    }
  }
}
```

- 테스트 전에 datamodel.prisma에서 User의 loginSecret을 필수값이 아니도록 수정한다.

```js
// datamodel.prisma
// 코드 수정
type User {
  id: ID! @id
  username: String! @unique
  email: String! @unique
  firstName: String @default(value: "")
  lastName: String
  bio: String
  followers: [User!]! @relation(name: "FollowRelation")
  following: [User!]! @relation(name: "FollowRelation")
  posts: [Post!]!
  likes: [Like!]!
  comment: [Comment!]!
  rooms: [Room!]!
  loginSecret: String // !를 제거했다.
}
```

`yarn prisma`

- client 코드를 generate하고 테스트를 해보자.

```js
// playground
mutation {
  requestSecret (email: "이메일입력")
}
```

- 위와 같이 입력하고 결과를 보면 정상적으로 "requestSecret": true 가 표시되는 것을 볼 수 있다.
- admin 페이지에 가서 해당 이메일을 가진 user를 확인해보면 로그인 비밀값이 저장되어 있는 것을 확인할 수 있다.

### #3.3 sendMail Function with Nodemailer

- 작업 전 아래 내용을 수정한다.

```js
// server.js
// 코드 수정
import dotenv from "dotenv";
import path from "path";
dotenv.config({
  path: path.resolve(__dirname, ".env")
});
```

- 기존 코드에서 process.env.PORT 는 사실 undefined 였으나, || 4000 예외 처리를 통해 에러가 나지 않고 있었다.
- dotenv는 같은 폴더에 있는 .env 파일을 가져온다.
- 하지만 dotenv는 prismagram(최상위폴더)에 있고 .env는 src 폴더에 있었으므로 dotenv는 .env 내용을 가져오지 못했다.
- 그래서 위와 같이 경로를 설정을 통해 .env 파일을 가져올 수 있도록 수정했다.

- 이제 비밀값을 이메일로 보내기 위해 nodemailer를 사용할 것이다.

`yarn add nodemailer`

- utils.js에 sendMail 함수와 sendSecretMail 함수를 만들어준다.

```js
// utils.js
// 코드 추가
export const sendMail = email => null;  // email은 주소만이 아닌, 필요한 모든 것이 담겨있다.

export const sendSecretMail = (address, secret) => {
  const email = {
    from: "kdh@prismagram.com",
    to: address,
    subject: "🔒Login Secret for Prismagram🔒",
    html: `Hello! Your login secret it ${secret}.<br/>Copy paste on the app/website to log in`
  }
};
```

- nodemailer는 기본 값들로 transport라는 것을 만들고, 그것으로 sendMail을 요청하면 끝이다.
- 우리는 sendgrid 라는 것을 사용해 볼 것이다.
- sendgrid 사이트로 이동해서 계정을 만들고 로그인한다.
- 로그인하고나서 sendgrid에서 테스트를 할 것이다.

`yarn add nodemailer-sendgrid-transport`

- sendgrid transport를 다운로드 받는다.
- utils.js에 nodemailer 모듈과 nodemailer=sendgrid-transport 모듈을 import 한다.

```js
// utils.js
// 코드 추가/수정
import dotenv from "dotenv";
import path from "path";
dotenv.config({
  path: path.resolve(__dirname, ".env") // server.js와 마찬가지로 dotenv를 import했다
});
import nodemailer from "nodemailer";
import sgTransport from "nodemailer-sendgrid-transport";

export const sendMail = email => {
  const options = {
    auth: {
      api_user: process.env.SENDGRID_USERNAME,  // sendgrid 계정 username 입력
      api_key: process.env.SENDGRID_PASSWORD    // sendgrid 계정 password 입력
    }
  };
};

// .env
SENDGRID_USERNAME= [sendgrid id]
SENDGRID_PASSWORD= [sendgrid pw]
```
- import 후 sendMail에 options을 만들었다.
- sendgrid 계정 정보는 .env에 넣어둔다.
- 다음으로 nodemailer.createTransport와 client.sendMail을 실행해야한다.

```js
// utils.js
// 코드 추가
const sendMail = email => {   // sendMail은 외부에서 사용하지 않을 것이므로 export 하지 않아도 된다.
  const options = {
    auth: {
      api_user: process.env.SENDGRID_USERNAME,
      api_key: process.env.SENDGRID_PASSWORD
    }
  };
  const client = nodemailer.createTransport(sgTransport(options));  // createTransport를 실행한 결과물을 client에 할당한다.
  return client.sendMail(email);  // 그리고 cilent.sendMail을 실행한다.
};

export const sendSecretMail = (address, secret) => {
  const email = {
    from: "kdh@prismagram.com",
    to: address,
    subject: "🔒Login Secret for Prismagram🔒",
    html: `Hello! Your login secret it ${secret}.<br/>Copy paste on the app/website to log in`
  };
  return sendMail(email);   // sendMail(email)은 여기서 실행한다.
};
```
- server.js 에서 테스트로 sendSecretMail 함수를 실행시키면 메일이 날아오는 것을 확인할 수 있다.

### #3.4 Passport JWT part One

- 테스트로 넣었던 코드를 삭제하고 requestSecret.js에서 sendSecretMail 함수를 실행하도록 추가한다.

```js
// requestSecret.js
// 코드 추가
try {
  await sendSecretMail(email, loginSecret);   // sendSecretMail 실행
  await prisma.updateUser({
    data: {
      loginSecret
    },
    where: {
      email
    }
  });
  return true;
}
```

- 메일 내용을 아주 약간 styling 해준다.

```js
// utils.js
// 코드 수정
export const sendSecretMail = (address, secret) => {
  const email = {
    from: "kdh@prismagram.com",
    to: address,
    subject: "🔒Login Secret for Prismagram🔒",
    html: `Hello! Your login secret is <strong>${secret}</strong>.<br/>Copy paste on the app/website to log in` // secret 코드를 강조했다.
  };
  return sendMail(email);
};
```

- 이제 confirmSecret 을 만든다.

```js
// api/User/confirmSecret/confirmSecret.graphql
type Mutation {
  confirmSecret(secret: String!, email: String!): String!
  // 인자로 secret과 email을 받고,  string을 리턴한다.
  // 이 함수는 jwt 토큰을 리턴할 것이다.
}

// api/User/confirmSecret/confirmSecret.js
import { prisma } from "../../../../generated/prisma-client";

export default {
  Mutation: {
    confirmSecret: async (_, args) => {
      const { email, secret } = args;   // args에서 email과 secret을 가져온다.
      const user = await prisma.user({ email }); // prisma.user 함수에 email을 인자로 입력해서 사용자를 가져온다.
      if (user.loginSecret === secret) {
        // user.loginSecret이 secret과 같다면 JWT 토큰을 리턴한다.
        // JWT
        return "TOKEN"
      } else {
        // 같지 않으면 에러를 전달한다.
        throw Error("Wrong email/secret combination");
      }
    }
  }
}
```

- playground에서 일부러 틀린 값을 입력해보자.

```js
// playground
mutation {
  confirmSecret(email:"nmwh47@gmail.com", secret:"calculating jellyyy")
}
```

- 에러를 확인할 수 있다.
- 이제 토큰을 만들어보자. 인증 토큰을 위해 passport.js를 사용할 것이다.
- passport에서 strategy라는 걸 확인한다. kakao, facebook, github 등 다양한 strategy가 있다.
- 우리가 사용하는 것은 passport-jwt 이다.
- passport와 passport-jwt를 설치한다.

`yarn add passport-jwt passport`

- passport.js 파일을 만들고 passport를 import한다.
- 그리고 옵션을 설정하고 passport.use를 실행한다.

```js
// src/passport.js
import dotenv from "dotenv";
import path from "path";
dotenv.config({path: path.resolve(__dirname, ".env")});
import passport from "passport";
import JwtStrategy from "passport-jwt";

const jwtOptions = {
  jwtFromRequest: JwtStrategy.ExtractJwt.fromAuthHeaderAsBearerToken(), // Authorization 헤더에서 jwt를 찾는 역할을 한다.
  secret: process.env.JWT_SECRET  // secret 값은 randomkeygen.com 에서 가져와서 .env에 입력했다.
};

// passport.use에서 사용할 콜백 함수
const verifyUser = (payload, done) => {
  // done은 우리가 사용자를 찾았을 때 호출해야 하는 함수이고
  // payload의 정보로 User를 찾아내는 것이다.
};

passport.use(new JwtStrategy(jwtOptions, verifyUser));
```

- passport는 인증 관련한 모든 일을 한다. jwt 토큰이나 쿠키에서 정보를 가져와서 사용자 정보에 serialize(저장)한다.
- 토큰에서 정보를 가져와서 (express의) request에 붙여주는 것이다. 토큰을 가져와서 해독한 후에 사용자 객체를 request에 추가해준다.

### #3.5 Passport JWT part Two

- verifyUser 함수를 작성해보자.

```js
// passport.js
// 코드 추가
const verifyUser = async (payload, done) => {
  try {
    const user = await prisma.user({id: payload.id}); // 사용자를 찾는다.
    if (user !== null) {
      return done(null, user);  // 사용자를 찾으면 에러가 없고(null), 우리가 찾은 user를 전달하겠다는 뜻이다.
    } else {
      return done(null, false);
    }
  } catch (error) {
    return done(error, false);  // 에러가 있으면 done(error, false) 리턴
  }
};
```

- 우리가 한 작업은 jwt를 가져와ㅏ서 해석하고 확인하는 작업들이다.
- 아직 jwt를 생성하지는 않았다. jwt를 생성하는 작업을 해보자.
- 먼저 jsonwebtoken 모듈을 설치한다.

`yarn add jsonwebtoken`

```js
// utils.js
// 코드 추가
import jwt from "jsonwebtoken";

 // sign 함수를 실행할 때 payload를 입력해야 한다. 우리는 사용자의 id를 입력할 것이고, secret key를 입력해야 한다.
export const generateToken = id => jwt.sign({ id }, process.env.JWT_SECRET);
// 암호화하고 해독할 때는 같은 private key를 사용한다.
```

- 토큰 생성 함수를 만들었다.
- 사용자 토큰 생성을 해보자.

```js
// confirmSecret.js
// 코드 수정
if (user.loginSecret === secret) {
  return generateToken(user.id);  // jwt가 id를 암호화해서 사용자 토큰을 생성한다.
} else {
  throw Error("Wrong email/secret combination");
}

// passport.js
// 코드 수정
// 코드가 잘못된 부분이 있어 수정함
import { Strategy, ExtractJwt } from "passport-jwt";

const jwtOptions = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET
};

passport.use(new Strategy(jwtOptions, verifyUser));
```

- server.js에 passport를 사용해보자.

```js
// server.js
// 코드 추가
import passport from "passport";
import "./passport";
// passport.js는 이렇게 import 해야 한다. server.js에서는 passport.js 파일에서 무언가를 받아서 사용할 필요가 없다. passport.js에서 어떤 일이 벌어지는지 몰라도 된다.

// 미들웨어를 사용한다.
// 경로를 미들웨어로 보호하고 싶을 때 사용한다.
server.express.use(passport.authenticate("jwt"));
// 모든 경로를 passport.authenticate("jwt") 로 보호했다.
```

### #3.6 Passport JWT part Three

- .env를 다른 방식으로 불러오도록 하자.

```js
// src/env.js
import dotenv from "dotenv";
import path from "path";
dotenv.config({
  path: path.resolve(__dirname, ".env")
});

// server.js
// 코드 수정
import "./env";
```

- env.js 파일을 생성하고 기존 .env를 import했던 코드를 입력한다.
- 그리고 server.js에서 해당 js 파일을 import한다.
- 마지막으로 passport.js.와 utils.js 에서 .env를 import 했던 코드를 삭제한다.

- 이제 authenticateJwt 라는 미들웨어 함수를 만들어보자. 토큰을 받아서 작업을 할 것이다.

```js
// passport.js
// 코드 추가
// 이 미들웨어가 실행되면 passport authenticate가 실행된다.
export const authenticateJwt = (req, res, next) =>
  // 이 함수에서는 passport에 어떤 것도 입력되지 않기를 원하므로, sessions: false 옵션을 추가했다.
  // passport가 함수에 사용자 정보를 전달해 줄 것이다. verifyUser 함수를 통해 사용자 정보를 전달해주는 것이다.
  passport.authenticate("jwt", { sessions: false }, (error, user) => {
    if (user) {
      // verifyUser에서 사용자를 받아온 후에, 사용자가 존재하면 그 사용자 정보를 req 객체에 붙여준다.
      req.user = user;
    }
    // express 에서는 미들웨어를 지나서 라우트가 실행된다.
    // 즉, 토큰을 받아서, 해석하고, 사용자를 찾고, 사용자가 존재한다면 req 객체에 사용자를 추가하고 나서 graphql 함수를 실행하는 것이다.
    next();
  })(req, res, next);

  passport.use(new Strategy(jwtOptions, verifyUser));
  // passport.use를 실행하고, passport.initialize를 실행해야 한다.
  passport.initialize();
```

- 마지막에 `(req, res, next)` 가 붙는 것이 이상해 보일 것이다.
- `(req, res, next)` 이전 부분은 함수이다. 즉, `fn(req, res, next);` 이런 함수를 실행해줘야 하는 것이고, 실행하고 나면 다음 함수로 넘어가는 것이다.
- `fn` 부분은 함수를 리턴하고 리턴된 함수를 `(req, res, next)`로 실행해줘야 한다.
- 이 경우에는 그 실행해야 하는 함수가 graphql 함수다.
- 이러한 방식은 ES6 문법 중 하나인 IIFE(즉시 실행 함수 표현)라는 문법이다.
- 참조: [IIFE](https://developer.mozilla.org/ko/docs/Glossary/IIFE)

- 이 함수들이 실행되는 순서는, server.js에서 `import "./passport"` 가 실행되면
먼저 initialize 가 실행되고, 그 다음 use가 실행되며, authenticate가 마지막으로 실행된다.
- 단, authenticate는 우리가 요청했을 때 실행된다.

```js
// server.js
// 코드 수정
// 기존 코드 => server.express.use(passport.authenticate("jwt"));
server.express.use(authenticateJwt);
```

- server.js 에서도 기존 코드를 삭제하고, 미들웨어 함수 authenticateJwt 를 사용하도록 수정했다.
- 이제 resolver에 전달해줘야 한다.
- resolver 사이에서 정보를 공유할 때 context를 사용한다.
- context에 prisma를 담아서 사용하는 방식은 많은 사람들이 사용하고 있는 방식 중 하나이다.
- 하지만 이 방식은 vscode가 이해하지 못하며, 자동 완성 기능도 사용할 수 없으므로, 우리는 이 방식을 사용하지는 않을 것이다.

```js
// server.js
// 코드 추가
const server = new GraphQLServer({
  schema,
  context: ({ request }) => ({ request }) // context에 request만 담긴 객체를 리턴한다.
});
```

- 위와 같이 하면 context에 request가 담겨서 전달될 것이다.
- reqeustSecret 함수에서 request 인자를 받도록 추가한다.

```js
// requestSecret.js
// 코드 추가
export default {
  Mutation: {
    requestSecret: async (_, args, { request }) => {  // request를 인자로 추가했다.
      const {
        email
      } = args;
      const loginSecret = generateSecret();
      try {
        await sendSecretMail(email, loginSecret);
        await prisma.updateUser({ data: { loginSecret }, where: { email }});
        return true;
      } catch {
        return false;
      }
    }
  }
}
```

- 서버에 전달되는 모든 요청은 authenticateJwt 함수를 통과한다.
- authenticateJwt 함수에서는 passport.authenticate('jwt') 함수를 실행하고, 이 함수는 strategy를 활용해서 jwt 토큰을 추출한다.
- 토큰이 추출되면 verifyUser를 payload와 함께 실행한다.
- payload는 토큰에서 해석된 id를 받아서, user를 찾아서 리턴한다.
- 그리고 콜백 함수가 실행되어서, 사용자가 있으면 그 사용자를 req에 추가한다.
- server.js에서는 context에 request를 담아준다.

### #3.7 toggleLike Resolver part One

- 좋아요 기능을 위해 api 폴더에 Post 폴더를 생성하고, 그 안에 toggleLike 폴더를 만들었다.
- toggleLike 는 우리에게 인증을 요구하는 첫번째 resolver가 될 것이다.

```js
// api/Like/toggleLike/toggleLike.graphql
type Mutation {
	toggleLike(postId: Strikg!): Boolean!
}

// api/Like/toggleLike/toggleLike.js
export default {
  Mutation: {
    toggleLike: async (_, args, {
      request
    }) => {
      // if (!request.user) {     -> 이러한 방식은 효율적이지 않다.
      //   throw Error('error');    왜냐하면 이 resolver 뿐만 아니라 다른 resolver들도 동일하게 예외 처리가 되어야 하기 때문이다.
      // }
    }
  }
}
```

- 효율적인 코드를 위해 middleware를 만들자.

```js
// src/middlewares.js
export const isAuthenticated = request => {
  if (!request.user) {
    throw Error("You need to log in to perform this action.");
  }
  return;
};

// toggleLike.js
import {
  isAuthenticated
} from "../../../middlewares";

export default {
  Mutation: {
    toggleLike: async (_, args, {
      request
    }) => {
      isAuthenticated(request); // request 안에 user가 없으면 모든 function이 멈출 것이다.
      const { postId } = args;
      const { user } = request;
      return true;
    }
  }
}
```

- 다음은 user가 존재(existence)하는지를 확인해야 한다.

```js
// toggleLike.js
import {
  isAuthenticated
} from "../../../middlewares";

export default {
  Mutation: {
    toggleLike: async (_, args, {
      request
    }) => {
      isAuthenticated(request); // request 안에 user가 없으면 모든 function이 멈출 것이다.
      const { postId } = args;
      const { user } = request;
      try {
        // 좋아요가 있는지 확인
        const existingLike = await prisma.$exists.like({
          AND: [{
            user: {
              id: user.id   // user.id가 있고
            }
          },
          {
            post: {
              id: postId  // postId가 있으면 좋아요가 존재하는 것임
            }
          }]
        });
        if (existingLike) {
          // existingLike가 존재하면 좋아요를 해제하는 작업이 처리되어야 한다.
          // To do
        } else {
          // exsitingLike가 없으면 좋아요를 만들어야 한다.
          const newLike = await prisma.createLike({
            user: {
              connect: {
                id: user.id
              }
            },
            post: {
              connect: {
                id: postId
              }
            }
          });
        }
        return true;
      } catch {
        return false;
      }
    }
  }
}
```

### #3.8 toggleLike and addComment Resolver

- 이제 좋아요를 취소(삭제) 기능을 구현한다.
- 먼저 바로 위 코드에서 AND 부분은 지울 수 없다. user 나 post 를 기반으로 삭제할 수 있는 방법이 없다.

```js
// toggleLike.js
// 코드 수정
const filterOptions = { // filterOptions 라는 변수에 동일한 내용을 담는다.
  AND: [{
      user: {
        id: user.id
      }
    },
    {
      post: {
        id: postId
      }
    }
  ]
};
try {
  const existingLike = await prisma.$exists.like(filterOptions);  // filterOptions를 전달해서 like를 얻는다.
  if (existingLike) {
    await prisma.deleteManyLikes(filterOptions); // filterOptions 를 전달해서 like를 찾아서 삭제한다.
  } else {
    const newLike = await prisma.createLike({
      user: {
        connect: {
          id: user.id
        }
      },
      post: {
        connect: {
          id: postId
        }
      }
    });
  }
  return true;
}
```

- deleteManyLikes 함수를 통해 like를 삭제하도록 했다.
- 이제 Comment 작업을 해보자.

```js
// api/Comment/addComment/addComment.graphql
type Mutation {
	addComment(text: String!, postId: String!): Comment!
}

// api/Comment/addComment/addComment.js
import { isAuthenticated } from "../../../middlewares";
import { prisma } from "../../../../generated/prisma-client";

export default {
  Mutation: {
    addComment: async (_, args, { request }) => {
      isAuthenticated(request);
      const { text, postId } = args;
      const { user } = request;
      const comment = await prisma.createComment({
        user: {
          connect: {
            id: user.id
          }
        },
        post: {
          connect: {
            id: postId
          }
        },
        text
      });
    }
  }
}
```

- addComment 폴더를 생성하고 addComment 관련 graphql / js 파일을 만들었다.
- prisma를 이용하여 매우 간단하고 멋지게 Comment를 만들었다.

### #3.9 searchUser and searchImage resolver

- searchUser 작업을 시작한다.
- api/User 폴더 밑에 searchUser 폴더를 생성하고, 파일을 만든다.

```js
// api/User/searchUser/searchUser.graphql
type Query {
	searchUser(term: String!): [User!]!   // User는 array 다.
}

// api/User/searchUser/searchUser.js
import { prisma } from "../../../../generated/prisma-client";

export default {
  Query: {
    searchUser: async (_, args) =>
      prisma.users({
        where: {
          OR: [
            { username_contains: args.term },
            { firstName_contains: args.term },
            { lastName_contains: args.term }
          ]
        }
      })
  }
}
```

- 위와 같이 prisma는 hard core한 검색을 쉽게 구현할 수 있다.
- 다음은 searchImage 를 해보자.
- api/Post 폴더 밑에 searchPost 폴더를 생성하고, 파일을 만든다.

```js
// api/Post/searchPost/searchPost.graphql
type Query {
	searchPost(term: String!): [Post!]!   // Posts는 array 다.
}

// api/Post/searchPost/searchPost.js
import { prisma } from "../../../generated/prisma-client";

export default {
  Query: {
    searchPost: async (_, args) => prisma.posts({
      where: {
        OR: [
          { location_starts_with: args.term },
          { caption_starts_with: args.term }
        ]
      }
    })
  }
}
```

- searchUser와 마찬가지로 쉽게 구현할 수 있다.

### #3.10 follow unfollow Resolver

- follow user를 작업해보자.
- User 폴더에 follow 폴더를 생성하고, 파일들을 만든다.

```js
// api/User/follow/follow.graphql
type Mutation {
	follow(id: String!): Boolean
}

// api/User/follow/follow.js
import { isAuthenticated } from "../../../middlewares";
import { prisma } from "../../../../generated/prisma-client";

export default {
  Mutation: {
    follow: async (_, args, { request }) => {
      isAuthenticated(request);
      const { id } = args;
      const {
        user
      } = reqeust;
      try {
        await prisma.updateUser({
          where: {
            id: user.id
          },
          data: {
            following: {
              connect: {
                id
              }
            }
          }
        })
      } catch {
        return false;
      }
    }
  }
}
```

- unfollow도 follow와 마찬가지다.

```js
// api/User/unfollow/unfollow.graphql
type Mutation {
  unfollow(id: String!): Boolean!
}

// api/User/unfollow/unfollow.js
import { isAuthenticated } from "../../../middlewares"
import { prisma } from "../../../../generated/prisma-client";

export default {
  Mutation: {
    unfollow: async (_, args, { request }) => {
      isAuthenticated(request);
      const { id } = args;
      const { user } = request;
      try {
        await prisma.updateUser({
          where: { id: user.id },
          data: {
            following: {
              disconnect: {
                id
              }
            }
          }
        })
        return true;
      } catch {
        return false;
      }
    }
  }
}
```

- HTTP HEADERS에 아래 내용 추가
```
{"Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImNrMm1tdGYza2I2OW8wYjAwazFlcTM3dXYiLCJpYXQiOjE1NzY0NjIyNjJ9.VE3duIC1HYXaaTlvRoUw-k_XTcIeb4zcc9OVNypPQFs"}
```