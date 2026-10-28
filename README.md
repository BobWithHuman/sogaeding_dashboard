# With Human Dashboard

`sogaeding` Control 엔진의 조회 전용 API를 사용하는 별도 프런트엔드입니다. Grafana의 정보 밀도와 운영 콘솔 패턴을 참고하되, With Human의 ALLOW / DENY / ASK와 실행 관측 모델에 맞게 구성했습니다.

## 실행 경계

- 브라우저는 same-origin `/dashboard/v1/*`만 호출합니다.
- operator bearer, TLS CA, ASK nonce, 사용자 원문과 파일 본문은 브라우저로 전달하지 않습니다.
- 화면은 조회 전용입니다. ASK 응답, 승인, 재실행, 정책 변경 기능을 추가하지 않습니다.
- 외부 CDN, 폰트, 분석 스크립트를 사용하지 않아 현재 CSP와 함께 동작합니다.

## 파일

- `index.html`: 화면 구조와 접근성 landmark
- `styles.css`: Grafana풍 다크 운영 UI와 반응형 레이아웃
- `app.js`: 조회, 필터, pagination, 자동 새로고침, 상세 dialog

빌드 과정은 없습니다. `scripts/dashboard_local.ps1` 또는 Compose의 `dashboard` profile이 이 디렉터리를 읽기 전용으로 마운트하고 localhost BFF가 제공합니다.

운영 자격이 없는 개발 머신에서는 dev 이미지로 실제 조회 projection과 합성 판정을 사용하는 preview를 실행할 수 있습니다.

```bash
docker run --rm -p 127.0.0.1:8150:8150 \
  -e PYTHONPATH=/workspace/src -e CONTRACT_DIR=/workspace/docs/mvp/contracts \
  -e WITH_HUMAN_DASHBOARD_ASSETS=/workspace/dashboard \
  -e WITH_HUMAN_CONTAINER_BOUNDARY=loopback-published \
  -v "$PWD:/workspace:ro" -w /workspace sogaeding-team-dev:local \
  python tools/dashboard_preview.py --port 8150
```

이 preview는 Jev에 연결하거나 Gateway/Upstream 실행 경로를 열지 않습니다.

## 조회 화면

1. Overview: 판정량, 시계열, 서비스 상태, ASK 및 Jev 장부
2. Tool calls: 필터된 실행 후보와 상세 검사·timeline
3. ASK lifecycle: 최초 ASK 후보만 조회
4. Security audit: 보안 경고와 감사 이벤트

실패 시 기존 숫자를 0이나 ONLINE으로 바꾸지 않고 stale 상태와 마지막 성공 시각을 표시합니다.
