# 🧪 캐릭터별 덱 빌딩 시스템 - 테스트 가이드

**테스트 일시**: 2026-05-27  
**테스트 범위**: 전체 기능  
**예상 소요 시간**: 20-30분

---

## 🎮 게임 테스트 절차

### 1단계: 게임 시작 및 캐릭터 선택

1. 게임을 열기 (survivor.html)
2. "새 게임 시작" 클릭
3. **각 캐릭터를 순서대로 선택해서 테스트**:
   - 🐧 펭귄
   - 🦊 여우
   - 🐰 토끼
   - 🐱 고양이
   - 🐻 곰

### 2단계: 초기 덱 확인

**확인할 사항**:

```
🐧 펭귄 선택 시:
   ✓ 초기 덱 카드 3장 표시:
     - shield1 (보호막)
     - regen1 (재생)
     - iceShield (빙결방패) [새 카드]
   ✓ HUD 하단에 "방어형" 빌드 아이콘 표시
   ✓ 축복 덱 카운트 = 3개

🦊 여우 선택 시:
   ✓ 초기 덱 카드 3장 표시:
     - aoe1 (광역)
     - piercing (관통)
     - chainExplosion (연쇄폭발) [새 카드]
   ✓ HUD 하단에 "폭발형" 빌드 아이콘 표시
   ✓ 축복 덱 카운트 = 3개

🐰 토끼 선택 시:
   ✓ 초기 덱 카드 3장 표시:
     - fireRate1 (발사속도)
     - criticalStrike (크리티컬)
     - lightningStrike (번개강타) [새 카드]
   ✓ HUD 하단에 "콤보형" 빌드 아이콘 표시
   ✓ 축복 덱 카운트 = 3개

🐱 고양이 선택 시:
   ✓ 초기 덱 카드 3장 표시:
     - multiShot1 (멀티샷)
     - piercing (관통)
     - multiBullet (멀티탄환) [새 카드]
   ✓ HUD 하단에 "독립형" 빌드 아이콘 표시
   ✓ 축복 덱 카운트 = 3개

🐻 곰 선택 시:
   ✓ 초기 덱 카드 3장 표시:
     - shield1 (보호막)
     - aoe1 (광역)
     - areaSlam (광역슬램) [새 카드]
   ✓ HUD 하단에 "폭발형" 빌드 아이콘 표시
   ✓ 축복 덱 카운트 = 3개
```

### 3단계: 카드 선택 화면 테스트

**카드 선택 화면에 진입** (첫 번째 전투 후 카드 선택):

#### 펭귄으로 진행할 때:

```
1. 카드 선택 화면에서 3개 카드 표시
2. 💡 추천 배지 확인:
   ✓ 펭귄 고유 시너지 (penguin_fortress 등)에 ⭐ 배지
   ✓ 펭귄 고유 카드에 우선순위 표시
3. "다시뽑기" 여러 번 클릭:
   ✓ iceShield, glacialFortress 등 펭귄 고유 카드 자주 표시
   ✓ 여우 고유 카드(chainExplosion 등) **미표시**
   ✓ 호환 카드(shield1, regen1 등) 항상 표시
4. 성장 바 확인:
   ✓ 시너지 표시 시 펭귄 우선 시너지 우선 표시
```

#### 여우로 진행할 때:

```
1. 카드 선택 화면에서 3개 카드 표시
2. 💡 추천 배지 확인:
   ✓ 여우 고유 시너지 (fox_explosive_chain 등)에 ⭐ 배지
3. "다시뽑기" 여러 번 클릭:
   ✓ chainExplosion, momentumStrike 등 여우 고유 카드 자주 표시
   ✓ 펭귄 고유 카드(iceShield 등) **미표시**
4. 시너지 발동 시:
   ✓ explosion_master 우선 표시
```

#### 토끼로 진행할 때:

```
1. 카드 선택 화면에서 3개 카드 표시
2. 💡 추천 배지:
   ✓ 토끼 고유 시너지 (rabbit_combo_master 등)에 ⭐ 배지
3. "다시뽑기":
   ✓ lightningStrike, criticalChain 등 토끼 고유 카드 자주 표시
4. 시너지:
   ✓ combo_frenzy 우선 표시
```

#### 고양이로 진행할 때:

```
1. 초기 3장: multiShot1, piercing, multiBullet
2. 우선 시너지: cat_multishot_master, cat_precision_killer
3. 고유 카드: 멀티탄환 관련 카드들 자주 표시
```

