# [Managing Your Typescript Monorepo With Lerna and Codefresh](https://codefresh.io/howtos/lerna-monorepo/)

## 프로젝트 셋업

| no  | 구분                                               | 설명 |
| --- | -------------------------------------------------- | ---- |
| 1   | $ git init lerna-tutorial-1 && cd lerna-tutorial-1 |      |
| 2   | $ echo “node_modules” > .gitignor                  |      |
| 3   | $ lerna init                                       |      |
| 4   | $ yarn add lerna -D -W                             |      |
| 5   | $ touch lerna.json                                 |      |
| 6   | $ touch package.json                               |      |

lerna.json

```json
{
  "packages": ["shared/**", "apps/**"],
  "version": "independent",
  "npmClient": "yarn",
  "useWorkspaces": true
}
```

package.json

```json
{
  "name": "root",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "workspaces": ["packages/*"],
  "scripts": {
    "start": "lerna exec yarn start",
    "test": "lerna run test --parallel",
    "boot": "yarn globla add lerna && lerna bootstrap",
    "release": "yarn install && lerna publish && yarn clean"
  },
  "devDependencies": {
    "lerna": "^4.0.0",
    "typescript": "^4.3.5"
  },
  "dependencies": {}
}
```

## typescript-project 패키지 만들기

| no  | 구분                              | 설명   |
| --- | --------------------------------- | ------ |
| 1   | $ lerna create typescript-project |        |
| 2   | $ cd typescript-project           |        |
| 3   | $ touch package.json              |        |
| 4   | $ touch tsconfig.json             |        |
| 5   | $ yarn add -W jest typescript     |        |
| 6   | $ mkdir src && cd src             |        |
| 7   | $ touch typescript-package.ts     |        |
| 8   | $ cd lerna-tutorial-1             |        |
| 9   | $ lerna run start                 | output |

packages/typescript-project/package.json

```json
{
  "name": "typescript-project",
  "version": "1.0.0",
  "description": "This is the typescript package of the Mono-Repo",
  "author": "anais-codefresh <anais.urlichs@codefresh.io>",
  "homepage": "",
  "license": "ISC",
  "main": "lib/typescript-package.js",
  "directories": {
    "lib": "lib",
    "test": "__tests__"
  },
  "files": ["lib"],
  "scripts": {
    "start": "tsc",
    "test": "jest"
  },
  "dependencies": {
    "jest": "^27.0.6",
    "typescript": "^4.3.5"
  }
}
```

packages/typescript-project/tsconfig.json

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "./lib"
  },
  "include": ["./src"]
}
```

packages/typescript-project/src/typescript-package.ts

```tsx
const printName = (firstName, lastName) => {
  const fullName = firstName + ' ' + lastName;
  console.log(fullName);
  return fullName;
};
module.exports = printName;
```

output

```sh
➜  lerna-tutorial-1 (main) ✗ lerna run start
info cli using local version of lerna
lerna notice cli v4.0.0
lerna info versioning independent
lerna info Executing command in 1 package: "yarn run start"
lerna info run Ran npm script 'start' in 'typescript-package' in 5.1s:
yarn run v1.22.11
$ tsc
Done in 4.87s.
lerna success run Ran npm script 'start' in 1 package in 5.1s:
lerna success - typescript-package
➜  lerna-tutorial-1 (main) ✗
```

## react-package 패키지 만들기

| no  | 구분                                                         | 설명           |
| --- | ------------------------------------------------------------ | -------------- |
| 1   | $ cd packages                                                |                |
| 2   | $ yarn create react-app react-package \--template typescript |                |
| 3   | $ cd react-package                                           |                |
| 4   | $ yarn start                                                 | localhost:3000 |

## typescript-project 에서 react-package의 data에 access 하는 방법

| no  | 구분                                                  | 설명                                                                                        |
| --- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 1   | $ touch typescript-package.ts                         | typescript-project/src/                                                                     |
| 2   | $ lerna add typescript-project \--scope=react-package | 이 명령을 하면, Lerna 에게 react-package 는 typescript-project 를 의존한다고 알리는 것이다. |
| 3   | $ touch App.tsx                                       | 업데이트                                                                                    |
|     | const name                                            | require(‘typescript-project’)                                                               |
|     | const variable                                        | name(“Hanna”, “Baum”)                                                                       |
| 4   | $ cd react-package                                    |                                                                                             |
| 5   | $ yarn start                                          | display “Hanna Baum”                                                                        |

packages/typescript-project/src/typescript-package.ts

```tsx
const printName = (firstName, lastName) => {
  const fullName = firstName + ' ' + lastName;
  console.log(fullName);
  return fullName;
};
module.exports = printName;
```

packages/react-package/App.tsx

```tsx
import React from 'react';
import logo from './logo.svg';
import './App.css';

const name = require('typescript-project');

function App() {
  const variable = name('Hanna', 'Baum');

  return (
    <div className='App'>
      <header className='App-header'>
        <img src={logo} className='App-logo' alt='logo' />
        <p>
          Edit <code>src/App.tsx</code> and save to reload.
        </p>
        <a
          className='App-link'
          href='https://reactjs.org'
          target='_blank'
          rel='noopener noreferrer'
        >
          Learn React
        </a>
      </header>
      <h2>{variable}</h2>
    </div>
  );
}

