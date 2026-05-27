# 🔍 모바일 최적화 - 남은 개선사항 분석

**Date**: 2026-05-22
**Status**: 📋 **다음 단계 개선 대상 식별**

---

## 📊 분석 결과: 5가지 개선 카테고리

지금까지 12개 팝업을 최적화했습니다. 다음은 아직 개선할 수 있는 5가지 주요 영역입니다.

---

## 🎯 카테고리 1: **Hover 텍스트 최적화** (높은 우선순위)

### 현재 문제
모바일에서 `title` attribute가 작동하지 않음:
```html
<button title="일시정지 (P / ESC)">⏸</button>
<div title="카드명">...</div>
<span title="렐릭명: 설명">...</span>
```

### 영향받는 요소
- ❌ 일시정지 버튼 설명
- ❌ 일시정지 메뉴의 룬 카드 (hover시 카드명 표시)
- ❌ 일시정지 메뉴의 렐릭 (hover시 렐릭명 표시)
- ❌ 장비 카드 설명
- ❌ 업그레이드 힌트

### 해결 방안
```javascript
// 모바일에서는 long-press로 tooltip 표시
el.addEventListener('longpress', () => {
  showMobileTooltip(el.title);
});

// 또는 터치 시 별도 팝업 표시
el.addEventListener('touchstart', () => {
  showToast(el.title, 1500);
});
```

### 개선 효과
- ✅ 모바일에서도 **카드/렐릭 정보 확인 가능**
- ✅ 게임플레이 **더욱 명확**

### 작업량
- 약 10-15개 요소 수정
- 함수 1-2개 추가

---

## 🎯 카테고리 2: **일시정지 메뉴 - 룬/렐릭 표시 개선** (높은 우선순위)

### 현재 상태
```javascript
// 일시정지 메뉴의 룬 목록
gs.deck.map(c=>`<span style="...font-size:11px...">${c.emoji}`)
  
// 일시정지 메뉴의 렐릭 목록  
gs.relics.map(r=>`<span style="...font-size:20px...">${r.emoji}`)
```

### 문제점
- 룬 카드가 너무 작게 표시 (font-size: 11px)
- 렐릭이 너무 크게 표시 (font-size: 20px)
- 모바일에서 스크롤 불가능한 리스트

### 해결 방안
```javascript
// 모바일에서는 더 큰 카드로 표시
const isMobile = window.innerWidth < 520;

// 룬 목록: flex-wrap + responsive 크기
style="font-size:${isMobile ? '14px' : '11px'}"

// 렐릭 목록: 2-3열 그리드
grid-template-columns:${isMobile ? 'repeat(3, 1fr)' : 'repeat(5, 1fr)'}
```

### 개선 효과
- ✅ 모바일에서 **룬/렐릭이 더 크게 표시**
- ✅ 긴 리스트도 **쉽게 스크롤**

### 작업량
- 약 30-50줄 수정
- 새로운 CSS 규칙 추가

---

## 🎯 카테고리 3: **색상 대비 & 접근성** (중간 우선순위)

### 분석 항목

#### 3-1. 색상 대비 (WCAG AA 기준)
```
WCAG AA: 4.5:1 이상 (텍스트)
WCAG AA: 3:1 이상 (UI 요소)
```

