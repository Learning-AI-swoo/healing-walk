# Known Issues & Notes — Healing Walk

## 주의사항

### gpt-4.1 선택지 형식 불안정
대화가 길어지면(10턴+) AI가 시스템 프롬프트 형식 지시를 무시하기 시작함.
→ 파싱 regex를 유연하게 처리해둠 (`**A)**`과 `A)` 형식 모두 매칭).
→ 22턴 제한으로 대화 길이 자체를 제어.

### AMBIENCE_LABELS_I18N 정의 순서
`UI_TEXT`에서 참조하므로 반드시 `UI_TEXT`보다 먼저 정의해야 함.
순서 바꾸면 ReferenceError 발생.

### 디버그 로그
`parseResponse`에 `console.log`가 남아있음.
프로덕션 배포 시 제거 권장 (필수는 아님).

### 브라우저 autoplay 정책
사운드 재생은 사용자 클릭 시점에 `Howler.ctx.resume()` 호출로 AudioContext를 unlock함.
`startWalk`, `handleChoice`, `handleWander` 세 곳에 적용돼 있음.

### AI 앰비언스 전환 의존성
AI가 같은 앰비언스에서 계속 머무는 경향 있음.
→ "다른 길로" 버튼으로 사용자가 직접 전환 가능.

---

## Vercel 배포 관련 주의

### CORS
로컬 테스트 시 `/api/chat`에 CORS 헤더 필요.
Vercel에 실제 배포하면 같은 도메인이라 불필요하지만, 로컬 `vercel dev` 테스트 대비로 넣어두면 좋음.

### 환경변수
Vercel 대시보드 → Settings → Environment Variables에서 `OPENAI_API_KEY` 직접 입력.
절대 코드에 하드코딩하거나 `.env`를 git에 커밋하지 말 것.

### 정적 파일 (images/, sounds/)
Vercel은 정적 파일 자동 서빙. `vercel.json`에 별도 설정 없어도 됨.
단, `sounds/*.mp3` 파일이 크면(17MB 등) 로딩이 느릴 수 있음 — 현재 구조 유지.

---

## 해결된 이슈 히스토리 (참고용)

1. 이미지 영역 너무 작음 → 70%로 확대
2. 밝은 이미지에서 텍스트 안 보임 → 글래스모피즘 카드
3. gpt-4o-mini 지시 불이행 → gpt-4.1 + 프롬프트 강화
4. 갑자기 종료 → 7턴 전 종료 금지
5. 이전 텍스트가 이미지 덮음 → 플로팅 패널 + 히스토리 접기
6. 텍스트가 선택지 영역 침범 → 나레이션 max-height + 카드 내 스크롤
7. 내러티브 단조로움 → 시스템 프롬프트 v2 (문체 가이드, few-shot)
8. 선택지 단순 이동 나열 → 3유형 프레임워크
9. 앰비언스 전환 안 됨 → 전환 경로 테이블 추가
10. gpt-5.1 테스트 → 느리고 장황, gpt-4.1이 최적
11. 사운드 없음 → Howler.js + 5개 앰비언스 사운드
12. 눈/해변 경험 불가 → "다른 길로" 버튼
13. 대화 무한 연장 시 AI 형식 붕괴 → 22턴 제한
14. D) 선택지 나레이션에 섞임 → 파싱 regex 확장 + 프롬프트에 "3개만" 명시
15. 선택지 볼드 없이 올 때 파싱 실패 → regex 유연화
16. 영어 지원 없음 → 언어 토글 + 영어 시스템 프롬프트 + i18n
17. 첫 턴 오디오 안 나옴 → Howler.ctx.resume() autoplay 대응
18. 숲 사운드 볼륨 낮음 → SOUND_VOLUMES 객체로 개별 조정 (forest: 0.6)
