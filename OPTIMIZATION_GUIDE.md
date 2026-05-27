# Survivor.HTML 코드 최적화 실행 가이드

## 현황 요약
- **완료**: Phase 1 (중복 함수 제거, 약 14줄 삭제)
- **다음**: Phase 2-7 구현
- **목표**: 파일 크기 2-3% 감소, 성능 1-3% 향상, 가독성 30% 향상

---

## Phase 2: 그래디언트 캐싱 확장 (30분)

### 현황 분석
- **createLinearGradient 호출**: 77회
- **createRadialGradient 호출**: 여러 번
- **문제**: 많은 호출이 매 프레임 반복
- **캐시 메커니즘**: 현재 없음

### 구현 방법

#### Step 1: 그래디언트 캐시 객체 생성 (라인 1500 근처)
```javascript
// 게임 시작 시 추가
const gradientCache = {};

function getOrCreateGradient(ctx, type, ...args) {
  // type: 'linear' 또는 'radial'
  const key = [type, ...args].join('|');
  
  if (!gradientCache[key]) {
    if (type === 'linear') {
      const [x0, y0, x1, y1] = args;
      gradientCache[key] = ctx.createLinearGradient(x0, y0, x1, y1);
    } else if (type === 'radial') {
      const [x0, y0, r0, x1, y1, r1] = args;
      gradientCache[key] = ctx.createRadialGradient(x0, y0, r0, x1, y1, r1);
    }
  }
  
  return gradientCache[key];
}
```

#### Step 2: 자주 사용되는 그래디언트 사전 생성
```javascript
function preloadCommonGradients(ctx) {
  // HP 바 그래디언트
  const hpHealthy = ctx.createLinearGradient(0, 0, 100, 0);
  hpHealthy.addColorStop(0, '#4ecf7a');
  hpHealthy.addColorStop(1, '#7ef0a0');
  gradientCache['hp-healthy'] = hpHealthy;
  
  // HP 바 (위험)
  const hpDanger = ctx.createLinearGradient(0, 0, 100, 0);
  hpDanger.addColorStop(0, '#e05050');
  hpDanger.addColorStop(1, '#f07050');
  gradientCache['hp-danger'] = hpDanger;
  
  // 추가로 필요한 그래디언트...
}
```

#### Step 3: drawBattle 내에서 기존 createLinearGradient 호출 교체
```javascript
// 변경 전:
const bg = ctx.createLinearGradient(x, y, x + w, y + h);
bg.addColorStop(0, color1);
bg.addColorStop(1, color2);

// 변경 후:
const bg = getOrCreateGradient(ctx, 'linear', x, y, x + w, y + h);
if (!bg._initialized) {
  bg.addColorStop(0, color1);
  bg.addColorStop(1, color2);
  bg._initialized = true;
}
```

### 성능 효과
- **예상 성능 향상**: 1-2% FPS 증가
- **메모리 추가**: 캐시 크기 제한으로 미미 (500개 그래디언트 = ~50KB)

---

## Phase 3: 거리 계산 최적화 (20분)

### 현황 분석
- **Math.hypot 호출**: 31회
- **문제**: sqrt 연산은 비용이 큼, 거리 비교만 필요한 경우 많음
- **최적화**: 거리^2 기반 비교 함수 도입

### 구현 방법

#### Step 1: distSquared() 함수 추가 (라인 1600 근처)
```javascript
// 거리의 제곱을 반환 (sqrt 없음)
function distSquared(x1, y1, x2, y2) {
  const dx = x2 - x1;
  const dy = y2 - y1;
  return dx * dx + dy * dy;
}

// 거리를 반환 (실제 거리 필요할 때만)
function distance(x1, y1, x2, y2) {
  return Math.sqrt(distSquared(x1, y1, x2, y2));
}
```

#### Step 2: 거리 비교 호출 교체
```javascript
// 패턴 1: 범위 확인
// 변경 전: if(Math.hypot(e.x-p.x,e.y-p.y)<120)
// 변경 후: if(distSquared(e.x,e.y,p.x,p.y)<14400)  // 120^2=14400

// 패턴 2: 거리 계산 (실제 값 필요)
// 변경 전: const dist = Math.hypot(dx, dy);
// 변경 후: const dist = distance(x1, y1, x2, y2);
```

#### Step 3: 최적화 대상 (priority 순서)
1. **라인 1790**: 적 정렬 (매 프레임, 많은 적)
2. **라인 9407**: 플레이어 업데이트 루프
3. **라인 9938**: 얼음 시너지 적용
4. **라인 10245**: 플레이어 충돌 확인
5. **라인 10678**: 아이템 수집 거리

