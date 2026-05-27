# 🚨 긴급: 게임 로비 버그 조사 및 해결 절차

**보고된 증상**: 게임 로비 화면이 검은색으로만 표시되고, 캐릭터 카드가 보이지 않음

## 즉시 시도할 조치

### 1️⃣ 브라우저 콘솔 확인 (F12 → Console 탭)
- 페이지 열기 → F12 또는 우클릭 > 검사
- **Console 탭** 확인
- 빨강 에러 메시지 있으면 스크린샷/복사

### 2️⃣ 캐시 초기화
```
게임 페이지 → F12 → Settings 탭 → Storage → Clear site data 클릭
또는: Ctrl+Shift+Delete → All time → Clear data
```

### 3️⃣ 페이지 새로고침
```
Ctrl+Shift+R (완전 새로고침)
또는: Ctrl+F5
```

### 4️⃣ 게임 파일 재확인
- survivor.html이 손상되지 않았는지 파일 크기 확인
- 현재 파일 크기: 약 16,870줄

---

## 🔍 잠재적 원인 분석

마지막 변경사항 (2026-05-26):
- **라인 9547**: frameCount 초기화 추가
- **라인 10271**: shouldCheckCollisions 조건 추가
- → 이들은 **battleScreen**에서만 실행되므로, **로비 렌더링에는 영향 없음**

가능한 원인:
1. **JavaScript 런타임 에러** (parseError, null reference 등)
   - initLobby() 함수 실행 중 에러
   - renderLobby() 함수 내부 HTML 렌더링 실패

2. **CSS 로드 문제**
   - @import font-awesome 또는 google fonts 로드 실패
   - 스타일시트가 모두 흰색 계열로 표시

3. **DOM 구조 손상**
   - startScreen 요소가 없거나 손상됨
   - 로비-specific CSS가 적용되지 않음

---

## 수정 계획

### Phase A: 콘솔 에러 확인 후
1. 에러 메시지를 알려주면 원인 파악
2. 해당 함수 또는 변수 수정
3. 재테스트

### Phase B: 콘솔 에러 없으면
1. 로비 관련 CSS 재확인
2. renderLobby() 함수 로직 검증
3. HTML 문자열 구성 검증

### Phase C: 최후의 수단
```javascript
// 브라우저 콘솔에서 직접 실행:
document.getElementById('startScreen').innerHTML  // 확인
gs.screen  // 확인
meta  // 데이터 확인
```

---

## ✅ 체크리스트

- [ ] F12 콘솔 확인 → 에러 메시지 정리
- [ ] 캐시 초기화
- [ ] 페이지 새로고침
- [ ] 정상 작동 확인

---

**다음 단계**: 콘솔 에러 메시지를 알려주면 즉시 수정하겠습니다.