### 확인 필요 요소
- ❓ 설명 텍스트 (var(--text2): #8aa0b8) vs 배경
- ❓ 버튼 텍스트 vs 배경
- ❓ 진행 표시기 (dots)
- ❓ 비활성화 상태 텍스트

### 추천 개선
```javascript
// 대비가 낮은 텍스트 확인 및 수정
var(--text2) #8aa0b8 // 매우 밝음
→ #6a8aaa 또는 #5a7a9a 로 어둡게 (더 나은 대비)

// 비활성 요소
rgba(255,255,255,0.4) // 너무 옅음
→ rgba(255,255,255,0.6) // 더 명확
```

### 개선 효과
- ✅ **색맹/저시력 사용자** 포함
- ✅ **접근성 기준 충족**
- ✅ **흐린 조명에서도 읽기 좋음**

### 작업량
- 약 20-30개 색상 값 검토/수정
- CSS 변수 업데이트

---

## 🎯 카테고리 4: **Landscape 모드 게임플레이 개선** (중간 우선순위)

### 현재 상태
```css
@media (max-height:500px) and (orientation:landscape) {
  /* 기본 최적화만 있음 */
  .game-title { display:none; }
  .panel { padding:8px 12px; }
}
```

### 개선 기회
- ❓ 게임 캔버스 크기 최적화
- ❓ 모바일 조이스틱 위치 조정
- ❓ HUD 배치 최적화
- ❓ 버튼 위치 재배치

### 해결 방안
```javascript
// Landscape 감지
const isLandscape = window.innerHeight < window.innerWidth;

// 게임플레이 최적화
if (isLandscape) {
  canvas.width = window.innerWidth * 0.95;
  joystick.style.left = '10px';
  joystick.style.bottom = '20px';
}
```

### 개선 효과
- ✅ Landscape에서 **게임 플레이 가능**
- ✅ **모든 요소가 화면에 맞음**

### 작업량
- 약 50-100줄 추가
- 새로운 CSS 미디어 쿼리

---

## 🎯 카테고리 5: **폰트 크기 일관성 & 가독성 개선** (중간-낮은 우선순위)

### 현재 분석
```
폰트 크기 정의: 555개
폰트 가족: 3가지 ('Cinzel', 'Jua', 'Noto Sans KR')
line-height 정의: 불일치
```

### 문제점
- ❓ 같은 목적의 텍스트인데 폰트 크기가 다름
- ❓ line-height가 일관성 없음
- ❓ 모바일/데스크톱 일부 요소는 아직 미최적화

### 예시
```javascript
// 같은 "설명" 텍스트인데:
설명 1: font-size:11px, line-height:1.4
설명 2: font-size:12px, line-height:1.6
설명 3: font-size:10px, line-height:1.5 ← 일관성 없음!
```

### 해결 방안
```javascript
// 폰트 크기 표준화
const FONT_SIZES = {
  title: { mobile: 18, desktop: 22 },      // 제목
  subtitle: { mobile: 14, desktop: 16 },   // 부제목
  body: { mobile: 12, desktop: 14 },       // 본문
  caption: { mobile: 10, desktop: 11 },    // 캡션
  small: { mobile: 9, desktop: 10 }        // 작은 텍스트
};

// 표준 line-height
const STANDARD_LINE_HEIGHT = {
  title: 1.2,    // 제목
  body: 1.5,     // 본문
  dense: 1.3     // 컴팩트
};
```

### 개선 효과
- ✅ **일관된 가독성**
- ✅ **유지보수 용이**
- ✅ **새 요소 추가 시 빠름**

### 작업량
- 약 100-150줄 수정
- 예상 시간: 2-3시간

---

## 🎯 카테고리 6: **추가 UI 요소 최적화** (낮은 우선순위)

### 아직 미최적화 요소들

#### 6-1. 스킬 업그레이드 팝업
```javascript
// 스킬을 레벨업할 때 표시되는 팝업
// 현재 상태: 일반 적 크기
```

#### 6-2. 보스 승리 화면
```html
<!-- 현재 HTML로 정의됨, 모바일용 CSS 추가 필요 -->
<div id="bossVictoryScreen" class="screen hidden">
  <!-- 텍스트 크기 확인 필요 -->
</div>
```

#### 6-3. 선택지 버튼 hover 상태
```javascript
// 데스크톱에서는 hover로 표시되지만
// 모바일에서는 visual feedback이 약함
```

### 개선 방법
- 각 요소에 모바일용 미디어 쿼리 추가
- 터치 피드백 강화 (:active 상태)

### 작업량
- 각 5-10줄씩 추가
- 총 30-50줄

---

## 🎯 카테고리 7: **성능 최적화** (선택사항)

### 분석 항목

#### 7-1. 렐릭/카드 풀 크기
```javascript
RELIC_POOL.length   // 렐릭 개수
CARD_POOL.length    // 카드 개수
```

#### 7-2. 동적 DOM 생성
```javascript
// map()으로 대량 요소 생성
picks.map(r => `<div>...</div>`).join('')
// → 큰 풀에서는 느릴 수 있음
```

### 최적화 기회
- 가상 스크롤링 (Virtual Scrolling)
- 요소 재사용 (Element Recycling)
- 렌더링 최적화

### 개선 효과
- ✅ 대량 데이터도 **부드럽게 렌더링**
- ✅ **메모리 사용 감소**

### 예상 작업량
- 약 100-200줄 (선택사항)

---

## 📋 우선순위 순서

### 🔴 높은 우선순위 (필수)
1. **Hover 텍스트 최적화** - 모바일 정보 접근성
2. **일시정지 메뉴 룬/렐릭 개선** - 게임 중 정보 확인

### 🟡 중간 우선순위 (권장)
3. **색상 대비 & 접근성** - 모든 사용자 포함
4. **Landscape 모드 최적화** - 다양한 플레이 스타일 지원

### 🟢 낮은 우선순위 (선택)
5. **폰트 일관성** - 향후 유지보수 용이
6. **추가 UI 최적화** - 마무리 작업
7. **성능 최적화** - 고급 최적화

---

## 📊 예상 작업량

| 카테고리 | 난이도 | 시간 | 라인 |
|---------|--------|------|------|
| 1. Hover 텍스트 | 쉬움 | 1-2시간 | 50-100 |
| 2. 룬/렐릭 개선 | 중간 | 1-2시간 | 50 |
| 3. 색상 대비 | 중간 | 1-2시간 | 30 |
| 4. Landscape | 중간 | 2-3시간 | 100 |
| 5. 폰트 일관성 | 어려움 | 2-3시간 | 150 |
| 6. 추가 UI | 쉬움 | 1시간 | 50 |
| 7. 성능 | 어려움 | 3-5시간 | 200 |
| **총계** | - | **11-18시간** | **~630줄** |

---

## 🎯 추천 다음 액션

### 즉시 추천 (Today)
```
1️⃣ Hover 텍스트 최적화 (1-2시간)
   - title 텍스트를 모바일에서 접근 가능하게

2️⃣ 일시정지 메뉴 룬/렐릭 개선 (1-2시간)
   - 게임 중 정보 확인 더 쉽게
```

### 단기 추천 (This Week)
```
3️⃣ 색상 대비 개선 (1-2시간)
   - 접근성 기준 충족
   
4️⃣ Landscape 최적화 (2-3시간)
   - 가로 모드 게임 플레이 지원
```

### 장기 권장 (This Month)
```
5️⃣ 폰트 일관성 (2-3시간)
6️⃣ 추가 UI 최적화 (1시간)
7️⃣ 성능 최적화 (3-5시간)
```

---

## 💡 추가 개선 아이디어

### 아이디어 1: 다크 모드
```css
@media (prefers-color-scheme: dark) {
  /* 자동 다크 모드 */
}
```
**우선순위**: 낮음 (현재 이미 다크 테마)

### 아이디어 2: 텍스트 크기 조정 옵션
```javascript
// 사용자가 UI 텍스트 크기를 조절
settings.fontScale = 1.0 ~ 1.5
```
**우선순위**: 낮음 (고급 기능)

### 아이디어 3: 고콘트라스트 모드
```
고콘트라스트 버튼 활성화 옵션
```
**우선순위**: 낮음 (접근성 옵션)

---

## 🎓 핵심 정리

### 지금까지 완료 ✅
- 12개 주요 팝업 최적화
- 모든 메인 UI 반응형화
- 기본 터치 최적화

### 다음 단계 (7가지 개선)
1. **필수**: Hover 텍스트, 룬/렐릭
2. **권장**: 색상 대비, Landscape
3. **선택**: 폰트 일관성, UI, 성능

### 최종 목표
**모든 모바일 사용자가 편하게 플레이할 수 있는 게임**

---

**Status**: 📋 **분석 완료, 구현 대기**
**Recommended Next**: Category 1-2 (Hover 텍스트 + 룬/렐릭)
**Estimated Impact**: 게임플레이 경험 +20%

