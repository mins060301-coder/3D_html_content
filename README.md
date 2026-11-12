# Three.js 디지털 트윈 프로젝트 분석 보고서

본 문서는 제공된 두 개의 단일 HTML 파일(`smart_classroom_camera_stable_2.html`, `ThreeJS_korean_bus.html`)의 실행 방법과 전체적인 코드 구조를 분석한 내용입니다. 두 프로젝트 모두 외부 에셋(3D 모델 파일, 이미지 등) 없이 Three.js의 기본 도형(Primitives)과 코드를 활용해 정교한 3D 환경을 렌더링한다는 공통점이 있습니다.

---

## 1. 실행 방법 (How to Run)

두 파일은 모두 JavaScript의 ES6 모듈(`import`) 방식을 사용하고 있으며, Three.js 라이브러리를 외부 CDN(Content Delivery Network)에서 동적으로 불러옵니다. 따라서 보안 및 CORS(교차 출처 리소스 공유) 정책으로 인해 파일을 그냥 더블클릭해서 열면 정상적으로 작동하지 않을 수 있습니다.

### 권장 실행 절차
1. **인터넷 연결 확인**: CDN에서 Three.js 코드를 다운로드해야 하므로 인터넷 연결이 필수적입니다.
2. **로컬 웹 서버 사용**: 로컬 웹 서버를 통해 파일을 호스팅해야 합니다.
   * **VS Code 사용자**: `Live Server` 확장 프로그램을 설치한 뒤, HTML 파일을 우클릭하고 `Open with Live Server`를 선택합니다.
   * **Node.js 사용자**: 터미널에서 `npx http-server` 명령어를 입력하여 서버를 실행하고 `http://localhost:8080`에 접속합니다.
   * **Python 사용자**: 터미널에서 `python -m http.server`를 입력하고 `http://localhost:8000`에 접속합니다.
3. **브라우저 확인**: WebGL 및 하드웨어 가속을 지원하는 최신 브라우저(Chrome, Edge, Safari 등)에서 접속합니다.

---

## 2. 전체적인 파일 및 문법 구조 (File Structure)

두 파일 모두 HTML, CSS, JavaScript가 하나의 파일에 통합된 **싱글 파일 구조(Single-file architecture)**로 작성되었습니다.

### 2.1. HTML 구조 (`<head>` 및 `<body>`)
* **메타 태그 및 뷰포트**: 모바일 기기에서의 호환성과 다크 모드 테마 색상을 지정하는 `<meta>` 태그들이 포함되어 있습니다.
* **UI 레이어**: `<div id="app">` 또는 `<div id="stage">` 등의 컨테이너 안에 2D UI 요소(사이드바, 제어 패널, 진행률 바, 헤더 등)가 배치되어 있습니다. 이 UI 레이어는 3D 캔버스 위에 오버레이(`z-index` 사용)됩니다.
* **데이터 분리 (스마트 교실 한정)**: 교실 프로젝트의 경우 `<script type="application/json" id="sceneSpec">` 형태의 블록을 사용해 3D 공간의 크기, 위치 데이터 등을 JSON 형태로 분리하여 관리하는 패턴을 보여줍니다.

### 2.2. CSS 스타일링 (`<style>`)
* **CSS 변수 (Custom Properties)**: `:root` 선택자를 사용하여 메인 테마 색상(배경, 패널, 텍스트 등)을 변수로 지정하여 일관성을 유지합니다.
* **Flexbox & CSS Grid**: 복잡한 대시보드 레이아웃을 구성하기 위해 최신 레이아웃 기법을 적극 활용했습니다.
* **반응형 디자인 (Media Queries)**: `@media` 쿼리를 사용해 화면 크기가 작아질 때(모바일 환경 등) UI 패널이 숨겨지거나 하단으로 이동하도록 처리했습니다.

### 2.3. JavaScript 및 Three.js 모듈 (`<script type="importmap">` & `<script type="module">`)
전체 로직의 핵심으로, 크게 4가지 파트로 나뉩니다.

#### A. 모듈 임포트 (Import Map)
```json
<script type="importmap">
  {"imports": {"three": "https://cdn.jsdelivr.net/npm/three@.../build/three.module.js", ...}}
</script>
```
NPM이나 웹팩(Webpack) 같은 빌드 도구 없이도 브라우저에서 바로 `import * as THREE from 'three';` 문법을 사용할 수 있게 해주는 최신 웹 표준 기술입니다.

#### B. 3D 씬 초기화 (Scene Setup)
* **Renderer**: `WebGLRenderer`를 생성하고 `PCFSoftShadowMap` 등 그림자와 톤 매핑(Tone Mapping) 설정을 추가해 실사풍 그래픽을 구현합니다.
* **Camera & Controls**: `PerspectiveCamera`를 사용하며, 사용자가 마우스로 화면을 돌려볼 수 있도록 `OrbitControls`를 연결합니다.
* **Lighting**: `HemisphereLight`(하늘/바닥 반사광)와 `DirectionalLight`(태양광/그림자)를 조합하여 사실적인 조명 환경을 구성합니다.

#### C. 절차적 3D 모델링 (Procedural Generation)
두 코드의 가장 큰 특징은 **GLTF 등 외부 3D 파일을 전혀 쓰지 않는다는 점**입니다.
* `BoxGeometry`, `CylinderGeometry`, `SphereGeometry` 등 Three.js 기본 기하학 도형을 사용합니다.
* `MeshStandardMaterial`, `MeshPhysicalMaterial`을 활용해 유리, 플라스틱, 메탈 등의 재질감을 코드로 부여합니다.
* 자주 사용하는 도형 생성 로직을 헬퍼 함수(`bx`, `cyl`, `sphere` 등)나 `GeometryFactory` 패턴으로 묶어 코드 중복을 줄였습니다.

#### D. 상태 관리 및 애니메이션 루프 (Animation Loop & UI Binding)
* **상태 객체**: `state` 또는 `sim` 이라는 JavaScript 객체를 만들어 현재 조립 단계, 온도, 자동 실행 여부 등의 상태 데이터를 관리합니다.
* **이벤트 리스너 (DOM Events)**: HTML 버튼이나 슬라이더(`input[type="range"]`)를 클릭/조작할 때 이 상태 객체를 업데이트하고 3D 모델의 속성을 변경합니다. (Raycaster를 이용한 3D 객체 클릭 이벤트도 포함)
* **루프 (Loop)**: `renderer.setAnimationLoop()` 또는 `requestAnimationFrame()` 안에서 매 프레임마다 카메라의 부드러운 이동(Lerp 보간 연산), 로봇 팔의 회전(`Math.sin`, `Math.cos` 사용), 부품의 이동 애니메이션을 처리하고 최종적으로 화면을 렌더링(`renderer.render()`) 합니다.

### 2.4. 폴백 스크립트 (Fallback Script)
파일 하단에는 일반 `<script>` 태그가 배치되어 있습니다. Three.js 로딩에 실패하거나 사용자의 그래픽 카드(WebGL)에 문제가 생겨 튕겼을 때, 무한 로딩에 빠지지 않도록 에러 메시지와 재시도 버튼을 띄워주는 안전장치 역할을 합니다.