같은 프론트엔드 팀원과 프로젝트를 하다가 패키지 매니저에 대해 의견충돌이 있었다. 나는 `npm` 을 사용하고 있었는데 팀원이

“왜 `npm` 을 쓰는거냐, 요즘 누가 `npm` 쓰냐”

이런식으로 물어봤다. 사실 나도 처음 개발을 시작하고 공부했을때 부터 쭉 `npm` 을 사용하고 있었기 때문에 다른 패키지 매니저에 대한 고려를 해본적이 없었다. 그래서 팀원의 물음에 명확히 대답하지 못하였다. 그래서 이번기회에 패키지 매니저에 대하여 알아보려고 한다.

## 1. 패키지 매니저란?

패키지 매니저는 기본적으로 패키지를 다루는 작업을 편리하고 안전하게 수행하기 위해 사용되는 툴이다. 패키지를 다루는 작업은 다음과 같다.

- 설치
- 업데이트
- 수정
- 삭제

보통 패키지들은 서로 종속성이 있는 경우가 많다. 따라서 하나의 패키지를 동작시키기 위해서는 많은수의 종속된 패키지들을 추가로 설치해야된다. 이런 종속된 패키지들을 `dependency` 라고 한다. 당연하겠지만 이런 dependency들을 패키지 설치 할때마다 개발자가 따로 고려해서 설치를 해야한다면 여간 피곤한일이 아닐 것이다. 그래서 등장한게 바로 이 패키지 매니저인 것이다. 다시 패키지 매니저가 수행하는 일을 정리해보면

- 패키지의 dependency 관리
- 패키지의 보안 관리
- 여러 패키지를 기능에 따라 그룹으로 묶어 정리
- 패키지 압축 해제
- Software repository로부터 패키지를 찾고, 다운로드하고, 설치 및 업데이트

## 2. Javascript의 패키지 매니저

Python에서 사용하는 pip, Java에서 사용하는 Maven, Gradle등 언어마다 여러 패키지 매니저들이 존재하지만 내가 직접적으로 사용하는 Javascript(Node.js)의 패키지 매니저만 알아보려고 한다.

### 1️⃣ npm

최초의 Javascript 패키지 매니저는 2010년에 나온 `npm` 이다. 이 npm은 흔히 “Node Package Manager”의 약자로 알고 있는데 사실 `pm` 이라는 bash 유틸리티에서 유래했고 그 유틸리티의 Node버전이라고 보면 된다. 앞서 언급했던 것 처럼 이전에는 dependency를 수동으로 관리하였기 때문에 npm의 등장이 엄청난 혁명이었다. 메타데이터를 가지고 있는 `package.json` 과 같은 개념, dependency들을 `node_modules` 에 설치한다는 개념들이 모두 npm에 의해 도입되었다고 생각하면 된다.

하지만 npm은 아래와 같은 다양한 단점을 가지고 있어서 추후 다른 패키지 매니저들의 등장에 영향을 미치게 된다.

- 느린설치 : npm은 dependency를 설치할 때 동기방식으로 진행하여 오래걸린다.
- dependency 중복 : npm은 패키지를 설치할 때 중복된 패키지를 여러 개 저장한다. 따라서 같은 패키지가 여러 번 설치되면서 `node_modules` 폴더가 비대해 진다.
- 보안문제 : npm V5.7.0에서 파일시스템 권한을 바꿀 수 있는 버그가 발견된 적 있다. 사실 이러한 문제를 해결하기 위해 최신 npm은 `SHA-512` 알고리즘을 적용한다.

### 2️⃣ yarn classic

2017년 페이스북은 구글과 몇몇 개발자들과 함께 npm이 가지고 있던 문제들을 해결하기 위해 yarn(yet another resource negotiator)을 발표했다. npm과 가장 큰 차이점은 설치 프로세스의 속도를 높이기 위해 병렬 설치를 지원한다는 것이다. 또한 `yarn.lock` 파일을 도입해서 다른 개발자가 해당 코드를 그대로 복사해서 가져와서 설치해도 동일한 패키지 버전이 설치되게 만들었다. 또한 보안을 더 강화함으로써 npm보다 빠르고 안정적인 패키지 설치를 제공하게 된다. 2020년부터 1.x버전은 모두 레거시로 간주하고 yarn classic으로 이름을 바꾸고 유지보수 모드로 전환되었다.

### 3️⃣ pnpm