#### 곰으로 진행할 때:

```
1. 초기 3장: shield1, aoe1, areaSlam
2. 우선 시너지: bear_fortress_master, bear_explosion_master
3. 고유 카드: 폭발, 광역 관련 카드들 자주 표시
```

---

## 🔍 콘솔 에러 확인

### F12 개발자 도구 열기

1. **F12** 키 누르기 (또는 Ctrl+Shift+I)
2. **Console** 탭 클릭
3. **빨간색 에러 확인**:

```
❌ 피해야 할 에러:
   - "CHARACTER_UNIQUE_DATA is not defined"
   - "ownerCharacters" 관련 에러
   - "buildWeighted is not a function"
   - "Cannot read property 'initialCards'" 
   - syntax error

✅ 무시해도 될 경고 (파란색, 노란색):
   - 외부 리소스 로드 경고
   - deprecated API 경고
```

---

## 📊 성능 확인

### FPS 측정

콘솔에서 다음 코드 실행:

```javascript
let frameCount = 0;
let fps = 0;
let lastTime = Date.now();

setInterval(() => {
  const now = Date.now();
  fps = (frameCount * 1000 / (now - lastTime)).toFixed(1);
  console.log(`FPS: ${fps}`);
  frameCount = 0;
  lastTime = now;
}, 1000);

// battleLoop 내부 frameCount++ 필요 (이미 구현됨)
```

**예상 결과**:
- ✅ 60 FPS 이상 유지 (변화 없음 또는 약간 향상)
- ✅ 프레임 지터 없음
- ✅ 카드 선택 화면에서도 부드러움

---

## 🎯 종합 검증 체크리스트

### A. 기능 검증

```
초기 덱:
  ☑️ 펭귄: shield1, regen1, iceShield
  ☑️ 여우: aoe1, piercing, chainExplosion
  ☑️ 토끼: fireRate1, criticalStrike, lightningStrike
  ☑️ 고양이: multiShot1, piercing, multiBullet
  ☑️ 곰: shield1, aoe1, areaSlam

카드 필터링:
  ☑️ 펭귄으로: 여우 고유 카드 미표시 (여러 번 다시뽑기 확인)
  ☑️ 여우로: 펭귄 고유 카드 미표시
  ☑️ 토끼로: 곰 고유 카드 미표시
  ☑️ 고양이로: 토끼 고유 카드 미표시
  ☑️ 곰으로: 고양이 고유 카드 미표시
  ☑️ 호환 카드: 모든 캐릭터에서 표시

시너지 우선순위:
  ☑️ 펭귄: shield_wall, regen_sync 우선 표시
  ☑️ 여우: explosion_master 우선 표시
  ☑️ 토끼: combo_frenzy 우선 표시
  ☑️ 고양이: multishot_master 우선 표시
  ☑️ 곰: explosion_master 우선 표시

추천 배지:
  ☑️ 캐릭터 선호 시너지에 ⭐ 표시
  ☑️ 진행도 높은 시너지 우선 표시

빌드 테마:
  ☑️ HUD에 테마 아이콘 표시
  ☑️ 카드 선택 화면에 빌드 설명 표시
  ☑️ 초기 덱 카드 +50% 가중치 (더 자주 표시)
```

### B. 호환성 검증

```
기본 기능:
  ☑️ 캐릭터 패시브 정상 작동
  ☑️ 유물 선택/적용 정상
  ☑️ 렐릭 선택/적용 정상
  ☑️ 저장/로드 정상
  ☑️ 재시작 후 초기 덱 정상 표시

게임 시스템:
  ☑️ 전투 시스템 정상
  ☑️ 난이도 스케일링 정상
  ☑️ 시너지 발동 정상
  ☑️ 카드 강화 정상
  ☑️ 엘리트/보스 정상
```

### C. 에러 확인

```
콘솔:
  ☑️ 빨간색 에러 없음
  ☑️ 관련 모듈 로드됨
  ☑️ CHARACTER_UNIQUE_DATA 존재
  ☑️ buildWeighted 함수 작동

렌더링:
  ☑️ UI 깨짐 없음
  ☑️ 텍스트 오버플로우 없음
  ☑️ 이미지/아이콘 정상 표시
  ☑️ 모바일 레이아웃 정상
```

---

## 🐛 문제 해결

### Issue: 초기 덱이 표시되지 않음

