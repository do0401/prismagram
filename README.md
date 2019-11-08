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

- graphql 파일에 mutation만 있으면 에러가 나므로 임의의 Query도 만들었다.

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
export const sendMail = email => null;

export const sendSecretMail = (address, secret) => {
  const email = {
    from: "kdh@prismagram.com",
    to: address,
    subject: "🔒Login Secret for Prismagram🔒",
    html: `Hello! Your login secret it ${secret}.<br/>Copy paste on the app/website to log in`
  }
};
```