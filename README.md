# Unity

1) Unity WebGL 빌드 결과물을 프론트의 정적 폴더로 넣기

Unity WebGL 빌드 결과가 보통 이렇게 나오잖아.

index.html
Build/
TemplateData/

이걸 프론트 프로젝트의 public/unity/ 아래에 그대로 넣어.

예시:

Frontend-main/
  public/
    unity/
      index.html
      Build/
      TemplateData/

그러면 프론트 개발 서버에서 이 주소로 열 수 있어.

http://localhost:5173/unity/index.html

중요한 건 index.html, Build, TemplateData를 같은 폴더 구조로 유지하는 거야.

2) 같은 도메인에서 열어야 함

이게 제일 중요해.

프론트 로그인 페이지가 예를 들어

http://localhost:5173

에서 열리고,
Unity도

http://localhost:5173/unity/index.html

에서 열려야 localStorage.accessToken을 공유해서 읽을 수 있어.

반대로 Unity를

http://127.0.0.1:5500

같은 다른 출처에서 열면 토큰 공유가 안 돼.
