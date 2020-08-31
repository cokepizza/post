# SEO

SEO의 배경과 GoogleBot의 동작원리, SEO 디버깅 테스트 방법을 정리합니다.

----

## Table of contents

- [Background](#Background)  
  - [What is a Search Engine](#What-is-a-Search-Engine)
  - [Operation process of Search Engine](#Operation-process-of-Search-Engine)
  - [From SSR to CSR](#From-SSR-to-CSR)
- [Googlebot MythBusting](#Googlebot-MythBusting)  
  - [Googlebot doesn't understand modern Javascript](#Googlebot-doesn't-understand-modern-Javascript)
  - [It takes a long time to render pages](#It-takes-a-long-time-to-render-pages)
  - [JavaScript is bad for SEO](#JavaScript-is-bad-for-SEO)
- [Debug tools for Search](#Debug-tools-for-Search)
  - [Google Console](#Google-Console)
- [SEO in React](#SEO-in-React)  
- [Reference](#Reference)  

## Background

### What is a Search Engine
  
  검색엔진이란 일종의 웹 로봇입니다.

### Operation process of Search Engine

<br />
<br />

### From SSR to CSR
  
<br />

  ![99763D4B5C99CB6514](https://user-images.githubusercontent.com/56418546/91708493-5de6b080-ebbc-11ea-9bcd-2f50003b1ad6.png)

- 전통적인 웹사이트에서는 Sever Side Rendering을 통해 Templete과 Data를 합쳐서 하나의 완성된 HTML markup 문서를 던져주었습니다.
  - 페이지마다 하나의 HTML 문서가 있었습니다.
  - 그래서 크롤러가 HTML만 잘 파싱하면 문제가 되지 않았습니다.
    - 크롤링에 필요한 critical한 컨텐츠가 모두 마크업에 존재했습니다.
    - JS는 보통 애니메이션 효과 등과 같은 user interaction에 주로 사용되었습니다.
    - 첫 로딩이 빠르지만, 페이지 이동이 느리고 page reolad가 일어납니다.
  
- 현대의 웹사이트는 Client Side Rendering을 통해서 Template(HTML)을 던져주고 Ajax를 기반으로 Data를 fetching하여 페이지를 완성하는 식으로 동작합니다. (실제로는 CSR의 단점을 상쇄하기 위해 SSR, CSR의 혼합형태도 자주 사용됩니다.)
  - 그렇기 때문에 하나의 HTML문서를 가지고 JS를 이용해 다양한 페이지를 만들어냅니다(Single Page Application)
  - React, Vue 등의 프론트엔드 프레임워크들이 CSR 기반입니다.
    - 실제 배포에서는 하나의 빈 HTML과 babel을 통해 transpiling하고 webpack을 통해 bundling하고 난독화하고 압축한 JS를 제공합니다.
  - 그래서 JS를 실행시켜 페이지를 렌더링 하지 않는 한 크롤러는 어떠한 정보도 얻을 수 없습니다.
  - 첫 로딩이 느리고 SEO 문제가 발생합니다. 서버에서 새롭게 페이지를 받아오지 않고 데이터만 ajax를 통해 받아오므로 서버 부담이 적고, 페이지 이동이 빨라 UX가 향상됩니다.
  
<br />
<br />

이제 부터가 영상의 내용입니다.  
요약하자면, CSR에서 문제가 되었던 SEO는 googlebot에서는 크게 문제가 되지 않습니다.  

<br />
<br />

## Googlebot MythBusting
  
----

### Googlebot doesn't understand modern Javascript

<br />

<img width="600" alt="evergreen googlebot" src="https://user-images.githubusercontent.com/56418546/91687396-8fe71b00-eb9a-11ea-8c1e-09657be82b99.png">

- 구글은 19년도에 이미 에버그린 구글봇을 출시했습니다. (2015년 크롬 41 기반 구글봇 => 2019년 에버그린 구글봇)
  - 에버그린이란 ongoing update를 의미합니다. 한번에 몰아서 업데이트가 이뤄지기 보다는 1~2주를 텀으로 지속적으로 업데이트가 이뤄집니다.
  - 이런 지속적인 업데이트가 주는 이점은 사람들이 업데이트가 늦은 googlebot을 위한 대체 버전을 만들지 않아도 된다는 점입니다.
  - 출처 : [Everything you need to know about Google's Evergreen Googlebot](https://www.youtube.com/watch?time_continue=89&v=Wkc67ETypNo&feature=emb_logo)
- 최신 자바스크립트 엔진을 탑재하고 있기 때문에 ES6 이상을 지원합니다.
  - babel을 통해 신형 js 문법을 구형 js 문법으로 낮춰주는 과정이 필요없습니다.
- 별도의 polyfill이 필요하지 않습니다.
  - polyfill을 통해 신형 js 문법에서 생긴 객체(promise 등), 메서드(Object.assign 등)에 대응하는 구형 js 코드를 채워줄 필요가 없습니다.
- Intersection Observer와 같은 새로운 웹 API를 지원해줍니다.
  - Lazy Loading은 컨텐츠 전체를 다 보여줄 필요가 없을 때, Viewport에 노출된 부분만 컨텐츠를 로딩하는 식으로 구현이 됩니다.
  - 만약 scroll 이벤트를 통해 Lazy Loading을 구현하게 되면, 구글봇이 scroll 이벤트를 체크하지 않아 크롤링이 대상이 될 수 없습니다.
  - 반면 Intersection Observer로 구현하게 되면, 구글봇의 크롤링이 가능해집니다.

  bad-lazyload.js

  ```js
    window.addEventListener('scroll', debounce(function lazyLoad() {
      var maxOffset = window.pageYOffset + window.innerHeight;
      lazyLoadedImgs.filter(function (img) {
        return !img.dataset.done && img.offsetTop < maxOffset;
      }).forEach(function (img) {
        img.src = img.dataset.src;
        img.dataset.done = true;
      });
    });  
  ```

  good-lazyload.js
  
  ```js
    document.addEventListener('DOMContentLoaded', function () {
      var observer = new IntersectionObserver(lazyLoad, {
        threshold: 0.5
      });

      var images = document.querySelectorAll('img');
      for(var i=0; i<images.length; i++) {
        observer.observer(images[i]);
      })
    });
  ```

  - 출처 : [Google Search and JavaScript Sites (Google I/O'19)](https://www.youtube.com/watch?v=Ey0N1Ry0BPM&feature=emb_logo)

<br />
<br />

### It takes a long time to render pages

<br />

  ![googlebot-process](https://user-images.githubusercontent.com/56418546/91688120-3da6f980-eb9c-11ea-9174-93d9a6c4c1ab.png)

- 보통은 렌더링과 크롤링을 합해 5초 정도밖에 걸리지 않습니다.
- 구글 검색엔진이 동작하는 방식은 다음과 같습니다.
  - Crawling Stage (General, first wave of indexing)
    - Crawl 큐에 쌓여있는 URL을 가져와서 크롤링을 시작합니다.
    - http 웹 트랜젝션을 통해 HTML문서를 받은 후 파싱하여 인덱싱합니다.
    - 링크를 찾아내게 되면 이걸 다음 크롤링을 위해 Crawl 큐에 쌓아둡니다.
  - Rendering Stage (Extra, second wave of indexing)
    - Crawling Stage 이후에 해당 페이지를 Render 큐에 쌓아둡니다.
    - 해당 작업은 GUI 환경에서 HTML, JS 등을 순차적으로 모두 받고 렌더링하는 시간과 동일합니다.
    - 구글봇은 CLI 환경인 Headless Chrome(rendertron)을 이용해 자바스크립트를 실행한 페이지를 렌더합니다.
    - 이 작업은 JS를 다운받고, 실행하고, 렌더하는 과정을 거쳐야하는 무거운 작업이기 때문에 순서대로 처리 되지 않고, 뒤로 연기됩니다.
    - 이후 작업이 완료되면 렌더된 페이지를 통해 다시 한번 인덱싱을 진행합니다.
- 출처: [How Google Search indexes Javascript sites](https://www.youtube.com/watch?v=LXF8bM4g-J4&list=PLKoqnv2vTMUPOalM1zuWDP9OQl851WMM9)


<br />
<br />

### JavaScript is bad for SEO

<br />

- 구글봇에게는 아무 문제가 되지 않지만, 일부 검색엔진 봇에게는 문제가 될 수도 있습니다.
  - React의 next.js나 Vue의 nuxt.js 같은 SSR을 고려해볼 수 있습니다.
    - 실제로는 CSR와 SSR의 장점을 취했다고 볼 수 있습니다.
    - 주소표시창을 통한 접근이나 링크를 통한 첫 접근은 SSR로 제공을 하고, 이외에 페이지 이동은 CSR로 제공합니다.
  - Dynamic Rendering 을 이용해 구글봇이 접근했을 때만 서버에서 미리 렌더링한 html을 제공하는 방법도 있습니다.
    ![how-dynamic-rendering-works](https://user-images.githubusercontent.com/56418546/91722871-dc9c1780-ebd5-11ea-9d9f-e0f1933a71c7.png)
    - user agent를 통해 크롤러 여부를 감지 합니다.
    - 크롤러일 경우 puppeteer와 같은 headless chrome을 통해 pre-rendered된 static html을 제공합니다.
    - 출처: [Dynamic Rendering for Javascipt web apps]
    (https://www.youtube.com/watch?time_continue=1&v=CrzUP6MmBW4&feature=emb_logo)

<br />
<br />
<br />

## Debug tools for Search

----

### Google Console

### Mobile Friendly Test

### 

<br />


## SEO in React

----

* code splitting
* hydrate


## Reference

- Google web.dev live 2020 Debugging Javascript Seo Issues: <https://www.youtube.com/watch?v=himvKu12YCY&list=PLNYkxOF6rcIDC0-BiwSL52yQ0n9rNozaF&index=9&t=0s>
- Google Web Master Series 1~8: <https://www.youtube.com/watch?v=LXF8bM4g-J4&list=PLKoqnv2vTMUPOalM1zuWDP9OQl851WMM9&index=1>

----
