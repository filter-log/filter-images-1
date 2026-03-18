# filter-images-1

`filter-images-1`는 GitHub Pages로 공개 갤러리를 배포하고, Cloudflare Worker로 원본 이미지를 업로드하며, GitHub Actions로 이미지를 후처리하는 실제 운영용 이미지 저장소다.

이 레포는 `filter-images-template` 구조를 기반으로 하지만, `repoName`, `galleryBaseUrl`, `workerApiUrl`이 이미 `filter-images-1` 기준으로 연결되어 있다.

## 이 레포가 제공하는 것

- `/` 정적 공개 갤러리
- `/upload/` Worker 연동 업로드 페이지
- `YYYY-MM-DD` 단일 날짜 폴더 구조
- `incoming/2026-03-18/file.jpg` 같은 유입 경로
- `images/2026-03-18/file.webp` 공개 이미지 경로
- `thumbs/2026-03-18/file.webp` 썸네일 경로
- `data/images.json` 기반 날짜별 탐색 UI
- `incoming/`을 처리하는 GitHub Actions 워크플로

## 날짜 구조

이 템플릿은 날짜를 반드시 하나의 폴더명으로 사용한다.

- 유입 경로: `incoming/2026-03-18/filename.ext`
- 공개 경로: `images/2026-03-18/filename.webp`
- 썸네일 경로: `thumbs/2026-03-18/filename.webp`

중요:

- `YYYY/MM/DD` 분리 구조를 쓰지 않는다.
- 업로드 페이지의 날짜 입력값도 `YYYY-MM-DD`다.
- `images.json`도 `date` 문자열 하나로 그룹핑한다.

## 저장소 구조

```text
.
├── .github/workflows/process-images.yml
├── assets/
│   ├── css/style.css
│   └── js/
│       ├── config.js
│       ├── gallery.js
│       └── upload.js
├── data/images.json
├── docs/
│   ├── ARCHITECTURE.md
│   └── OPERATIONS.md
├── images/.gitkeep
├── incoming/.gitkeep
├── scripts/
│   ├── generate-images-json.mjs
│   └── process-images.mjs
├── thumbs/.gitkeep
├── upload/index.html
└── index.html
```

## 현재 설정

현재 업로드 설정은 [assets/js/config.js](assets/js/config.js)에 들어 있다.

예:

```js
window.UPLOAD_CONFIG = {
  repoName: "filter-images-1",
  galleryBaseUrl: "https://filter-log.github.io/filter-images-1",
  workerApiUrl: "https://filter-image-upload-worker.filter-log.workers.dev",
  maxFiles: 100,
};
```

중요:

- `workerApiUrl`에는 Worker base URL만 넣는다.
- 프론트엔드는 `${workerApiUrl}/auth`, `${workerApiUrl}/upload`를 조합해서 사용한다.
- `repoName`은 반드시 `filter-images-1`이어야 한다.

## 비전공자 사용 흐름

1. 업로드 페이지 `/upload/`를 연다.
2. 날짜를 선택한다. 기본값은 오늘 날짜다.
3. 이미지를 드래그앤드롭하거나 여러 장 선택한다.
4. 암호를 입력한다.
5. 화면에서 `incoming/YYYY-MM-DD/...` 경로 미리보기를 확인한다.
6. 업로드 버튼을 누른다.
7. 업로드 페이지가 Worker의 `/auth`로 암호를 검증해 짧은 Bearer 토큰을 발급받는다.
8. 같은 화면에서 Worker의 `/upload`로 파일을 전송한다.
9. 업로드된 파일은 `incoming/YYYY-MM-DD/...` 경로에 저장된다.
10. 해당 레포의 GitHub Actions가 `images/`, `thumbs/`, `data/images.json`을 갱신한다.

## 공개 갤러리 동작

갤러리는 [data/images.json](data/images.json)을 읽어 날짜별 필터와 카드 UI를 렌더링한다.

각 카드에서 제공하는 기능:

- 썸네일 보기
- 원본 보기
- URL 복사
- Markdown 복사

Markdown 예시:

```md
![a](https://filter-log.github.io/filter-images-1/images/2026-03-18/a.webp)
```

## GitHub Actions 후처리 구조

[.github/workflows/process-images.yml](.github/workflows/process-images.yml)은 `incoming/**` 변경을 감지한다.

워크플로가 실행되면:

1. `incoming/2026-03-18/*.jpg` 같은 원본을 읽는다.
2. 긴 변 1600px 이하로 리사이즈한다.
3. WebP로 변환한다.
4. 400px 수준의 썸네일을 만든다.
5. `images/2026-03-18/`와 `thumbs/2026-03-18/`에 결과를 쓴다.
6. `data/images.json`을 다시 생성한다.

## Worker 연결 규칙

- `workerApiUrl`에는 Worker의 base URL만 넣는다.
- 프론트엔드는 `${workerApiUrl}/auth`와 `${workerApiUrl}/upload`를 사용한다.
- `repoName`은 `filter-images-1`과 정확히 같아야 한다.
- Worker의 `ALLOWED_REPOS`에 `filter-images-1`이 포함돼 있어야 한다.

## 운영자 메모

- 업로드 경로 규칙은 코드 전체에서 `YYYY-MM-DD` 한 단계 폴더 구조로 통일돼 있다.
- 장소 정보나 별도 폴더 정보는 넣지 않는다.
- 업로드 후에는 GitHub Actions가 `incoming/` 원본을 `images/`, `thumbs/`, `data/images.json`으로 정리한다.

## 참고 문서

- 아키텍처: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- 운영 가이드: [docs/OPERATIONS.md](docs/OPERATIONS.md)
