# NEXT JS 공식문서

## Pre-rendering의 두가지 형태
1.Static Generation
- build-time에 HTML 생성.(처리 속도 빠름)
- getStaticProps() 사용.
    - page가 렌더링 되기 전에 서버에서 사용.
    - 위 이유 때문에 pages 디렉토리 안에서만 export 가능.

2. Server-side Rendering