### 성능 효과
- **예상 성능 향상**: 0.5-1% FPS 증가
- **특히 효과적**: 많은 적/아이템이 화면에 있을 때

---

## Phase 4: drawBattle 함수 분할 (2-3시간)

### 현황
- **총 줄 수**: 1,314줄 (라인 11363-12677)
- **단일 책임 위반**: 배경/적/탄환/UI 모두 렌더링
- **목표**: 7개 함수로 분할

### 분할 구조

#### 함수 목록
```
drawBattle() [1314줄]
├─ renderBackground() [150줄]
│  └─ 배경, 그리드, 파티클 렌더링
├─ renderPlayer() [100줄]
│  └─ 플레이어, 칼, 선택 원 렌더링
├─ renderEnemies() [200줄]
│  └─ 적, 보스, 상태 이펙트 렌더링
├─ renderBullets() [150줄]
│  └─ 탄환, 궤적, 충돌 이펙트 렌더링
├─ renderUI() [200줄]
│  └─ 프레임, 웨이브, 경험치, 성장바 렌더링
├─ renderEffects() [150줄]
│  └─ 데미지 번호, 크리티컬, 플래시 이펙트
└─ renderHUD() [150줄]
   └─ 능력, 렐릭, 카드 선택 표시
```

### 구현 방법

#### 원본 drawBattle 구조 분석
원본 drawBattle 함수 라인 11363-12677의 주요 섹션:
1. Canvas 컨텍스트 기본 설정 (줄 11363-11380)
2. 배경 렌더링 (줄 11380-11450)
3. 파티클 렌더링 (줄 11450-11500)
4. 적 렌더링 (줄 11500-11700)
5. 플레이어 렌더링 (줄 11700-11800)
6. 탄환 렌더링 (줄 11800-11900)
7. UI 렌더링 (줄 11900-12100)
8. 이펙트 렌더링 (줄 12100-12300)
9. HUD 렌더링 (줄 12300-12677)

#### 추출 전략
1. **공유 변수 정의** (drawBattle 최상단)
```javascript
function drawBattle() {
  // 모든 함수가 접근 가능한 변수
  const ctx = c.getContext('2d');
  const mctx = mCanvas.getContext('2d');
  const cvs = { canvas: c, ctx: ctx, mCanvas: mCanvas, mCtx: mctx };
  
  // 렌더링 실행
  renderBackground(cvs);
  renderEnemies(cvs);
  renderPlayer(cvs);
  renderBullets(cvs);
  renderUI(cvs);
  renderEffects(cvs);
  renderHUD(cvs);
}
```

2. **각 함수 추출**
```javascript
function renderBackground(cvs) {
  // 배경 관련 전체 코드
  // 원본 drawBattle의 배경 섹션 이동
}

function renderEnemies(cvs) {
  // 적 관련 전체 코드
}

// ... 나머지 함수들
```

#### 주의사항
- Z-index 순서 유지 (배경 < 객체 < UI < HUD)
- cvs 객체를 통한 컨텍스트 전달
- 각 함수는 독립적으로 테스트 가능하도록 구조화
- 성능 측정 (분할 전후 FPS 비교)

### 예상 효과
- **코드 가독성**: 50% 향상
- **유지보수 효율**: 3배 향상
- **버그 수정 시간**: 50% 단축
- **성능**: 변화 없거나 0.5% 향상

---

## Phase 5: updateBattle 함수 분할 (2-3시간)

### 현황
- **총 줄 수**: 1,195줄 (라인 9623-10818)
- **문제**: 플레이어/적/탄환/타이밍 모두 혼합
- **목표**: 5개 함수로 분할

### 분할 구조
```
updateBattle()
├─ updatePlayer() [250줄]
│  └─ 플레이어 이동, 회전, 애니메이션
├─ updateEnemies() [300줄]
│  └─ 적 AI, 이동, 피해, 사망
├─ updateBullets() [200줄]
│  └─ 탄환 이동, 충돌, 제거
├─ updateTiming() [150줄]
│  └─ 웨이브, 전투, 타이머 관리
└─ updateCamera() [100줄]
   └─ 카메라 흔들림, 스크린 셰이크
```

### 구현 순서
1. **의존성 그래프 작성**: 어떤 함수가 어떤 데이터에 의존하는지
2. **업데이트 순서 결정**: 플레이어 → 적 → 탄환 → 타이밍
3. **각 함수 추출**: 독립적으로 테스트 가능하도록
4. **통합 테스트**: 게임 로직 일관성 확인

### 성능 고려사항
- 업데이트 순서 변경 금지 (물리 계산 일관성)
- 각 함수 처리 시간 모니터링
- 미세한 로직 오류 검증

---

## Phase 6: drawHUD 함수 분할 (1-1.5시간)