2017년에 만들어졌으며, npm의 drop-in replacement(설정을 바꿀 필요 없이 바로 사용가능하며, 속도와 안정성등 다양한 기능이 향상된 대체품)으로, npm만 있다면 바로 사용할 수 있다. 사실 yarn, npm모두 큰 문제점이 여전히 존재한다. 바로 프로젝트 간에 사용되는 **dependency의 중복 저장**이다. 두 패키지 매니저 모두 node_modules 내부에 flat하게 패키지를 설치하여 관리를 하기 때문에 무한으로 비대해진다.

```
A 프로젝트/
  ├── node_modules/
  │   ├── react/
B 프로젝트/
  ├── node_modules/
  │   ├── react/  (같은 패키지가 중복 저장됨)
```

pnpm은 이러한 방식 대신, `content-addressable storage` 를 사용한다. 이 방법을 사용하면 글로벌 저장소 `pnpm-store` 에 패키지를 저장하는 중첩된 node_modules 폴더가 생성된다. 따라서 모든 버전의 dependency는 해당 폴더에 물리적으로 한번만 저장되므로 디스크 공간을 상당히 절약할 수 있다.

이는 node_modules의 레이아웃을 통해 이루어지고, `symlinks`(가상의 바로가기) 를 사용하여 dependency의 중첩된 구조를 생성한다. 여기서 폴더 내부의 모든 패키지 파일은 저장소에 대한 `하드 링크`(같은 파일을 여러 위치에서 참조할 수 있도록 하는 방식)로 구성되어 있다. 요약하자면 다음과 같은 흐름이다. `node_modules/react/` → (`symlink`) → `.pnpm/react@16.13.1/` → (`hard link`) → `pnpm- store/registry.npmjs.org/react/16.13.1/`

![image.png](<../images/20250406_Package%20Manager(Chuck).png>)

### 4️⃣ yarn berry, plug n play

yarn berry는 2020년에 출시되었으며, yarn classic의 업그레이드 버전이다. 이 버전에서 주목해야할 것은 `plug n play` 로 node_modules를 수정하기 위한 전략이다. node_modules를 생성하는 대신 `.pnp.cjs` 라고 불리는 의존성 lookup파일이 생성되는데, 이는 중첩된 폴더 구조 대신 단일 파일이기 때문에 더 효율적으로 처리할 수 있다. 또한 모든 패키지는 `.yarn/cache` 폴더 내부에 zip파일로 저장되기 때문에 node_modules 폴더보다 디스크 공간을 더 적게 차지한다. 하지만 node_modules → pnp로의 변화는 너무나 빨랐기 때문에 기존 패키지들과 호환이 잘 안되는 문제점이 발생하기도 하였다. 하지만 Javascript 생태계가 시간이 지남에 따라 지원을 점점 더 제공하는 추세이기 때문에 일부 프로젝트에서는 yarn berry를 채택하기도 한다.

## 3. 결론

- [https://p.datadoghq.eu/sb/d2wdprp9uki7gfks-c562c42f4dfd0ade4885690fa719c818?fromUser=false&refresh*mode=sliding&tpl_var_npm[0]=*&tpl*var_pnpm[0]=*&tpl*var_yarn-classic[0]=*&tpl*var_yarn-modern[0]=*&tpl_var_yarn-nm[0]=\*&tpl_var_yarn-pnpm[0]=no&from_ts=1735697034145&to_ts=1743473034145&live=true](https://p.datadoghq.eu/sb/d2wdprp9uki7gfks-c562c42f4dfd0ade4885690fa719c818?fromUser=false&refresh_mode=sliding&tpl_var_npm%5B0%5D=%2A&tpl_var_pnpm%5B0%5D=%2A&tpl_var_yarn-classic%5B0%5D=%2A&tpl_var_yarn-modern%5B0%5D=%2A&tpl_var_yarn-nm%5B0%5D=%2A&tpl_var_yarn-pnpm%5B0%5D=no&from_ts=1735697034145&to_ts=1743473034145&live=true)
- https://pnpm.io/benchmarks

이 사이트들에서 각각의 패키지 매니저들을 사용했을 때 벤치마크 결과를 살펴볼 수 있다.
성능으로 살펴보면 `yarn berry, plug n play`가 가장 설치도 빠르고, 디스크 효율도 높았다.

무지성으로 npm을 썼던 과거와 달리 현재 많은 레퍼런스를 살펴보고, 트렌드를 살펴보니 더 다양한 시각을 기를 수 있었던 것 같다. 항상 느끼는거지만 프론트엔드 생태계는 정말 숨쉴틈 없이 변화하는 것 같다. 트렌드에 뒤쳐지고, 새로운 기술에 등한시하면 개발자로서의 역량도 같이 떨어지는 것을 이번 기회에 크게 체감했던 것 같다.

\*출처 : https://yceffort.kr/2022/05/npm-vs-yarn-vs-pnpm
