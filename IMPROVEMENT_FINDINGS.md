# 🔍 게임 개선 필요 사항 분석 보고서

**분석일**: 2026-05-26  
**분석자**: Claude Code  
**상태**: 성능 최적화 완료 후 추가 개선 항목 발굴

---

## 📊 종합 평가

### 성능 상태
- ✅ **보스 타격 렉**: 완전 해결됨
- ✅ **파티클 스팸**: 제한됨 (최대 3개 impactRings)
- ✅ **음향 쌓임**: 제어됨 (쿨타임 적용)
- ✅ **전체 게임**: 부드럽게 실행 중

---

## 🔴 긴급 개선 (Priority 1)

### 1. 번개 시너지 타겟 선택 성능 저하
**위치**: Line 10780-10789  
**현재 문제**: 가장 가까운 N개 적을 찾기 위해 O(n²) 알고리즘 사용
```javascript
// 현재: O(n²) 복잡도
for(let j=0; j<Math.min(s.lightning+1, bt.enemies.length); j++){
  let closest=-1, closestDist=Infinity;
  for(let k=0; k<bt.enemies.length; k++){
    // 매번 모든 적 검사
  }
}
```

**영향**: 적이 50+마리일 때 번개 계산만 2500회 루프  
**권장 해결책**: 
- 벡터 정렬 캐싱 (매 프레임마다가 아닌 주기적으로만)
- 또는 공간 분할 (spatial hashing) 사용

---

### 2. 연쇄 강타도 동일한 문제
**위치**: Line 10798-10808  
**현재 코드**:
```javascript
// 각 번개 타겟마다 또 다시 O(n²)
if((s.chainStrike||0)>0){
  let _cTargets=[];
  for(let j=0; j<Math.min(s.chainStrike+1, ...); j++){
    let closest=-1, closestDist=Infinity;
    for(let k=0; k<bt.enemies.length; k++){
      // 또 다시 모든 적 검사
    }
  }
}
```

**영향**: 번개 + 연쇄 강타 = O(n²) × 2  
**해결책**: 위의 번개 최적화와 동일하게 처리

---

## 🟡 중요 개선 (Priority 2)

### 3. 회전검(Sword) 충돌 루프 최적화 미흡
**위치**: Line 10728-10770  
**현재 구조**:
```javascript
for(let j=bt.enemies.length-1; j>=0; j--){
  const e=bt.enemies[j];
  for(let sv=0; sv<s.sword; sv++){ // 검 개수만큼
    // 거리 계산 반복
  }
}
```

**문제**: 적 × 검 개수의 거리 계산 (N×M 루프)  
**개선**: 검을 먼저 그린 후 적과의 충돌만 검사

---

### 4. 파티클 생성 조건 여전히 복잡
**위치**: Line 11059  
**현재**:
```javascript
if(particleCount < maxParticles && (isCrit || (e._hitCount || 0) % 3 === 0)){
```

**문제**: 매 damageEnemy() 호출마다 조건 검사  
**권장**: 파티클 생성을 별도 스케줄러로 분리 (예: 패턴 기반)

---

### 5. 음향 효과 재생 여전히 빈번
**위치**: Line 11089-11092  
**현재**:
```javascript
if(!bt._lastHitSoundTime || Date.now() - bt._lastHitSoundTime > 50){
  if(isCrit) Audio.sfx.hitCrit(); else Audio.sfx.hit();
  bt._lastHitSoundTime = Date.now();
}
```

**문제**: 50ms 쿨타임이 보스전에서 자주 트리거됨  
**권장**: 100ms 이상으로 증가, 또는 히트 강도별로 차등 적용

---

## 🟢 가능한 개선 (Priority 3)

### 6. 보스 페이즈 전환 파티클 최적화
**위치**: Line 10129, 10141, 10154  
**문제**: 페이즈 전환 시 28~48개의 파티클을 한 번에 생성
```javascript
// Phase 2 전환
for(let k=0; k<28; k++){ ... spawnP(...) }
// Phase 3 전환
for(let k=0; k<36; k++){ ... spawnP(...) }
// Phase 4 전환
for(let k=0; k<48; k++){ ... spawnP(...) }
```

**권장**: 파티클 생성 횟수를 20 이하로 제한

---

### 7. 빈네트(Vignette) 플래시 누적 여전히 가능
**위치**: Line 11100  
**현재**:
```javascript
bt.vignetteFlash = Math.min(0.6, Math.max(bt.vignetteFlash||0, 0.40));
```