### 현황
- **총 줄 수**: 895줄 (라인 12680-13575)
- **문제**: UI 요소 혼합, 복잡한 레이아웃
- **목표**: 4개 함수로 분할

### 분할 구조
```
drawHUD()
├─ drawStats() [150줄] → HP, 렉, 경험치, 렐릭
├─ drawGrowthBar() [200줄] → 성장바, 시너지, 보너스
├─ drawCards() [300줄] → 카드 선택 화면 (가장 큼)
└─ drawTutorial() [200줄] → 튜토리얼, 가이드
```

### 구현 시 주의
- 터치/마우스 이벤트 리스너 위치 유지
- 모바일/데스크톱 레이아웃 호환성
- Z-index 순서 정확성

---

## Phase 7: 메모리 최적화 (30분)

### 1. 렐릭 계산 캐싱
```javascript
// 게임 상태 변수
gs._relicsBulletMultCache = null;
gs._relicsDamageCache = null;

function relicsBulletMultCached() {
  if (!gs._relicsBulletMultCache) {
    gs._relicsBulletMultCache = relicsBulletMult();
  }
  return gs._relicsBulletMultCache;
}

// 덱 변경 시 캐시 무효화
function onDeckChange() {
  gs._relicsBulletMultCache = null;
  gs._relicsDamageCache = null;
}
```

### 2. 임시 객체 재사용
```javascript
// 벡터 계산용 재사용 객체
const _tempVec = { x: 0, y: 0 };

function someFunction() {
  _tempVec.x = newX;
  _tempVec.y = newY;
  // 사용...
}
```

### 3. 이벤트 리스너 정리
```javascript
function cleanupGame() {
  window.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('touchmove', onTouchMove);
  // ... 기타 리스너
}
```

---

## 전체 구현 체크리스트

### Phase 1: 중복 제거
- [x] spawnP/spawnDN 중복 삭제

### Phase 2: 그래디언트 캐싱
- [ ] gradientCache 객체 생성
- [ ] getOrCreateGradient() 함수 추가
- [ ] preloadCommonGradients() 구현
- [ ] drawBattle 내 호출 교체
- [ ] 캐시 크기 모니터링

### Phase 3: 거리 최적화
- [ ] distSquared() 함수 추가
- [ ] 라인 1790의 거리 비교 최적화
- [ ] 라인 9407 근처 최적화
- [ ] 나머지 호출 교체
- [ ] 성능 측정

### Phase 4: drawBattle 분할
- [ ] 함수 구조 설계
- [ ] renderBackground() 추출
- [ ] renderPlayer() 추출
- [ ] renderEnemies() 추출
- [ ] renderBullets() 추출
- [ ] renderUI() 추출
- [ ] renderEffects() 추출
- [ ] renderHUD() 추출
- [ ] Z-index 순서 검증
- [ ] 성능 테스트

### Phase 5: updateBattle 분할
- [ ] 의존성 분석
- [ ] updatePlayer() 추출
- [ ] updateEnemies() 추출
- [ ] updateBullets() 추출
- [ ] updateTiming() 추출
- [ ] updateCamera() 추출
- [ ] 게임 로직 검증
- [ ] 성능 테스트

### Phase 6: drawHUD 분할
- [ ] drawStats() 추출
- [ ] drawGrowthBar() 추출
- [ ] drawCards() 추출
- [ ] drawTutorial() 추출
- [ ] 터치 이벤트 검증
- [ ] 모바일 레이아웃 확인

### Phase 7: 메모리 최적화
- [ ] 렐릭 캐싱 구현
- [ ] 임시 객체 재사용 세팅
- [ ] 이벤트 리스너 정리
- [ ] 메모리 누수 검증

### 통합 테스트
- [ ] 전체 게임 플레이 (5분)
- [ ] 모든 캐릭터 테스트
- [ ] 모든 Act 완주
- [ ] 콘솔 에러 확인
- [ ] FPS 측정
- [ ] 메모리 모니터링

---

## 예상 완료 일정
- **DAY 1**: Phase 1, 2, 3 (1시간)
- **DAY 2**: Phase 4, 6 (3.5시간)
- **DAY 3**: Phase 5 (2-3시간)
- **DAY 4**: Phase 7 + 테스트 (1.5시간)

**총 소요 시간**: 약 8-10시간

---

## 최적화 성공 기준
1. ✅ 파일 크기 2-3% 감소 (약 500-900줄)
2. ✅ FPS 1-3% 향상 또는 동일
3. ✅ 메모리 누수 없음
4. ✅ 모든 함수 < 300줄 (권장 < 200줄)
5. ✅ 중복 제거 완료
6. ✅ 콘솔 에러 없음
7. ✅ 코드 가독성 30% 향상 (주관적 평가)
