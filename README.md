# Unity

1. 프론트 public 폴더에 Unity 빌드 넣기

프론트가 React/Vite든 Vue든 보통 public 폴더가 있어.
거기에 unity 폴더를 만들고, Unity 빌드 결과물을 통째로 넣어.

예시:

frontend/
 ┣ public/
 ┃ ┗ unity/
 ┃    ┣ index.html
 ┃    ┣ Build/
 ┃    ┗ TemplateData/
 ┣ src/
 ┣ package.json
 ┗ ...

중요한 건 index.html, Build, TemplateData를 따로따로 고르지 말고 빌드 결과 폴더 내용을 그대로 복사하는 거야.

2. 프론트 실행하기

프론트 폴더에서 실행해.

npm install
npm run dev

프론트가 http://localhost:3000에서 실행된다면 Unity는 이렇게 접속하면 돼.

http://localhost:3000/unity/index.html

프론트가 http://localhost:5173이면:

http://localhost:5173/unity/index.html
3. 로그인 토큰 연동 때문에 꼭 같은 포트에서 열기

너희는 Unity WebGL이 프론트의 localStorage.accessToken을 읽는 구조였잖아.
그래서 프론트 로그인 페이지와 Unity 페이지가 같은 출처여야 해.

가능한 구조:

프론트 로그인: http://localhost:3000
Unity WebGL:  http://localhost:3000/unity/index.html

이건 가능.

안 좋은 구조:

프론트 로그인: http://localhost:3000
Unity WebGL:  http://localhost:5173/unity/index.html

이러면 포트가 달라서 localStorage가 공유되지 않아.

4. 프론트에 버튼으로 연결하고 싶으면

프론트 페이지에서 게임 시작 버튼을 만들고 아래 주소로 이동시키면 돼.

<button onClick={() => window.location.href = "/unity/index.html"}>
  게임 시작
</button>

또는 새 탭으로 열고 싶으면:

<button onClick={() => window.open("/unity/index.html", "_blank")}>
  게임 시작
</button>
5. 프론트 안에 Unity를 끼워 넣고 싶으면 iframe 사용

게임 페이지 안에 Unity를 넣고 싶으면 이렇게 할 수 있어.

export default function GamePage() {
  return (
    <div style={{ width: "100%", height: "100vh" }}>
      <iframe
        src="/unity/index.html"
        title="Unity WebGL Game"
        style={{
          width: "100%",
          height: "100%",
          border: "none"
        }}
      />
    </div>
  );
}

이 경우에도 Unity 주소는 같은 도메인 안의 /unity/index.html이라서 토큰 공유가 가능해.

6. 백엔드 주소 확인

Unity의 EduQuestApiManager에서 Base Url이 예를 들어:

http://localhost:8080

이면, Unity가 백엔드로 요청할 때 백엔드에서는 프론트 출처를 CORS 허용해야 해.

프론트가 http://localhost:3000이면 백엔드 CORS에 이것이 들어가야 해.

http://localhost:3000