export default App;
```

## Testing

지금까지는 CRA 로 셋업한 boilerplate React App 의 파일을 테스트했지만, Typescript app 도 테스트하기를 바랄 것이다.

따라서, jest 를 typescript-project 의 package.json 파일에 추가할 것이다.

| no  | 구분                                         | 설명                    |
| --- | -------------------------------------------- | ----------------------- |
| 1   | $ lerna add jest \--scope=typescript-project |                         |
| 2   | $ touch package.json                         | typescript-project 폴더 |
|     | test                                         | jest                    |

packages/typescript-project/package.json

```json
{
  "name": "typescript-project",
  "version": "1.0.0",
  "description": "This is the typescript package of the Mono-Repo",
  "author": "anais-codefresh <anais.urlichs@codefresh.io>",
  "homepage": "",
  "license": "ISC",
  "main": "lib/typescript-package.js",
  "directories": {
    "lib": "lib",
    "test": "__tests__"
  },
  "files": ["lib"],
  "scripts": {
    "start": "tsc",
    "test": "jest"
  },
  "dependencies": {
    "jest": "^27.0.6",
    "typescript": "^4.3.5"
  }
}
```

## 의존성들을 설치하고, 프로젝트들을 테스트하고 시작할 수 있는 스크립트들을 추가하기

이제 2개의 프로젝트들을 실행하고 테스트할 수 있게 되었다.

- 2개의 패키지들에서 동시에 테스트를 할 수 있게 하기를 바랄 것이다.
- 일단 우리의 monorepo 에서 좀 더 많은 프로젝트들이 만들어지면, 전체 패키지들에서 동일한 명령을 여러 번 하지 않고, 순서대로 테스트하는 것도 점점 시간이 걸리게 될 것이다.
- 따라서, test packages 도 병행적으로 하도록 해야 한다.
- 당신의 root 폴더의 package.json 을 다음과 같은 스크립트를 추가한다.

| no  | 구분                                   | 설명                                                                         |
| --- | -------------------------------------- | ---------------------------------------------------------------------------- |
| 1   | $ touch package.json                   | root                                                                         |
|     | test                                   | "lerna run test --parallel"                                                  |
|     |                                        | \--parallel 옵션을 사용하면, 모든 패키지에 있는 test 를 병행적으로 실행한다. |
|     | start                                  | lerna exec yarn start                                                        |
|     | boot                                   | yarn global add lerna && lerna bootstrap                                     |
|     | release                                | yarn install && lerna publish && yarn clean                                  |
|     | [참조](https://github.com/lerna/lerna) |                                                                              |
| 2   | $ yarn test                            | root                                                                         |

package.json

```json
{
  "name": "root",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "workspaces": ["packages/*"],
  "scripts": {
    "start": "lerna exec yarn start",
    "test": "lerna run test --parallel",
    "boot": "yarn globla add lerna && lerna bootstrap",
    "release": "yarn install && lerna publish && yarn clean"
  },
  "devDependencies": {
    "lerna": "^4.0.0",
    "typescript": "^4.3.5"
  },
  "dependencies": {}
}
```

## 의존성들을 설치하고 업데이트하는 명령어들

| no  | 구분                                                  | 설명                                                                           |
| --- | ----------------------------------------------------- | ------------------------------------------------------------------------------ |
| 1   | $ lerna add package-A \--scope=package-B              | package-B 에 대한 의존성으로 package-A를 추가한다.                             |
| 2   | $ lerna add external dependency \--scope-package-name | 특정 패키지에 외부 의존성을 add 한다.                                          |
| 3   | $ lerna add package-name                              | root 폴더의 package.json 을 포함하여 모든 패키지들에 a dependency 를 add 한다. |

## Codefresh 사용하기

codefresh 를 사용하면, Docker images 를 만들고, 그 이미지들을 Docker registries 에 push 할 수 있다.

- 전용 쿠버네티스 대시보드와 combine 된 codefresh 는 microservice 개발에 대한 one-stop-shop 이다.

codefresh pipeline 을 셋업하여, lerna 를 사용하는 방법을 알아보자.

- codefresh pipeline 을 사용하면, monorepo 의 변경들을 Docker Hub 에 자동으로 push 할 수 있다.

| no  | 구분                       | 설명                      |
| --- | -------------------------- | ------------------------- |
| 1   | $ Dockerfile               | root 에서 Dockerfile 생성 |
| 2   | $ touch .dockerignore      | [참조]()                  |
| 3   | $ docker build -t .        | Dockerfile 로 build       |
| 4   | $ docker run -d -p 3000:80 | 작동 확인                 |
| 5   |                            |                           |

Dockerfile

```tsx
# Pull official base image
FROM node:14.9.0 as build-deps

# A directory within the virtualized Docker environment
# Becomes more relevant when using Docker Compose later
WORKDIR /usr/src/app

# Install lerna globally using npm
RUN npm i lerna -g

# Copy your package
COPY packages/react-package ./packages/react-package
COPY packages/typescript-project ./packages/typescript-project

# Copy package.json, yarn.lock and lerna.json to Docker environment
COPY package.json yarn.lock lerna.json ./

# Copy everything over to Docker environment
COPY . ./

# Install all node packages
RUN cd ./packages/react-packate && npm run build

# the base image for this is an alpine based nginx image
FROM nginx:1.19-alpine

# Copy the build folder from react to the root of nginx (www)
COPY --from=build-deps /usr/src/app/packages/react-package/build /user/share/nginx/html

# expose port 80 to the outer world
EXPOSE 80

# start nginx
CMD ["nginx", "-g", "daemon off;"]
```

.dockerignore

```tsx
node_modules;
build.dockerignore.gitignore;
Dockerfile;
Dockerfile.prod.git;
```
