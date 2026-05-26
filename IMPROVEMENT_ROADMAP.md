# 🛣️ Last Animal Standing - 게임 개선 로드맵

> **작성일**: 2026-05-26  
> **상태**: 개선 가능 영역 분석 완료

---

## 📊 개선 우선순위 분류

### 🔴 **우선순위 HIGH** (사용자 경험에 직접 영향)

#### 1. **튜토리얼/온보딩 시스템** ⭐⭐⭐
**현재 상태**: 게임 시작 시 빌드 선택만 함
**문제점**:
- 신규 플레이어가 게임 메커니즘을 모름
- 첫 번째 게임이 혼란스러울 수 있음
- 조작법이 명확하지 않음

**개선안**:
```javascript
// 첫 게임 플레이 감지
if (localStorage.getItem('firstPlay') === null) {
  // 튜토리얼 활성화
  showTutorial([
    '화면 클릭으로 방향 조정',
    '스페이스바로 대시 (대시 스킬 습득 시)',
    '화면 좌측에 HUD 정보 표시',
    '적을 처치하면 카드 3장 중 선택',
    '휴식처에서 HP 회복 또는 능력 강화',
  ]);
  localStorage.setItem('firstPlay', 'done');
}
```

**예상 효과**: 신규 플레이어 이탈율 감소, 게임 이해도 상승

---

#### 2. **시너지 표시 개선** ⭐⭐⭐
**현재 상태**: 시너지는 활성화되지만, 플레이어가 명확히 알기 어려움
**문제점**:
- 시너지 활성화 시 은은한 토스트 메시지만 표시
- 현재 활성 시너지 목록이 명확하지 않음
- 어떤 카드 조합이 시너지인지 불명확

**개선안**:
```
추가 UI 패널:
┌─────────────────────────────┐
│  ✨ 활성 시너지              │
├─────────────────────────────┤
│ 🗡️ 검 마스터                 │
│ └─ 검 데미지 +30%           │
│                             │
│ ⚡ 뇌빙 폭풍 (2/3)           │
│ └─ 번개 + 빙결 = 정전 효과  │
│                             │
│ 💎 금 탐욕 (1/2)            │
│ └─ 한 장만 더...             │
└─────────────────────────────┘
```

**구현 복잡도**: 중간 (UI 추가, 시너지 체크 로직)

---

#### 3. **카드 추천/힌트 시스템** ⭐⭐⭐
**현재 상태**: 카드만 표시되고 선택지만 제공
**문제점**:
- 플레이어가 어떤 카드를 선택해야 할지 모름
- 현재 빌드에 맞는 카드를 구분 못 함

**개선안**:
```javascript
// 카드 선택 시 추천 표시
function suggestCard(card, currentBuild) {
  let suggestion = '';
  let emoji = '';
  
  if (currentBuild.sword > 0 && card.id === 'sword1') {
    suggestion = '🗡️ 검 빌드와 완벽 조화!';
    emoji = '⭐';
  } else if (hasActivesynergy(card)) {
    suggestion = '✨ 시너지 활성화!';
    emoji = '🌟';
  } else if (isWeakCard(card, currentBuild)) {
    suggestion = '약간 아쉬운 선택이에요';
    emoji = '😔';
  }
  
  showSuggestion(emoji, suggestion);
}
```

**예상 효과**: 신규 플레이어의 의사결정 도움, 더 나은 빌드 구성

---

### 🟡 **우선순위 MEDIUM** (게임플레이 개선)

#### 4. **라운드/웨이브 전환 연출** ⭐⭐
**현재 상태**: 적이 사라지고 새로 나타남
**개선 아이디어**:
- 다음 웨이브 카운트다운 (3... 2... 1...)
- 적의 개수/강도 표시
- 보스 웨이브 전환 시 특별한 연출

```javascript
// 웨이브 시작 전 연출
async function showWaveTransition(waveNum, enemyCount, isBossWave) {
  const text = isBossWave 
    ? `⚔️ WAVE ${waveNum} - 보스 출현!` 
    : `WAVE ${waveNum} - 적 ${enemyCount}마리`;
  
  showBigText(text, 1500);
  await sleep(800);
  playWaveStartSound();
  bt.screenShake = 12;
}
```

---

#### 5. **보스 전투 입장 연출** ⭐⭐
**현재 상태**: 보스가 일반 적처럼 나타남
**개선안**:
- 보스 이름과 설명 표시
- 화면 흔들림과 극적인 음향
- 보스 체력바를 더 눈에 띄게

