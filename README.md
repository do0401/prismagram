# prismagram
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