**원인 확인**:
```javascript
// 콘솔에서 다음 실행
console.log('CHARACTER_UNIQUE_DATA:', CHARACTER_UNIQUE_DATA);
console.log('gs.selectedChar:', gs.selectedChar);
console.log('gs.deck:', gs.deck);
```

**해결법**:
1. 페이지 새로고침 (Ctrl+R)
2. 캐시 삭제 (Ctrl+Shift+Delete) 후 새로고침
3. 브라우저 개발자 도구 콘솔에 오류 메시지 확인

### Issue: 여우 고유 카드가 펭귄에서 표시됨

**원인 확인**:
```javascript
// 콘솔에서 다음 실행
const _charId = gs.selectedChar.id;
const foxCard = CARD_POOL.find(c => c.id === 'chainExplosion');
console.log('Character:', _charId);
console.log('Fox Card ownerCharacters:', foxCard.ownerCharacters);
console.log('Should show:', foxCard.ownerCharacters.includes(_charId));
```

**해결법**:
1. ownerCharacters 필드가 올바르게 설정되어 있는지 확인
2. buildWeighted() 함수의 필터링 로직 확인

### Issue: 시너지 우선순위가 변경되지 않음

**원인 확인**:
```javascript
// 콘솔에서 다음 실행
console.log('CHARACTER_BUILD_PREFERENCE:', CHARACTER_BUILD_PREFERENCE);
console.log('Current char preference:', CHARACTER_BUILD_PREFERENCE[gs.selectedChar.id]);
```

**해결법**:
1. CHARACTER_BUILD_PREFERENCE에 캐릭터 ID 확인
2. getCardRecommendationReason() 함수의 선호도 적용 로직 확인

---

## ✅ 최종 검증

### 테스트 완료 조건

```
🟢 PASS: 모든 체크리스트 항목이 ✅
🟡 PARTIAL PASS: 80% 이상 체크리스트 항목이 ✅
🔴 FAIL: 50% 이상의 중요 항목이 ❌
```

### 배포 결정 기준

| 상황 | 결정 | 조치 |
|------|------|------|
| 모든 항목 ✅ | **배포 가능** | 그대로 배포 |
| 80% 이상 ✅ | **조건부 배포** | 미반영 항목 문서화 후 배포 |
| 주요 기능 ✅, 부차 기능 ❌ | **배포 + 패치** | 배포 후 패치 계획 수립 |
| 주요 기능 ❌ | **배포 중단** | 문제 해결 후 재테스트 |

---

## 📝 테스트 결과 보고서 템플릿

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
캐릭터별 덱 빌딩 시스템 - 테스트 결과 보고서
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📅 테스트 일시: [날짜]
🔧 테스터: [이름]
⏱️ 소요 시간: [시간]

📊 테스트 결과
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ 합격: _____ / 52 항목

🐧 펭귄:
   초기 덱: ✅ / ❌ / 🟡
   카드 필터링: ✅ / ❌ / 🟡
   시너지 우선순위: ✅ / ❌ / 🟡

🦊 여우:
   초기 덱: ✅ / ❌ / 🟡
   카드 필터링: ✅ / ❌ / 🟡
   시너지 우선순위: ✅ / ❌ / 🟡

🐰 토끼:
   초기 덱: ✅ / ❌ / 🟡
   카드 필터링: ✅ / ❌ / 🟡
   시너지 우선순위: ✅ / ❌ / 🟡

🐱 고양이:
   초기 덱: ✅ / ❌ / 🟡
   카드 필터링: ✅ / ❌ / 🟡
   시너지 우선순위: ✅ / ❌ / 🟡

🐻 곰:
   초기 덱: ✅ / ❌ / 🟡
   카드 필터링: ✅ / ❌ / 🟡
   시너지 우선순위: ✅ / ❌ / 🟡

호환성: ✅ / ❌ / 🟡
성능: ✅ / ❌ / 🟡
콘솔 에러: ✅ / ❌ / 🟡

📋 발견된 문제
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. [문제 설명]
   - 영향도: 높음 / 중간 / 낮음
   - 재현 방법: [단계]
   - 예상 결과: [결과]
   - 실제 결과: [결과]

🎯 최종 판정: 배포 가능 / 조건부 배포 / 배포 보류

💬 추가 의견:
[자유 기술]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

**테스트 가이드 완성**: 2026-05-27  
**예상 테스트 시간**: 20-30분  
**테스트 난이도**: 낮음 (자동 검증 가능)