```javascript
// 보스 전투 시작 시
function showBossIntro(boss) {
  freezeGame(2000);
  showBossCard({
    name: boss.name,
    emoji: boss.emoji,
    description: '강력한 적이 나타났다!',
  });
  Audio.sfx.bossAppear();
  bt.screenShake = 30;
}
```

---

#### 6. **게임 내 팁/혼잣말 시스템** ⭐⭐
**현재 상태**: 튜토리얼 외 팁이 없음
**개선 아이디어**:
```
화면 모서리에 간단한 팁:
• "💬 여러 번 대시하면 적을 피할 수 있어"
• "💬 2개 이상의 번개로 체인 반응이 일어나!"
• "💬 보호막으로 보스의 한 방을 막아내자"
• "💬 시너지를 이루면 효과가 강해진다"
```

---

#### 7. **카메라 흔들림 효과의 깊이** ⭐⭐
**현재 상태**: 기본적인 screenShake만 구현
**개선안**:
- 보스 공격 시 더 강한 진동
- 멀티킬 시 느린 모션 + 강한 진동
- 대미지 크기에 따른 차등 진동

```javascript
function improvedScreenShake(magnitude, duration, curve) {
  // magnitude: 진동 강도 (1-30)
  // duration: 지속 시간 (ms)
  // curve: 'linear' / 'ease-out' / 'ease-in'
  
  bt.screenShake = magnitude;
  bt.screenShakeCurve = curve;
  setTimeout(() => bt.screenShake = 0, duration);
}
```

---

#### 8. **승리/패배 연출 개선** ⭐⭐
**현재 상태**: 
- 패배: 검은 화면 → 게임 오버 팝업
- 승리: 보스 효과 → 엔딩 화면

**개선안**:
```javascript
// 패배 연출
function enhancedDefeatAnimation() {
  // 1. 플레이어 캐릭터 투명도 감소
  // 2. 캔버스 색상 어두워짐 (검은색 페이드)
  // 3. 비통성 음향 재생
  // 4. 화면 중앙에 "패배" 텍스트 표시
  // 5. 느린 페이드로 게임 오버 화면으로 전환
}

// 승리 연출
function enhancedVictoryAnimation() {
  // 1. 보스 사망 시 강한 폭발
  // 2. 화면 밝기 증가
  // 3. 승리 음악 재생
  // 4. 캐릭터가 빛에 싸여 올라감
  // 5. 천천히 엔딩 화면으로 전환
}
```

---

### 🟢 **우선순위 LOW** (기능 개선)

#### 9. **게임 설정 UI** ⭐
**기능 추가**:
- 🔊 사운드 볼륨 조절
- 🔆 화면 밝기 조절
- ⏱️ 게임 속도 (1x / 1.25x / 1.5x)
- 🎨 색맹 모드 (색상 필터)

```html
<div class="settings-panel">
  <label>🔊 음량
    <input type="range" min="0" max="100" value="80" onchange="setVolume(this.value)">
  </label>
  <label>⏱️ 게임 속도
    <select onchange="setGameSpeed(this.value)">
      <option value="1">1x (정상)</option>
      <option value="1.25">1.25x (빠름)</option>
      <option value="1.5">1.5x (매우 빠름)</option>
    </select>
  </label>
</div>
```

---

#### 10. **통계 추적 시스템** ⭐
**추가 정보**:
- 📊 총 플레이 시간
- 💀 모든 시간의 총 처치 수
- 🏆 최고 도달 층수
- 🎯 가장 많이 사용한 카드
- ⚡ 최고 콤보 기록

```javascript
stats = {
  totalPlayTime: 12345, // seconds
  totalKills: 5432,
  maxAct: 5,
  avgAct: 2.3,
  favoriteCard: 'sword1',
  maxCombo: 47,
  runsCompleted: 12,
  avgRunLength: 8.2, // minutes
}
```

---

#### 11. **스킬 시너지 가이드** ⭐
**목적**: 게임 내 스킬 조합 정보 제공

```javascript
// 시너지 가이드 예시
const synergyGuide = [
  {
    name: '검 마스터',
    cards: ['sword1', 'sword1', 'sword1'],
    effect: '회전 검 데미지 +30%',
    description: '검을 3개 이상 모으면 강력한 회전검 마스터가 된다'
  },
  {
    name: '뇌빙 폭풍',
    cards: ['lightning1', 'freeze1'],
    effect: '번개 적중 시 적 빙결',
    description: '번개와 빙결을 함께 사용하면 정전 효과!'
  }
];
```

---

#### 12. **성능 프로파일링 & 최적화** ⭐
**분석 대상**:
- 🎬 FPS 모니터링 (현재는 60fps 고정인가?)
- 💾 메모리 사용량
- ⏱️ 프레임당 처리 시간

