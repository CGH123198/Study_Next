## Pre-rendering의 두가지 형태
1.Static Generation
- build-time에 HTML 생성.(처리 속도 빠름)
- getStaticProps() 사용. 위부 데이터를 fetch할 때 사용.
    - page가 렌더링 되기 전에 서버에서 사용.
    - 위 이유 때문에 pages 디렉토리 안에서만 export 가능.
    - Next.js코드에 getStaticProps가 있다면 Next.js가 빌드시 함수를 실행하고, 그 props를 컴포넌트에 전달.
    - Next.js에게 "이 페이지에 data dependencies가 있으니 먼저 실행하라"고 알림.
```
export async function getStaticProps(context) {
    //Get external data from API, DB, etc...
    const data = ...

    return  {
        props: ...
    };
}
```
    - context Parameter
        - `params`: dynamic routes를 사용하는 pages에 대한 route parameters. e.g) page이름이 `[id].js`일 때, `params`는  { id: ... }이다.
        - `preview`: Preview Mode에 있는 page가 `undefined`가 아니라면 `true`.
        - `previewData`: `setPreviewData`에 의해 세팅되는 previewData
        - `locale`, `locales`, `defaultLocale`


2. Server-side Rendering
- 데이터가 빈번히 업데이트 되는 페이지에 적합.
- request가 올 때마다 rendering.
- SEO에 적합. Static Generation에 비해 처리 속도 느림.
- getServerSideProps() 사용.
```
export async function getServerSideProps(context) {
    return {
        props: {
            //props for your component            
        },
    };
}
```

- SWR (Data를 fetching하기 위한 React Hook)
    - client side에서 data를 fetching할 때 추천함.
    - caching, revalidation, focus tracking, refetching 등등 처리.
    ```
    import userSWR from 'swr';

    function Profile() {
        const { data, error } = useSWR('/api/user', fetch);
        if (error) return <div>failed to load</div>;
        if (!data) return <div> loading...</div>;
        return <div>hello {data.name}!</div>;
    }
    ```
    - [SWR 자세한 내용](https://swr.vercel.app/ko)


## Dynamic Routes
1. external data에 따라  동적으로 page path가 생성된다.
Dynamic Routes pages 생성을 위해서는 다음과 같은 내용이 page file에 포함되어야 한다.(/pages/posts/[id].js)
    - page에 렌더할 React Component
    - id값으로 된 배열을 반환하는 getStaticPaths
    - id값을 사용해 post에 대한 data를 가져오는 getStaticProps

2. getStaticPaths
- dynamic routes를 사용하는 page에서 호출되면, next.js는 getStaticPaths에 의해 특정된 모든 경로들을 정적으로 pre-render한다.
- getStaticPaths는 getServerSideProps가 아닌 getStaticProps와 함께 써야만 한다.
```
export async function getStaticPaths() {
  return {
    paths: [
      { params: { ... } }
    ],
    fallback: true // false or 'blocking'
  };
}
```
- `getStaticPaths`는 언제 사용되는가?
    - dynamic routes를 사용하는 page들을 pre-render될 떄
    - headless CMS, DB, FileSystem에서 data가 호출 될 때
    - data를 공개적으로 캐싱할 수 있을 때
    - page가 (SEO전용)매우 빨라야 하고 pre-rendering되어야 할 때.
    (`getStaticProps`는  성능을 위해 CDN에 cache될 수 있는 `HTML`과 `JSON`파일을 생성한다.)
- getStaticPaths는 언제 실행되는가?
    - In production: build Time, In development: every request
- getStaticProps는 getStaticPaths와 관련하여 어떻게 실행되는가?
    - `getStaticProps`는 build하는 동안 반환된 path들에 대해 'next build'하는 동안 실행된다.
    - `fallback: true`일 때 background에서 실행된다.
    - `fallback: blocking`일 때 초기 렌더링 이전에 호출된다.
- getStaticPaths는 어디서 사용할 수 있는가?(위치)
    - `dynamic routes`를 사용하는 곳에서 export
    - non-page files에서 export 불가
    - 누군가의 props로 사용하지 못한다.(standalone function)
- Fallback
    - `fallback: false`: `getStaticPaths에 의해 반환되지 않은 경로는 404 반환.
    - `fallback: true`: `getStaticProps`의 행동이 변화한다.
        - `getStaticPaths`에서 반환된 path들이 buildTime에 HTML에서 렌더된다.
        - buildTime에 생성되지 않은 경로들은 404. Next.js는 그런 경로들에 대한 첫 요청에는 page의 `fallback`version을 제공한다.(`fallback` version = 대체 페이지)
        - background에서 Next.js는 요청된 경로를 정적으로 생성할 것이다. 동일 경로에 대한 재요청은 buildTime에 pre-render된 다른 페이지처럼 처리한다.
    - `fallback: blocking`: 새로운 경로는 `getStaticProps`와 함께 server-side에서 렌더링된다. 미래의 요청에 대비해 cach되므로 경로 당 한 번만 수행된다.

## API Routes : Next.js app에 API endpoint를 만들어준다.(`pages/api`안에 함수 정의)
    - `getStaticProps`와 `getStaticPaths`에서 API Route를 Fetch 하지마라. `getStaticProps`와 `getStaticPaths`에는  server-side code를 직접 써라. 이 두 함수는 client-side 코드는 절대 돌아가지 않는다. 그리고 이 두 함수는 브라우저를 위한 JS bundle에 포함되지 않는다. 이것은 우리가 db query문 같은 코드들을 브라우저에 보내지 않고 직접 작성할 수 있음을 의미한다.(~~HTML~~)
    ```
    function handler(req, res) {
        res.status(200).join({ message: 'Success'})
    }
    ```
    [Writing Server-side code](https://nextjs.org/docs/basic-features/data-fetching/get-static-props#write-server-side-code-directly)

    - good usecase: Handling Form Input, Preview Mode, Dynamic API Routes
