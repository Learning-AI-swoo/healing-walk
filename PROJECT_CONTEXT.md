# Project Context — Healing Walk

## 앱 개요

AI가 나레이션하는 자연 속 산책 텍스트 어드벤처.
사용자가 선택지를 고르며 숲/비/밤/눈/해변 중 하나를 걷는 경험을 제공한다.
한국어/영어 이중언어 지원.

## 기술 스택

- **프론트엔드**: 순수 HTML + CSS + JavaScript (프레임워크 없음)
- **AI**: OpenAI `gpt-4.1` (chat completions)
- **사운드**: Howler.js 2.2.4 (CDN)
- **폰트**: Noto Sans KR (한국어) + DM Sans (영어) — Google Fonts CDN
- **파티클**: HTML5 Canvas
- **UI**: backdrop-filter 글래스모피즘
- **배포**: Vercel (서버리스 함수 + 정적 파일)

## 앰비언스 (5종)

| ID | 이름(ko) | 이름(en) | 파티클 | 사운드 |
|---|---|---|---|---|
| `forest` | 숲길 🌿 | Forest 🌿 | 낙엽 | forest.mp3 |
| `rain` | 빗속 🌧️ | Rain 🌧️ | 빗줄기 | rain.mp3 |
| `night` | 밤길 ✨ | Night ✨ | 반딧불 | night.mp3 |
| `snow` | 설경 ❄️ | Snow ❄️ | 눈 | snow.mp3 |
| `beach` | 해변 🌅 | Beach 🌅 | 없음 | beach.mp3 |

## 씬 이미지 (21개)

- forest: forest_path, forest_stream, forest_bench, forest_cabin, forest_bridge
- rain: rain_path, rain_shelter, rain_puddle, rain_umbrella
- night: night_moon, night_fireflies, night_stars, night_campfire
- snow: snow_path, snow_lake, snow_village, snow_field
- beach: beach_sunset, beach_lighthouse, beach_tide, beach_dock

모두 `./images/[id].jpeg` 경로에 위치.

## 화면 구성

1. **타이틀 화면** — 언어 토글(🇰🇷/🇺🇸) + 시작 버튼  
   *(Vercel 버전에서는 API 키 입력 필드 없음)*
2. **산책 화면** — 이미지 배경(70%) + 하단 플로팅 패널(나레이션 + 선택지) + 🔊 토글 + 🧭 다른 길로 버튼
3. **앰비언스 선택 오버레이** — "다른 길로" 클릭 시
4. **일기 화면** — 3줄 산책 일기 + 다시 걷기 버튼

## API 응답 파싱 (프론트엔드, 변경 없음)

AI 응답에서 다음 태그를 파싱:

1. `[AMBIENCE:id]` → 배경 + 파티클 + 사운드 전환
2. `[SCENE:id]` → 이미지 전환
3. `[SURPRISE:emoji:text]` → 풀스크린 오버레이 3.5초
4. `---DIARY---...---END---` → 엔딩 일기 화면
5. 선택지: `**A)**` 또는 `A)` 형식 (A~C)
6. 나머지 → 나레이션 텍스트

## 사운드 시스템

- Howler.js 루프 재생
- 앰비언스 전환 시 2초 크로스페이드
- 각 앰비언스별 볼륨 설정 (SOUND_VOLUMES 객체)
- 🔊/🔇 토글, 타이틀/일기 화면에서 정지

## 턴 제한

- 7턴 미만: 귀가 불가
- 7턴 이상: 선택지 C)에 귀가 옵션
- 22턴 이상: 강제 귀가 (선택지 대신 귀가 버튼 하나만)
- "다른 길로" 버튼: 7~21턴 사이에만 표시

## callAPI() 함수 — 변경 전후

### 변경 전 (직접 호출)
```javascript
const res = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${state.apiKey}`,
  },
  body: JSON.stringify({
    model: 'gpt-4.1',
    messages: [...systemMsg, ...state.messages],
    max_tokens: 700,
  })
});
```

### 변경 후 (서버 프록시)
```javascript
const res = await fetch('/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    messages: state.messages,
    lang: state.lang,
  })
});
const data = await res.json();
// data.content가 AI 응답 텍스트
```

## state 객체 주요 필드

```javascript
const state = {
  lang: 'ko',           // 'ko' | 'en'
  ambience: null,       // 현재 앰비언스 ID
  turn: 0,              // 현재 턴 수
  messages: [],         // 전체 대화 히스토리 (OpenAI format)
  // apiKey, provider 필드는 Vercel 버전에서 제거
};
```
