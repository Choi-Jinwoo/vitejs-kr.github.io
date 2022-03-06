# Vite를 사용해야 하는 이유 {#why-vite}

## 이런 문제점이 있었어요 {#the-problems}

브라우저에서 ESM(ES Modules)을 지원하기 전까지는, JavaScript 모듈화를 네이티브 레벨에서 진행할 수 없었습니다. 따라서 개발자들은 "번들링(Bundling)\*"이라는 우회적인 방법을 사용할 수 밖에 없었죠. (\* 번들링: 모듈화된 소스 코드를 브라우저에서 실행할 수 있는 파일로 한데 묶어 연결해주는 작업)

[Webpack](https://webpack.js.org/), [Rollup](https://rollupjs.org) 그리고 [Parcel](https://parceljs.org/)과 같은 도구는 이런 번들링 작업을 진행해줌으로써 프런트엔드 개발자의 생산성을 크게 향상시켰습니다.

하지만, 1000개의 JavaScript 모듈이 있는 거대한 프로젝트라면 어떨까요? 실제로 이러한 상황을 어렵지 않게 마주할 수 있으며, 이 경우 Webpack과 같은 JavaScript 기반의 도구는 병목 현상이 발생하곤 합니다. (개발 서버를 실행하는 데 있어 비합리적으로 긴 시간을 기다려 본 적이 있나요? 또는 HMR을 사용할 때 편집한 코드가 브라우저에 반영되기까지 수 초 이상이 소요되어 답답했던 적이 있나요?)

vite는 이러한 것에 초점을 맞춰, 브라우저에서 지원하는 ES Modules(ESM) 및 네이티브 언어로 작성된 JavaScript 도구 등을 활용해 문제를 해결하고자 합니다.

### 지루할 정도로 길었던 서버 구동 {#slow-server-start}

Cold-Starting\* 방식으로 개발 서버를 구동할 때, 번들러 기반의 도구의 경우 애플리케이션 내 모든 소스 코드에 대해 크롤링 및 빌드 작업을 마쳐야지만이 실제 페이지를 제공할 수 있습니다. (\* Cold-Starting: 최초로 실행되어 이전에 캐싱한 데이터가 없는 경우를 의미)

vite는 이 문제를 **dependencies** 그리고 **source code** 두 가지 카테고리로 나누어 개발 서버를 시작하도록 함으로써 해결했죠.

- **Dependencies**: 개발 시 그 내용이 바뀌지 않을 일반적인(Plain) JavaScript 소스 코드입니다. 기존 번들러로는 컴포넌트 라이브러리와 같이 몇 백 개의 JavaScript 모듈을 갖고 있는 매우 큰 디펜던시에 대한 번들링 과정이 매우 비효율적이었고 많은 시간을 필요로 했습니다.

  Vite의 [사전 번들링](./dep-pre-bundling) 기능은 [Esbuild](https://esbuild.github.io/)를 사용하고 있습니다. Go로 작성된 Esbuild는 Webpack, Parcel과 같은 기존의 번들러 대비 10-100배 빠른 번들링 속도를 보였죠.

- **Source code**: JSX, CSS 또는 Vue/Svelte 컴포넌트와 같이 컴파일링이 필요하고, 수정 또한 매우 잦은 Non-plain JavaScript 소스 코드는 어떻게 할까요? (물론 이들 역시 특정 시점에서 모두 불러올 필요는 없습니다.)

  vite는 [Native ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)을 이용해 소스 코드를 제공하도록 하고 있습니다. 다시말해, 브라우저가 곧 번들러라는 말이죠. vite는 그저 브라우저의 판단 아래 특정 모듈에 대한 소스 코드를 요청하면 이를 전달할 뿐입니다.

  ![번들러 기반의 개발 서버](/images/bundler.png)

  ![ESM 기반의 개발 서버](/images/esm.png)

### 느렸던 소스 코드 갱신 {#slow-updates}

기존의 번들러 기반으로 개발을 진행할 때, 소스 코드를 업데이트 하게 되면 번들링 과정을 다시 거쳐야 했었습니다. 따라서 서비스가 커질수록 소스 코드 갱신 시간 또한 선형적으로 증가할 수 밖에 없었죠.

일부 번들러의 경우 메모리 상에서 이를 진행하여 실제로 갱신에 영향을 받는 파일들만을 새로이 번들링하게끔 하였으나, 결국 이 역시 처음에는 모든 파일에 대한 번들링을 진행해야 했었죠. "모든 파일"을 번들링 하고, 이를 다시 웹 페이지에서 불러오는 것이 얼마나 비효율적인 것인지 느껴지시나요? 이러한 이슈를 우회하고자 HMR(Hot Module Replacement)\* 이라는 대안이 나왔으나, 이 역시 명확한 해답은 아니었습니다. (\* HMR: 앱을 종료하지 않고 갱신된 파일만을 교체하는(Replacement) 방식. 다만 마찬가지로 앱 사이즈가 커질수록 선형적으로 갱신에 필요한 시간이 증가한다.)

물론, vite는 HMR을 지원합니다. 번들러가 아닌 ESM을 이용해서 말이죠. 어떤 모듈이 수정되면 vite는 그저 수정된 모듈과 관련된 부분만을 교체할 뿐이고, 브라우저에서 해당 모듈을 요청하면 교체된 모듈을 전달할 뿐입니다. 전 과정에서 완벽하게 ESM을 이용하기에, 앱 사이즈가 커져도 HMR을 포함한 갱신 시간에는 영향을 끼치지 않습니다.

또한 vite는 HTTP 헤더를 이용해 퍼포먼스를 한 단계 높였습니다. 필요에 따라 소스 코드는 `304 Not Modified`로, 디펜던시는 `Cache-Control: max-age=31536000,immutable`을 이용해 캐시되도록 함으로써, 한 번의 요청이라도 덜 하도록 말이죠.

이렇게나 빠른 Vite를 사용하지 않을 이유가 있나요?

## 배포 시 번들링 과정이 필요한 이유 {#why-bundle-for-production}

이렇게나 다양한 이점을 가진 ESM 이지만, 비교적 최근에야 지원되기 시작했기에 실제 배포 시 사용이 불가능한 브라우저 환경이 있을 수 있습니다. 또한 네트워크를 통해 필요한 시점에 해당 모듈을 요청하기에, HTTP/2를 이용하더라도 오버헤드가 발생될 수 있구요.

따라서, 모든 사용자 경험을 충족시키기 위해서라도 배포 시에는 번들링 과정이 필수적입니다. 이것이 바로 vite가 미리 설정된 [빌드 커맨드](./build)를 이용하고, [빌드 퍼포먼스 최적화](./features#build-optimizations)를 진행하는 이유입니다.

## 왜 번들링 시에는 Esbuild를 사용하지 않나요? {#why-not-bundle-with-esbuild}

`Esbuild`는 굉장히 빠른 속도로 번들링이 가능하다는 장점이 있으나, 번들링에 필수적으로 요구되는 기능인 코드 분할(Code-splitting) 및 CSS와 관련된 처리가 아직 미비합니다. Vite에서 사용중인 Rollup은 이에 대해 조금 더 검정되었고 유연한 처리가 가능하게끔 구현되어 있기에 현재로서는 이를 사용하고 있으며, 향후 `Esbuild`가 안정화 되었을 때 어떤 프로덕션 번들링 도구가 적절할 것인지 다시 논의할 예정입니다.

## Vite와 다른 도구의 차이점 {#how-is-vite-different-from-x}

[다른 빌드 도구와의 차이점](./comparisons) 섹션에서 Vite와 다른 번들러 도구의 차이점에 대해 자세히 다루고 있습니다.