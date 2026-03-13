# Kiwi v2.1.0 인수인계 문서

> 다른 PC/세션에서 이어서 작업할 때 이 문서를 먼저 읽을 것.

---

## 현재 상태

- **버전**: v2.1.0
- **커밋**: `8dd612c` (master, pushed)
- **완료**: Phase 0~12 (코드 구현 완료)
- **미완료**: Phase 13 (테스트), Phase 14 (빌드 & 배포)

---

## 무엇을 했나

v1.0.3 (commit `f38cc64`, 498줄)을 베이스로 완전 재빌드.
이전 v2.0.1~v2.0.5 시도는 모두 폐기 — 동적 폴링에서 throw 해서 오히려 느려졌음.

동료의 Ezbaro_Downloader.exe (Python)에서 핵심 패턴 차용:
- **stamp 기반 폴링** (120ms 간격, 연속 2회 동일 시 settled)
- **타임아웃 시 절대 throw 안 함** ← 이전 실패의 핵심 원인

### 구현된 기능 (Phase 1~12)

| Phase | 기능 | 파일 |
|-------|------|------|
| 1 | stamp 폴링 (waitSearchSettled, waitReasonInputReady) | server/index.js |
| 2 | WSL 경로 변환 (/mnt/c/ → C:\) + 한글 파일명 수정 | server/index.js |
| 3 | 세션 Keep-Alive (4분 간격) | server/index.js |
| 4 | 로그인/OTP 감지 | server/index.js |
| 5 | 자동 네비게이션 + ezbaro 우선 매칭 | server/index.js |
| 6 | 3라운드 자동 재시도 (runBatchWithRetry) | server/index.js |
| 7 | 일시정지/재개 + OTP 자동 일시정지 | server/index.js |
| 8 | 병합 (merged_YYYYMMDD.xlsx) + .crdownload 정리 | server/index.js |
| 9 | CDP 세션 재생성, 브라우저 끊김 감지 | server/index.js |
| 10 | /api/state 응답 확장 | server/index.js |
| 11 | web/index.html 전면 재빌드 | web/index.html |
| 12 | package.json v2.1.0 | package.json |

### 파일 현황

| 파일 | 줄 수 | 비고 |
|------|-------|------|
| server/index.js | 974줄 | 구문 검사 통과 |
| web/index.html | 612줄 | v2.1.0 UI |
| package.json | 47줄 | version 2.1.0 |

### API 엔드포인트

| Method | Path | 용도 |
|--------|------|------|
| GET | /api/state | 전체 상태 조회 |
| POST | /api/set-download-dir | 저장 폴더 설정 |
| POST | /api/upload | 엑셀 업로드 |
| POST | /api/browser/launch | 브라우저 연결 |
| GET | /api/browser/status | 브라우저 상태 |
| POST | /api/start | 배치 시작 |
| POST | /api/stop | 배치 중지 |
| POST | /api/navigate | 상시점검 화면 자동 이동 |
| POST | /api/pause | 일시정지/재개 토글 |
| POST | /api/retry-failed | 실패분 재시도 |
| POST | /api/dismiss-alert | 경고 닫기 |

---

## 다음에 할 일

### Phase 13: 테스트

```bash
# 1. 서버 시작
cd /home/myeol/tessera_addon
node -e "require('./server/index').start(14040)"

# 2. Chrome 디버그 모드 (Windows에서)
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9446 --user-data-dir=C:\kiwi-chrome-data https://www.gaia.go.kr/main.do

# 3. 브라우저에서 http://localhost:14040 접속
# 4. 브라우저 열기 → 로그인 → 자동 이동 → 엑셀 업로드 → 시작
```

테스트 항목:
- [ ] 서버 기동 (port 14040)
- [ ] 브라우저 연결 (CDP 9446)
- [ ] 샘플 업로드
- [ ] 다운로드 폴더 설정
- [ ] 5건 실다운로드 (건당 시간 측정)
- [ ] rename 정상
- [ ] 병합 파일 생성 확인

### Phase 14: 빌드 & 배포

```bash
# Windows에서 실행
cd C:\projects\tessera_addon
npx electron-builder --win
```

- [ ] exe 동작 확인
- [ ] Git tag v2.1.0
- [ ] GitHub 릴리즈

---

## 참고 문서

- `docs/PLAN.md` — 전체 구현 계획 (Phase 0~14)
- `docs/CONTEXT.md` — 컨텍스트 추적 (v1.0.3 구조, Ezbaro 분석, 실패 교훈)
- `docs/CHECKLIST.md` — 세부 체크리스트 (Phase별 항목)

---

## 절대 주의사항

1. **타임아웃 시 throw 절대 금지** — 이게 v2.0 실패의 핵심 원인
2. **stamp 폴링은 120ms 간격, 연속 2회 동일** — Ezbaro 방식 그대로
3. **v1.0.3 Nexacro API 방식 유지** — DOM 클릭(page.mouse.click) 사용 금지
4. **CDP GUID 기반 다운로드 추적 유지** — 파일 크기 폴링 사용 금지
5. **WSL에서 CDP downloadPath는 Windows 경로** — /mnt/c/ → C:\ 변환 필수
