## Electron with React+Typescript+Tailwind from Figma MCP.
--------------------------------------------------------------------------------

당신은 제 Electron + React(Typescript) 프로젝트(루트: .)의 리팩토링 봇입니다.
Figma MCP가 활성화되어 있고, 아래 노드로부터 UI 코드를 생성해야 합니다.

[Figma 입력값]
- fileKey: lHANIyYZhOBMnbOJOBXArE
- nodeId : 1:204

[해야 할 일 — 순서대로 반드시 수행]
1) Figma MCP 툴 호출:
   - /Figma/get_code(nodeId="1:204", fileKey="lHANIyYZhOBMnbOJOBXArE")
   - /Figma/get_screenshot(nodeId="1:204", fileKey="lHANIyYZhOBMnbOJOBXArE")
   - 반환된 code와 asset URL 목록을 확보.

2) Electron + React(Typescript) + Tailwind ^4.1.13 맞춤 변환:
   - TSX 컴포넌트 파일 생성: ./src/renderer/components/Main.tsx
   - Tailwind v4 유틸 클래스 유지(또는 최소 보정)하되, 전역 CSS 한 곳에서만 Tailwind import.
   - 인라인 style/class는 유지 가능하나, 불필요한 style 속성은 유틸리티로 치환.

5) 엔트리 연결:
   - ./src/renderer/index.tsx 에서:
     import "./index.css";
     import Main from "./components/Main";
     createRoot(...).render(<Main />);
   - 위와 유사한 형태가 있으면 위 코드의 동작과 동일하도록 수정해주고.
   - 없으면 생성/수정.

6) 에셋(이미지/SVG) 로컬화(권장):
   - ./src/assets/ 폴더가 있으니 이곳에 저장.
   - 중복된 이름이 있으면 확장자가 아닌 이름에 "-dup"을 추가해서 저장.
   - /Figma/get_code 결과의 asset URL들을 모두 다운로드하여 저장(확장자는 .png 또는 .svg 추론).
   - Main.tsx에서 원격 URL 대신 로컬 import 경로로 치환:
     import icon0 from "../assets/<파일명>.png"; <img src={icon0} ... />

   - 필요 시 스크립트 생성: ./scripts/fetch-figma-assets.mjs
     (에셋 URL 목록을 하드코딩하는 간단 스크립트. 즉시 실행해 저장 후, TSX를 로컬 경로로 교체)

7) Electron 보안/동작 검수(참고):
   - ./main.js 에서 BrowserWindow webPreferences:
     { preload: 유효경로, contextIsolation: true, nodeIntegration: false, sandbox: true, zoomFactor: 1.0 }
   - preload.js 없으면 루트에 생성 및 build.files에 포함.

8) 산출물 출력:
   - 수정/생성된 파일 전체 경로 목록
   - 주요 TSX 코드(요약)와 최종 실행 방법
   - 실패 지점이 있으면 원인/해결책 설명

[제약 조건]
- 모든 리소스들은 런타임에 다운로드가 아닌 이미 다운로드 된 것을 사용하도록 한다.
- 인터넷이 안되는 환경에서 실행해야 하니 인터넷으로 다운 받는 코드가 있으면 미리 다운 받아서 다운받은 경로를 사용하게 해줘.

[검증 기준]
- npm run build:renderer 후 에러 0
- npm run start 로 Electron 실행 시, Figma 스크린샷과 시각적으로 동일하거나 근사
- Tailwind v4(@tailwindcss/postcss) 경고/에러 없음
- 컴포넌트 CSS에 Tailwind 재-import 금지(전역 index.css 1회만 import)
