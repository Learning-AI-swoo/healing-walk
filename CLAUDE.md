# Healing Walk — Vercel 배포 작업 (Claude Code 초기화 문서)

## 이 세션의 목표

Healing Walk 웹앱을 Vercel 서버리스 구조로 전환하는 작업이다.
**원본 GitHub Pages 레포는 건드리지 않는다.** 이 폴더는 새로 복사된 작업 폴더다.

---

## 작업 내용 요약

현재 `index.html`은 브라우저에서 OpenAI API를 직접 호출한다 (API 키가 사용자에게 노출됨).
이를 다음 구조로 바꾸는 게 목표:

```
사용자 브라우저 → /api/chat (Vercel 서버리스 함수) → OpenAI API
```

API 키는 Vercel 환경변수에만 존재하고, 프론트엔드 코드에서 완전히 제거된다.

---

## 해야 할 작업 목록

### 1. `index.html` 수정
- Provider 선택 UI (OpenAI/Anthropic 토글 버튼) 제거
- API 키 입력 필드 제거
- 타이틀 화면에 언어 선택 + 시작 버튼만 남기기
- `callAPI()` 함수: OpenAI 직접 호출 코드 제거, `/api/chat`으로 POST 요청으로 교체
- 요청 body: `{ messages: [...], lang: 'ko' | 'en' }`
- 응답: OpenAI가 반환하는 텍스트를 그대로 파싱 (기존 파싱 로직 유지)
- 시스템 프롬프트 (`SYSTEM_PROMPT`, `SYSTEM_PROMPT_EN`) 코드에서 제거 (서버로 이동)

### 2. `api/chat.js` 생성 (Vercel 서버리스 함수)
- `OPENAI_API_KEY`는 `process.env.OPENAI_API_KEY`에서 읽기
- 시스템 프롬프트 (`SYSTEM_PROMPT_KO`, `SYSTEM_PROMPT_EN`)를 이 파일 안에 포함
- 클라이언트로부터 `{ messages, lang }` 수신
- lang에 따라 시스템 프롬프트 선택
- OpenAI `gpt-4.1` 모델로 호출 (`max_tokens: 700`)
- 결과 텍스트만 클라이언트에 반환
- CORS 헤더 추가 (로컬 테스트 대비)

### 3. `vercel.json` 생성
- 라우팅 설정: `/api/*` → 서버리스 함수
- 정적 파일 서빙 설정

### 4. `.gitignore` 확인/생성
- `.env`, `.env.local` 포함 확인
- `node_modules` 포함 확인

### 5. `package.json` 생성 (필요 시)
- Vercel이 Node.js 환경 인식하도록

---

## 프로젝트 구조 (작업 후 목표)

```
healing-walk-vercel/
├── CLAUDE.md               ← 이 파일
├── index.html              ← 수정된 프론트엔드
├── vercel.json             ← Vercel 라우팅 설정
├── package.json            ← Node.js 환경 선언
├── .gitignore
├── api/
│   └── chat.js             ← 서버리스 함수 (API 키 여기)
├── images/                 ← 픽셀아트 이미지 21개 (건드리지 않음)
└── sounds/                 ← 앰비언스 사운드 5개 (건드리지 않음)
```

---

## 참고 문서

- `docs/PROJECT_CONTEXT.md` — 프로젝트 전체 맥락 및 기술 스택
- `docs/SYSTEM_PROMPTS.md` — 한국어/영어 시스템 프롬프트 전문
- `docs/KNOWN_ISSUES.md` — 알려진 이슈 및 주의사항

---

## 배포 방법 (작업 완료 후)

1. 이 폴더를 새 GitHub 레포로 push
2. vercel.com → New Project → 해당 레포 연결
3. Environment Variables에 `OPENAI_API_KEY` 추가
4. Deploy

또는 `vercel` CLI 사용:
```bash
vercel --prod
```

---

## 중요 제약

- 원본 레포(`learning-ai-swoo/healing-walk`)는 건드리지 않는다
- OpenAI만 지원 (Anthropic 코드 제거)
- `gpt-4.1` 모델 고정
- 이미지/사운드 파일은 수정 없음
