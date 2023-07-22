# Minimal TypeScript-Webpack-SystemJS Example

SystemJS 기반으로 구성된 micro-frontend(?) 환경에서 개별 클라이언트 사이드 자바스크립트 모듈 개발을 위한 간단한 프로젝트 구성을 설명해보기 위한 프로젝트.


## 요구사항

* TypeScript를 쓰고싶다
* Webpack을 쓰고싶다
* SystemJS 모듈로 번들링되어야 하지만, SystemJS 자체는 번들에 포함되어서는 안된다

## 프로젝트 구성

### SystemJS 모듈 방식 설정

* TS 설정(`tsconfig.json`)의 `module` 필드는 최종 빌드 결과물에 영향을 끼치지 않음. Webpack에서 `ts-loader`를 통해 TS 변환 후 번들링 과정에서 Webpack의 설정에 따라 모듈방식이 결정됨.
* Webpack 설정(`webpack.config.js`)의 `output.library.type` 필드가 최종 빌드 결과물을 결정. 여기 값을 `system`으로 설정하면 SystemJS 모듈로 빌드. Webpack은 SystemJS 자체는 외부환경에서 주입될 것으로 간주.

### 로컬 개발환경에서 SystemJS 주입

로컬 개발환경에서는 [html-webpack-plugin](https://webpack.js.org/plugins/html-webpack-plugin/)을 이용해 html 파일에 `<script/>` 태그를 통해 주입 가능.
`src/index.html` 파일 참고.

`html-webpack-plugin`은 빌드된 번들 파일을 `<script/>` 태그로 주입하는데, 기본적으로 `defer` 속성으로 지연 로드함. 
(ref: https://javascript.info/script-async-defer#defer)
하지만 SystemJS는 defer 속성을 가진 script 태그로 로드하는 스크립트 내부의 `System.register` 함수의 `execute` 선언부를 실행하지 않는 것으로 보임. 별다른 오류 메시지 없이 번들 파일 자체가 실행되지 않음. 네트워크 탭을 확인해보면 스크립트 자체를 로드하기는 함.

두가지 방법으로 회피 가능:
1. 플러그인 옵션에 `scriptLoading: "blocking"`를 추가해서 `defer` 속성을 제거.
2. html 파일의 외부 SystemJS 스크립트 태그 아래 `<script type="systemjs-module" src="bundle.js"></script>`를 추가해서 명시적으로 번들파일을 SystemJS 모듈로 로드. (여기서 `bundle.js`는 Webpack 번들 output 파일)
    * 이 경우 플러그인 옵션에 `inject: false`를 추가하면 번들 파일 스크립트 태그 주입을 막아줄 수 있음.