**구현**:
```javascript
const perf = {
  fps: 60,
  avgFrameTime: 16.67, // ms
  memoryUsage: 0, // MB
  particleCount: 0,
  enemyCount: 0,
};

// 성능 대시보드 (개발자 모드)
if (DEBUG_MODE) {
  drawDebugInfo({
    fps: perf.fps,
    particles: perf.particleCount,
    enemies: perf.enemyCount,
    memory: `${Math.round(performance.memory?.usedJSHeapSize / 1048576)}MB`,
  });
}
```

---

#### 13. **모바일 세로/가로 전환 개선** ⭐
**현재 상태**: 가로 전환 대응은 되어 있지만 UX 개선 가능
**개선 아이디어**:
- 화면 회전 시 게임 일시정지
- "화면을 세로로 유지하세요" 가이드
- 세로 모드에서도 플레이 가능하도록 UI 조정

---

#### 14. **아키브먼트/미션 시스템** ⭐
**아이디어**:
```javascript
const achievements = [
  { id: 'first_kill', name: '첫 처치', description: '첫 적을 처치하기' },
  { id: 'boss_hunter', name: '보스 사냥꾼', description: '5개 보스 처치' },
  { id: 'master_builder', name: '빌드 마스터', description: '시너지 5개 활성화' },
  { id: 'speed_runner', name: '스피드런', description: '5분 이내에 3층 도달' },
  { id: 'perfect_run', name: '완벽한 여정', description: '손상 없이 3층 클리어' },
];
```

---

## 📈 개선 효과 예상

### 단기 (1-2주)
- ✅ 신규 플레이어 튜토리얼 추가 → 이탈율 ↓ 20%
- ✅ 시너지 UI 개선 → 사용자 만족도 ↑ 15%
- ✅ 카드 추천 시스템 → 게임 이해도 ↑ 25%

### 중기 (3-4주)
- ✅ 보스 연출 개선 → 게임플레이 임팩트 ↑ 30%
- ✅ 게임 내 팁 시스템 → 게임 직관성 ↑ 20%
- ✅ 설정 UI 추가 → 플레이 만족도 ↑ 10%

### 장기 (1개월+)
- ✅ 성능 최적화 → 모바일 안정성 ↑ 40%
- ✅ 통계 시스템 → 재플레이 의욕 ↑ 35%
- ✅ 아키브먼트 → 장기 활성화 사용자 ↑ 50%

---

## 🎯 권장 구현 순서

### Phase 1: 사용자 경험 개선 (2주)
1. 튜토리얼 시스템 ⭐⭐⭐
2. 시너지 UI 개선 ⭐⭐⭐
3. 카드 추천 시스템 ⭐⭐⭐

### Phase 2: 게임플레이 향상 (2주)
4. 보스 전투 연출 ⭐⭐
5. 라운드 전환 연출 ⭐⭐
6. 게임 내 팁 시스템 ⭐⭐

### Phase 3: 기능 확장 (3주)
7. 게임 설정 UI ⭐
8. 통계 추적 ⭐
9. 성능 프로파일링 ⭐
10. 아키브먼트 시스템 ⭐

---

## 💡 빠른 승리 (Quick Wins)

**높은 영향도, 낮은 난이도**:

```javascript
// 1. 화면 쓸 때 팁 표시 (5분)
function showRandomTip() {
  const tips = [
    '💬 대시로 적의 총알을 피하세요!',
    '💬 휴식처에서 HP를 회복할 수 있어요',
    '💬 같은 카드를 여러 번 선택하면 더 강해져요',
  ];
  showToast(tips[Math.random() * tips.length | 0], 3000);
}

// 2. 게임 오버 화면에 다음 팁 (10분)
const nextGameTip = randomTip();
overlay.innerHTML += `<div class="next-tip">💡 다음 도전: ${nextGameTip}</div>`;

// 3. 보스 이름을 HD하이라이트 (15분)
bossFightStart.style.color = boss.color;
bossFightStart.textShadow = `0 0 20px ${boss.color}`;
```

---

## 📝 추후 고려사항

- **멀티플레이어**: 로컬/온라인 코옵 가능성 검토
- **시즌 시스템**: 월별 특별 챌린지/리더보드
- **커스터마이제이션**: 캐릭터 스킨, 테마 변경
- **스토리 추가**: 게임 세계관 확장
- **사운드트랙**: 각 지역별 배경음악 강화

---

**Next Step**: Phase 1 구현 시작 (튜토리얼 시스템)

