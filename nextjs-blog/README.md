# NEXT JS 공식문서

## Pre-rendering의 두가지 형태
1.Static Generation
- build-time에 HTML 생성.(처리 속도 빠름)
- getStaticProps() 사용.
    - page가 렌더링 되기 전에 서버에서 사용.
    - 위 이유 때문에 pages 디렉토리 안에서만 export 가능.
```
export async function getStaticProps() {
    //Get external data from API, DB, etc...
    const data = ...

    return  {
        props: ...
    };
}
```



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

## SWR (Data를 fetching하기 위한 React Hook)
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