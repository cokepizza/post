# Debugging Javascript SEO issues

검색엔진의 기본적인 동작과정과 배경, GoogleBot만의 특별한 동작과정, SEO 테스트 방법을 정리합니다.

----

## Table of contents

- [Background](#Background)  
  - [What is a Search Engine Bot](#What-is-a-Search-Engine-Bot)
  - [From SSR to CSR](#From-SSR-to-CSR)
- [Googlebot MythBusting](#Googlebot-MythBusting)  
  - [Googlebot doesn't understand modern Javascript](#Googlebot-doesn't-understand-modern-Javascript)
  - [It takes a long time to render pages](#It-takes-a-long-time-to-render-pages)
  - [JavaScript is bad for SEO](#JavaScript-is-bad-for-SEO)
- [Debug tools for Search](#Debug-tools-for-Search)
  - [Search Console](#Search-Console)
  - [Rich Results Test](#Rich-Results-Test)
  - [Mobile Friendly Test](#Mobile-Friendly-Test)
- [Debugging issues in the wild](#Debugging-issues-in-the-wild)
- [Reference](#Reference)  

<br />
<br />
<br />

## Background

----

<br />
<br />

### What is a Search Engine Bot

  <img width="800" alt="스크린샷 2020-09-01 오전 2 14 28" src="https://user-images.githubusercontent.com/56418546/91747353-fd299900-ebf8-11ea-85c4-bbec8ec27361.png">

- 검색엔진봇이란 일종의 웹 로봇입니다.
- 웹 로봇이란 것은 사람과의 상호작용이 없이 웹 트랜잭션(http get, etc.)을 자동으로 수행하는 봇을 의미합니다.
- 그래서 검색엔진봇이 어떤 웹페이지에 진입하면, 링크에 있는 다른 웹페이지들을 수집해 Crawling Queue에 담는 것을 재귀적으로 반복합니다.
- 일반적으로 검색엔진은 수집(크롤링) => 정제 => 인덱싱 => 랭크 => 질의의 프로세스를 가지고 있습니다.
- 검색결과를 향상시키려면 처음 검색엔진봇이 시작하는 지점인 '루트집합'의 선정이 중요합니다.
  - 관심있는 모든 웹페이지를 가져올 있도록 충분히 다른 장소에 있는 URL을 선택합니다.
  - 크고 인기 있는 사이트를 선정해야 합니다.
  - 새로 만들어지거나 잘 알려지지 않은 사이트 목록을 선정해야 합니다.
- 링크간 순환을 배제하여 중복된 크롤링을 방지합니다.
  - Binary Search Tree, HashTable 등을 통해 웹사이트 방문 여부를 체크한다.
  - BFS 알고리즘을 이용해 순환 가능성을 배제한다.
- 로봇이 잘 동작하기 위해서는 지침, 안내가 필요합니다.
  - robots.txt은 로봇이 부적절하게 동작하지 않기 위한 지침이자 에티켓입니다.
    - 로봇을 식별하고(user agent), 접근(allow), 차단(disallow)을 지시합니다.
    - <https://en.wikipedia.org/robots.txt>
    - <https://naver.com/robots.txt>
    - <https://www.kifrs.com/robots.txt>
  - sitemap.xml은 로봇에게 색인이 쉽도록 해당 웹사이트 내의 링크를 안내해줍니다.
    - <https://www.kifrs.com/sitemap.xml>
- 색인은 '풀 텍스트 색인'을 통해 이루어집니다. 문서별로 단어를 정리하는 것이 아닌, 단어를 기준으로 포함된 문서를 정리합니다.

<br />

<img width="600" alt="link" src="https://user-images.githubusercontent.com/56418546/91764669-ae88f880-ec12-11ea-8faa-582661af0fa9.png">

출처: <https://www.youtube.com/watch?time_continue=241&v=tagJ0lm6CK8&feature=emb_logo>

<br />

- 사용자가 질의를 하게되면 검색어와 관련이 높은 웹페이지부터 우선적으로 노출되게 됩니다.
  - 질의에 대응하기 위해서는 검색엔진이 동일한 주제에 대해서 순위를 매겨 검색 노출 순서를 정하는 '페이지 랭크'가 필요합니다.
    - 가정1) 다른 웹사이트로부터 많은 링크를 받았다면 더 좋은 컨텐츠를 가질 확률이 높다.
    - 가정2) 페이지 랭크가 높은 사이트가 링크해줬을 때가 페이지 랭크가 낮은 사이트의 링크해줬을 때 보다 더 좋은 컨텐츠를 가질 확률이 높다.
- 일반적으로 각 검색엔진별 페이지를 어떻게 정제하고, 인덱싱하고, 랭킹을 매기는지에 대해서는 기밀이라 잘 알려져 있지 않습니다.

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

<img width="1000" alt="evergreen googlebot" src="https://user-images.githubusercontent.com/56418546/91687396-8fe71b00-eb9a-11ea-8c1e-09657be82b99.png">

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

<br />

### Search Console

- 사용자가 Google 검색결과를 종합적으로 모니터링하고 관리하는 도구입니다.
- 이곳에 등록을 안해도 Google 검색결과가 가능하지만, 등록을 하게되면 사이트 인지도를 향상시킬 수 있는 다양한 지표를 보여줍니다.
- 그래서 본인의 도메인을 등록하고, Search Console로 사이트의 소유권을 확인받은 후 관리가 가능합니다.
- 색인 생성 범위와 URL 검사를 통해 색인이 제대로 되어있는지 여부를 판단할 수 있습니다.
- <https://chesssup.com>
- <https://search.google.com/search-console?resource_id=https%3A%2F%2Fchesssup.com%2F&hl=ko>
- <https://www.google.com/search?q=site%3Ahttps%3A%2F%2Fchesssup.com&oq=site%3Ahttps%3A%2F%2Fchesssup.com&aqs=chrome.0.69i59j69i58j69i61.1670j0j7&sourceid=chrome&ie=UTF-8>

<br />
<br />

### Rich Results Test

- Structured Data가 잘 들어가 있는지를 파악할 수 있는 도구입니다.
- Structured Data라는 것은 페이지에 관한 확실한 정보와 분류를 제공해주는 표준화된 양식입니다.
- Data의 type마다 권장하는 속성들이 있는데 이에 대해 잘 정의해놓을수록 Google 검색결과에 더 유리합니다.
- <https://www.google.com/search?q=pumpkin+pie+recipe&oq=pumpkin+pie+recipe&aqs=chrome..69i57j0l7.6469j0j7&sourceid=chrome&ie=UTF-8>
- <https://developers.google.com/search/docs/data-types/recipe?hl=ko>
- <https://search.google.com/test/rich-results?id=ULc6vfdNrC2oRglzRU8S7g>

```js
  <html>
    <head>
      <title>Party Coffee Cake</title>
      <script type="application/ld+json">
      {
        "@context": "https://schema.org/",
        "@type": "Recipe",
        "name": "Party Coffee Cake",
        "author": {
          "@type": "Person",
          "name": "Mary Stone"
        },
        "datePublished": "2018-03-10",
        "description": "This coffee cake is awesome and perfect for parties.",
        "prepTime": "PT20M"
      }
      </script>
    </head>
    <body>
    <h2>Party coffee cake recipe</h2>
    <p>
      This coffee cake is awesome and perfect for parties.
    </p>
    </body>
  </html>
```

<br />
<br />

### Mobile Friendly Test

- Google 검색결과를 유리하게 하기 위한 조건
  - 타이틀 및 메타태그(meta description, title)
  - Semantic markup(Headings, Article, Section, Form elements, etc.)
  - 로딩이 빠른 사이트(ssr > csr)
  - 보안 프로토콜 사용여부(https)
  - 모바일 친화적인 사이트(2015년 구글의 모바일 퍼스트 전략)
- 그래서 해당 사이트가 모바일에서 제대로 디스플레이되는지 여부를 파악하고 개선해나가는 작업이 중요합니다.
- <https://search.google.com/test/mobile-friendly?url=https%3A%2F%2Fchesssup.com%2F>
- <https://search.google.com/test/mobile-friendly?url=https%3A%2F%2Fwww.wikipedia.org%2F>


<br />
<br />
<br />

## Debugging issues in the wild  

- GoogleBot이 컨텐츠를 가져가는 것을 방해하는 요소
  - robots.txt
  - lazy loading 시에 scroll event  
  - service worker (PWA)

<br />
<br />
<br />

## Reference

- Google web.dev live 2020 Debugging Javascript Seo Issues: <https://www.youtube.com/watch?v=himvKu12YCY&list=PLNYkxOF6rcIDC0-BiwSL52yQ0n9rNozaF&index=9&t=0s>
- Google Web Master Series 1~8: <https://www.youtube.com/watch?v=LXF8bM4g-J4&list=PLKoqnv2vTMUPOalM1zuWDP9OQl851WMM9&index=1>
- HTTP the definitive guide - Web Robot
- 검색엔진 최적화(페이지랭크): <https://opentutorials.org/course/2039/10995>
- Dynamic Rendering for Javascipt web apps: <https://www.youtube.com/watch?time_continue=1&v=CrzUP6MmBW4&feature=emb_logo>
- How Google Search indexes Javascript sites: <https://www.youtube.com/watch?v=LXF8bM4g-J4&list=PLKoqnv2vTMUPOalM1zuWDP9OQl851WMM9>
- Google Search and JavaScript Sites (Google I/O'19): <https://www.youtube.com/watch?v=Ey0N1Ry0BPM&feature=emb_logo>
- Everything you need to know about Google's Evergreen Googlebot: <https://www.youtube.com/watch?time_continue=89&v=Wkc67ETypNo&feature=emb_logo>