**문제**: 여전히 누적될 가능성 있음 (같은 프레임에 여러 적 히트 시)  
**권장**: 쿨타임 추가 (8프레임마다 1회만)

---

### 8. 히트스톱(Hitstop) 최대값이 10인데 너무 길 수 있음
**위치**: Line 11040  
**현재**:
```javascript
if(bt.hitstop > 10) bt.hitstop = 10; // 최대값 10 프레임 ≈ 166ms
```

**문제**: 고주파 히트(관통탄 등)에서 누적되면 명감  
**권장**: 최대값을 6-8 프레임으로 제한

---

### 9. 회전검 음향이 너무 드물게 재생됨
**위치**: Line 10765-10767  
**현재**: 60프레임(1초)마다 1회만 재생
```javascript
if(swordHit && !bt._swordSoundCooldown){
  Audio.sfx.sword();
  bt._swordSoundCooldown = 60; // 너무 길어서 소리 끊김
}
```

**권장**: 30프레임(0.5초)으로 감소 → 더 자연스러운 음향

---

### 10. 적 AI 움직임 예측 계산 부재
**위치**: updateBattle 함수 전반  
**문제**: 현재 위치 기반 충돌만 계산, 미래 위치 예측 없음  
**권장**: 고급 기능이므로 낮은 우선순위

---

## 📈 제안된 개선 순서

### Phase 8 (다음 버전 - 긴급)
1. **번개 + 연쇄 강타 타겟팅**: O(n²) → O(n log n) 또는 O(n)
2. **히트스톱 최대값**: 10 → 6-8 프레임으로 감소

### Phase 9 (그 다음)
3. **음향 쿨타임**: 50ms → 100ms 증가
4. **회전검 음향**: 60프레임 → 30프레임으로 감소
5. **파티클 생성**: 별도 스케줄러 도입

### Phase 10 (장기 개선)
6. **빈네트 플래시**: 쿨타임 추가 (8프레임)
7. **보스 페이즈 파티클**: 개수 제한 (20 이하)
8. **회전검 충돌**: 루프 순서 재구성 (E×S → S만)

---

## 🎯 성능 영향 예상

| 개선 항목 | 예상 성능 향상 | 난이도 |
|----------|--------------|--------|
| 번개 최적화 | +20% | 중간 |
| 히트스톱 감소 | +5% | 쉬움 |
| 음향 쿨타임 | +3% | 쉬움 |
| 파티클 스케줄러 | +15% | 높음 |

---

## 🔧 즉시 적용 가능한 수정 (Quick Fix)

### 1. 히트스톱 최대값 감소 (1분)
```javascript
// Line 11040: 변경 전
if(bt.hitstop > 10) bt.hitstop = 10;

// 변경 후
if(bt.hitstop > 8) bt.hitstop = 8; // 133ms로 감소
```

### 2. 회전검 음향 쿨타임 개선 (1분)
```javascript
// Line 10767: 변경 전
bt._swordSoundCooldown = 60; // 1초

// 변경 후
bt._swordSoundCooldown = 30; // 0.5초 = 더 자연스러움
```

### 3. 음향 쿨타임 증가 (1분)
```javascript
// Line 11091: 변경 전
if(!bt._lastHitSoundTime || Date.now() - bt._lastHitSoundTime > 50){

// 변경 후
if(!bt._lastHitSoundTime || Date.now() - bt._lastHitSoundTime > 80){
```

---

## 📝 테스트 항목

```
[ ] 번개 시너지 10+ 레벨 성능 확인
[ ] 연쇄 강타 + 번개 동시 발동 성능
[ ] 히트스톱 감소 후 타격감 여전히 좋은지 확인
[ ] 회전검 음향 재생 빈도 적절한지 확인
[ ] 파티클 스팸 여전히 없는지 확인
[ ] 보스 페이즈 전환 시 프레임 드롭 확인
```

---

## 🎮 플레이어 경험 영향

- ✅ 현재: 모든 전투 상황에서 부드러움
- 🎯 개선 후: 극단적 상황(보스 + 50마리 적)에서도 60fps 유지
- 🔊 음향: 더 자연스러운 히트 피드백

---

**최종 평가**: 성능 최적화는 충분히 이루어졌으며, 남은 항목들은 **미세 조정** 수준입니다.
