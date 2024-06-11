# Monorepo Template

1. [프로젝트 생성](#프로젝트-생성)
2. [pnpm 설치](#pnpm-설치)
3. [폴더 구조](#폴더-구조)
4. [Workspace 설정](#workspace-설정)
5. [TypeScript 설정](#typescript-설정)
6. [ESLint, Prettier, Husky, lint-staged, commitlint](#eslint-prettier-husky-lint-staged-commitlint)
   - 6.1 [Prettier](#prettier)
   - 6.2 [ESLint](#eslint)
     - 6.2.1 [VSCode 설정](#vscode-설정)
   - 6.3 [Prettier와 ESLint 통합](#prettier와-eslint-통합)
   - 6.4 [Husky & lint-staged & commitlint](#husky--lint-staged--commitlint)
     - 6.4.1 [commitlint](#commitlint)
     - 6.4.2 [husky, lint-staged](#husky--lint-staged)
7. [예시 : package 만들기 (with. Vite)](#예시--package-만들기-with-vite)
8. [예시 : app 만들기 (with. Vite)](#예시--app-만들기-with-vite)

---

#### 1. 프로젝트 생성

```bash
mkdir pnpm-monorepo
cd pnpm-monorepo
```

#### 2. pnpm 설치

- Node 버전 v16.14 이상 필요
- 설치 (pnpm v9.3.0 사용)

```bash
# Install pnpm with your preferred method: https://pnpm.io/installation.
npm i -g pnpm

pnpm init
```

```bash
git init
echo -e "node_modules" > .gitignore
npm pkg set engines.node=">=18.17.1" // Use the same node version you installed
npm pkg set type="module"
echo "#PNPM monorepo" > README.md
```

#### 3. 폴더 구조

```
├── package.json
├── apps // 실행할 앱
├── packages // npm에 게시하고, 설치하여 사용할 패키지들
│ ├── foo
│ │ ├── package.json
│ │ ├── src
│ │ │ └── index.ts
│ │ ├── tsconfig.build.json
│ │ └── tsconfig.json
├── tsconfig.build.json
└── tsconfig.json
```

- `apps/` : 실행할 앱들. npm에 게시하거나 설치하지 않고 CRA 등을 사용해 개발하고, 실행하고, 배포하는 앱들의 모음
- `packages/` : npm에 게시하고 설치하여 사용하는 패키지들 모음

#### 4. Workspace 설정

`pnpm-workspace.yaml` 파일 생성

```
touch pnpm-workspace.yaml
```

`pnpm-workspace.yaml` 파일에 아래 내용 추가

```
packages:
  - 'apps/*'
  - 'packages/*'
```

```
mkdir apps packages
```

---

#### 5. TypeScript 설정

해당 템플릿에서는 Root에서 공통의 TypeScript 설정을 사용한다.

Root에 typescript 설치

```
pnpm add -D typescript @types/node -w
```

Root에 `tsconfig.base.json`을 추가하여 아래 내용 추가

```
// tsconfig.base.json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true,
    "esModuleInterop": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "noUnusedLocals": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "jsx": "react-jsx",
    "moduleResolution": "node"
  }
}
```

개별 프로젝트에서 `tsconfig.json`을 추가하고 아래 내용을 추가한다.

```
// 개별 프로젝트에서 tsconfig.json
{
  "extends": "./tsconfig.base.json"
}
```

---

#### 6. ESLint, Prettier, Husky, lint-staged, commitlint

#### <span style="background:#b1ffff">Prettier</span>

```bash
pnpm add -D prettier
```

`.prettierignore` 파일 생성 후, 아래 내용 추가

```
coverage
public
dist
pnpm-lock.yaml
pnpm-workspace.yaml
```

`.prettierrc` 파일 생성 후, 아래 내용 추가 (프로젝트에 맞게 수정)

```
{
  "semi": true,
  "singleQuote": true
}
```

##### VSCode 설정

1. VSCode의 `Prettier - Code formatter` 플러그인 설치
2. 파일 저장 시, 자동으로 Workspace 내의 코드가 포멧팅 되도록 설정

```
mkdir .vscode && touch .vscode/settings.json
```

```
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

---

#### <span style="background:#affad1">ESLint</span>

```bash
pnpm create @eslint/config
```

```
✔ How would you like to use ESLint? · problems
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · react
✔ The React plugin doesn't officially support ESLint v9 yet. What would you like to do? · 8.x
✔ Does your project use TypeScript? · typescript
✔ Where does your code run? · browser
The config that you've selected requires the following dependencies:

eslint@8.x, globals, @eslint/js, typescript-eslint, eslint-plugin-react
✔ Would you like to install them now? · No / Yes
✔ Which package manager do you want to use? · pnpm
```

`.eslintignore` 파일 생성 후, 아래 내용 추가

```
coverage
public
dist
pnpm-lock.yaml
pnpm-workspace.yaml
```

`.eslintrc.cjs` 파일 생성 후, 아래 내용 추가

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
  ],
  overrides: [
    {
      env: {
        node: true,
      },
      files: ['.eslintrc.{js,cjs}'],
      parserOptions: {
        sourceType: 'script',
      },
    },
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint'],
  rules: {},
};
```

---

#### Prettier와 ESLint 통합

ESLint에는 코드 퀄리티 관련 규칙과 스타일 관련 규칙이 있다. Prettier를 사용하는 경우, ESLint의 스타일 규칙과 충돌해 문제가 발생할 수 있기 때문에 이를 방지하기 위한 플러그인들을 추가적으로 설치한다.

```
pnpm add -D eslint-config-prettier eslint-plugin-prettier
```

`.eslintrc.js`의 extends의 마지막에 아래 내용을 추가한다.

```
module.exports = {
  extends: [..., 'plugin:prettier/recommended'],
}
```

linter와 prettier를 실행할 script를 package.json에 추가한다.

```
  pnpm pkg set scripts.lint="eslint ."
  pnpm pkg set scripts.format="prettier --write ."
```

혹은 package.json에 아래와 같이 직접 추가해도 된다.

```json
"scripts": {
	"lint": "eslint .",
	"format": "prettier --write ."
},
```

아래 명령어를 실행하여 linter와 prettier가 잘 설정되었는 지 확인한다.

```bash
pnpm lint // linter 실행 확인
pnpm format // prettier 실행 확인
```

---

#### <span style="background:#fdbfff">Husky & lint-staged & commitlint</span>

##### commitlint

커밋 메세지 컨벤션을 위한 [commitlint](https://commitlint.js.org/) 설치

```
  pnpm add -D @commitlint/cli @commitlint/config-conventional
```

`commitlint.config.js` 파일을 생성하고, 아래 내용을 추가한다.

```js
export default { extends: ['@commitlint/config-conventional'] };
```

- `@commitlint/config-conventional` 규칙
  - 위 확장을 사용하면 아래와 같은 규칙을 적용한다.
    - **type-enum**: 커밋 타입이 `feat`, `fix`, `docs` 등으로 시작해야 합니다.
    * **type-case**: 커밋 타입은 소문자여야 합니다.
    * **type-empty**: 커밋 타입은 비어 있으면 안 됩니다.
    * **subject-empty**: 커밋 제목은 비어 있으면 안 됩니다.
    * **header-max-length**: 커밋 메시지의 헤더는 100자를 넘으면 안 됩니다.

##### husky, lint-staged

```bash
pnpm add -D husky -w // `-w` 플래그는 루트 워크스페이스 디렉토리에서 명령을 실행
pnpm add -D lint-staged -w
pnpm exec husky init
```

`.husky/pre-commit` 파일 생성 후, 아래 내용 추가

```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
```

`.husky/commit-msg` 파일 생성 후, 아래 내용 추가

```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx --no -- commitlint --edit ${1}

if [ $? -eq 0 ]; then
	echo "🚀 커밋 메시지가 규칙에 맞게 작성되었습니다! 🚀"
	exit 0
fi
```

package.json에 prepare script 추가

```json
"scripts": {
	"postinstall": "npx mrm lint-staged",
	"prepare": "husky i"
},
```

package.json에 아래 속성 추가

```json
 "lint-staged": {
    "**/*.{js,ts,tsx}": [
      "eslint --fix"
    ],
  },
```

---

### 7. 예시 : package 만들기 (with. Vite)

`/packages`에 들어가는 예시 pacakge 프로젝트 생성하기

```
cd packages
pnpm create vite common --template vanilla-ts
cd common
pnpm i

# common 패키지에만 명령을 실행하도록 필터링하는 스크립트를 추가
npm pkg set scripts.common="pnpm --filter common"
```

Root에서 특정 패키지를 바로 실행할 수 있도록, package.json에 필터링 스크립트를 추가한다.

```
# Root의 package.json
 "scripts": {     
	 "common": "pnpm --filter common",
 }
```

`src/main.ts`에 아래 예시 코드 추가

```js
export const getCommonText = () => 'common 텍스트입니다.';
```

기본적으로 Vite는 App 모드로 에셋(JS 파일, CSS 파일, 이미지 등)을 빌드하고, index.html을 엔트리 파일로 사용한다. 하지만 지금과 같이 package를 빌드하는 경우에는 main.ts 파일이 엔트리 파일로 노출되길 원한다. 이러한 설정을 `vite.config.ts`에서 할 수 있다.

먼저 라이브러리에서 타입 정의를 자동 생성하는 Vite 패키지를 설치한다.

```
pnpm common add -D vite-plugin-dts
```

`vite.config.ts` 생성

```
 touch vite.config.ts
```

```js
import { defineConfig } from 'vite';
import { resolve } from 'path';
import dts from 'vite-plugin-dts';

// https://vitejs.dev/config/
export default defineConfig({
  build: { lib: { entry: resolve(__dirname, 'src/main.ts'), formats: ['es'] } },
  resolve: { alias: { src: resolve('src/') } },
  plugins: [dts()],
});
```

`common-components/package.json`에 아래 내용을 추가한다.

```
 "main": "./dist/common.js",
 "types": "./dist/main.d.ts",
```

모노레포 안의 다른 앱들이 `common`패키지를 찾을 수 있도록, 앱을 빌드

```
pnpm common build
```

---

#### 8. 예시 : app 만들기 (with. Vite)

```
cd apps
pnpm create vite sample-app --template react-ts
cd sample-app
pnpm i
npm pkg set scripts.app="pnpm --filter sample-app"
```

Root에서 특정 패키지를 바로 실행할 수 있도록, package.json에 필터링 스크립트를 추가한다.

```
# Root의 package.json
 "scripts": {     
	 "sample-app": "pnpm --filter sample-app",
 }
```

예시로 추가한 `common` 패키지 설치하기

```
// package.json
"dependencies": {
 "common": "workspace:*",
 ...,
 }
```

`sample-app`이 common 패키지에 대한 심볼릭 링크를 생성할 수 있도록 의존성 설치

```
pnpm i
```

`App.tsx`에서 common 패키지를 테스트한다.

```tsx
import { getCommonText } from 'common';
import './App.css';

function App() {
  return (
    <>
      <div>{getCommonText()}</div>
    </>
  );
}

export default App;
```

sample-app 앱 실행

```
pnpm app dev
```

