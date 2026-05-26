# QA 분석 결과 - 버그 및 성능 이슈

**분석일**: 2026-05-26  
**상태**: 🔴 심각 버그 1개, 🟡 중요 이슈 3개, 🟢 개선 권장 5개

---

## 🔴 심각 버그 (Critical)

### 1. 관통탄 Hit Cooldown 불완전 해결
**위치**: Line 10270-10276  
**문제**: Hit cooldown이 10프레임이지만, 충돌 검사 자체는 여전히 매 프레임 실행
```javascript
for(let j=bt.enemies.length-1;j>=0;j--){  // 매 프레임 전체 루프
  if(d2 < collideR*collideR){
    if(cooldown > 0){ continue; }  // 스킵만 함
  }
}
// = 여전히 충돌 검사 O(n) 실행!
```

**영향**: 관통탄이 많은 적을 맞을 때도 충돌 검사 루프가 계속 실행  
**심각도**: ⭐⭐⭐ (10-20ms 낭비)  
**해결책**: 관통탄 별도 루프 또는 충돌 검사 빈도 조절

---

## 🟡 중요 이슈 (Major)

### 2. 파티클 생성 제한이 동적이 아님
**위치**: Line 10724-10735  
**문제**: 
```javascript
const particleCount = (bt.particles||[]).length + (bt.impactRings||[]).length;
const shouldSpawnParticles = particleCount < 400; // 정적 체크
```
파티클이 399개면 추가 생성, 1개면 스킵 → 프로비저닝 불균형

**영향**: 간헐적 파티클 폭증  
**심각도**: ⭐⭐⭐

---

### 3. 음향 함수 호출 과다
**위치**: Line 10743, 곳곳  
**문제**: 매 damageEnemy() 호출마다 Audio.sfx.hit() 또는 Audio.sfx.hitCrit() 호출
```javascript
// Phase 4 이후:
if(isMultiHit){
  if(Math.random() < 0.1) { /* 음향 */ }  // 여전히 호출됨
}
```

**영향**: 
- 다단히트 시 음향 채널 점유
- 실제로 10% 확률이지만 여전히 자주 호출

**심각도**: ⭐⭐

---

### 4. 넉백(Knockback) 계산 매 프레임
**위치**: Line 10714-10717  
**문제**:
```javascript
if(!e.isBoss){
  const kbAngle = Math.atan2(e.y-gs.player.y, e.x-gs.player.x);  // 불필요한 계산
  // ...
}
```

**영향**: 삼각함수 계산이 damageEnemy() 호출마다 실행  
**심각도**: ⭐⭐ (O(1)이지만 빈번함)

---

## 🟢 개선 권장 (Minor/Enhancement)

### 5. spawnDN() 호출 위치 최적화
**위치**: Line 10737-10741  
**현황**: 다단히트 시 8번마다만 표시 (good)  
**권장**: 시뮬레이션 숫자 자체 줄이기 (현재 50-100개/프레임)

### 6. 히트스톱(Hitstop) 누적 문제
**위치**: Line 10710  
**현황**: `bt.hitstop = Math.max(bt.hitstop||0, isCrit ? 4 : 2);`  
**문제**: 여러 적에게 동시에 맞으면 hitstop이 누적될 수 있음  
**권장**: 클램핑 추가

### 7. 빈네트 플래시 누적
**위치**: Line 10747  
**현황**: `bt.vignetteFlash = Math.max(bt.vignetteFlash||0, 0.40);`  
**권장**: 음향처럼 cooldown 추가

### 8. AOE 폭발 내부 forEach 최적화
**위치**: Line 10266  
**현황**: 매 폭발마다 전체 enemies 순회
```javascript
bt.enemies.forEach(e=>{ ... });
```
**권장**: 거리 기반 필터링 후 순회

### 9. Lightning targeting 정렬 비용
**위치**: Line 10480  
**현황**: 매 번개 발동마다 sort() 실행
```javascript
const targets=[...bt.enemies].sort(...).slice(0, s.lightning+1);
```
**권장**: 거리 계산 캐싱 또는 지연 정렬

---

## 📊 버그 우선순위 기반 해결 순서

### **Phase 5 (긴급 - 이번 버전)**
1. ✅ ~~관통탄 cooldown~~ (10프레임 강화 완료)
2. 🔴 **관통탄 충돌 검사 별도 처리** (남은 버벅거림의 원인)
3. 🟡 파티클 동적 제한 시스템

### **Phase 6 (다음 버전)**
4. 음향 채널 제한 (동시 최대 5개)
5. 넉백 계산 캐싱
6. 번개 정렬 최적화

---

## 🔧 즉시 적용 가능한 수정 (Critical Fix)

### 관통탄 충돌 검사 최적화
```javascript
// 현재 (문제)
for(let j=bt.enemies.length-1;j>=0;j--){
  if(d2 < collideR*collideR){ damageEnemy(); }
}

// 해결책: 충돌 검사를 2프레임마다만 수행
if(!b._collisionCheckFrame) b._collisionCheckFrame = 0;
if((bt.frameCount || 0) % 2 === b._collisionCheckFrame % 2){
  for(let j=bt.enemies.length-1;j>=0;j--){
    if(d2 < collideR*collideR){ damageEnemy(); }
  }
}
```

또는:
```javascript
// 대안: 관통탄만 별도 충돌 배열 관리
if((s.piercing || b.windBlade) && b._pierceHitMap){
  // 빠른 검사만 수행 (이미 맞은 적 제외)
  for(const uid in b._pierceHitMap){
    if(b._pierceHitMap[uid] > 0) continue;
    // 검사...
  }
}
```

---

## 📝 테스트 검증 필요

```
[ ] 관통탄 + 50마리 적 → FPS 유지 확인
[ ] 음향 동시 재생 수 → 콘솔에서 확인
[ ] 파티클 누적 → 400 제한이 작동하는지
[ ] 넉백 모션 → 부자연스러운 움직임 없는지
[ ] 비네트 플래시 → 과도한 밝기 변화 없는지
```

---

## 🎯 권장 수정 우선순위

1. **✅ 완료**: Math.hypot() → 제곱 거리 (Phase 3)
2. **✅ 완료**: damageEnemy() 다단히트 경량화 (Phase 4)
3. **⏳ 필요**: 관통탄 충돌 검사 재구조화 (Phase 5)
4. **⏳ 필요**: 파티클/음향 동적 제한 (Phase 5)

---

**다음 단계**: Phase 5 - 관통탄 충돌 검사 최적화 및 파티클 시스템 개선

