# 엘리베이터 자동화 패키지

Home Assistant용 엘리베이터 호출 및 음성 알람 자동화 시스템

## 📁 파일 구조

```
packages/elevator/
├── elevator_config.yaml         # ⚠️ 사용 중단 (참고용)
├── elevator_helpers.yaml        # 도우미 엔티티 (변수, 상태 머신, 타이머)
├── elevator_scripts.yaml        # TTS 스크립트
├── elevator_automations.yaml    # 5개 자동화 규칙
└── README.md                    # 이 문서
```

**주의**: `elevator_config.yaml`은 더 이상 사용되지 않습니다. 모든 설정은 이제 `elevator_helpers.yaml`의 도우미 엔티티로 관리됩니다.

## ⚙️ 설치 방법

### 1. 패키지 로드 설정

`configuration.yaml`에 다음 내용 추가:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

### 2. Home Assistant 재시작

```bash
# Home Assistant 재시작
# 설정 → 시스템 → 재시작
```

### 3. 도우미 엔티티 확인

재시작 후 다음 엔티티들이 자동 생성되었는지 확인:

**상태 관리**:
- `input_select.elevator_state` - 엘베 상태 (idle/called/moving/arrived/error)
- `input_boolean.elevator_user_called` - 사용자 호출 추적

**변수 (Entity IDs)**:
- `input_text.elevator_switch_id` - 엘베 스위치 entity_id
- `input_text.elevator_trigger_id` - 엘베 트리거 entity_id
- `input_text.direction_sensor_id` - Direction 센서 entity_id
- `input_text.floor_sensor_id` - Floor 센서 entity_id
- `input_text.primary_speaker_id` - 주 스피커 (거실)
- `input_text.secondary_speaker_id` - 보조 스피커 (주방)
- `input_text.tts_service_name` - TTS 서비스명

**변수 (메시지)**:
- `input_text.msg_call_initiated` - "엘베가 호출되었습니다"
- `input_text.msg_already_arrived` - "엘베가 이미 도착해있습니다"
- `input_text.msg_arriving_soon` - "엘베가 곧 도착합니다"
- `input_text.msg_arrived` - "엘베가 도착했습니다"
- `input_text.msg_error_timeout` - 타임아웃 에러 메시지
- `input_text.msg_error_sensor` - 센서 에러 메시지
- `input_text.msg_direction_upward` - "올라가는"
- `input_text.msg_direction_downward` - "내려가는"
- `input_text.last_announcement` - 마지막 알람 (내부용)

**변수 (숫자)**:
- `input_number.elevator_target_floor` - 목표층 (기본: 21)
- `input_number.elevator_arrival_threshold` - 도착 임계값 (기본: 5)
- `input_number.elevator_volume_day` - 주간 볼륨 (기본: 0.7)
- `input_number.elevator_volume_night` - 야간 볼륨 (기본: 0.3)

**시간 설정**:
- `input_datetime.elevator_quiet_start` - 조용한 시간 시작 (기본: 22:00)
- `input_datetime.elevator_quiet_end` - 조용한 시간 종료 (기본: 07:00)

**타이머**:
- `timer.elevator_timeout` - 타임아웃 타이머 (30초)

## 🎯 기능

### 핵심 기능
1. ✅ **엘베 호출 알람**: 스위치 누르면 즉각 음성 알람
2. ✅ **상태 추적 알람**: 실시간 엘베 위치/방향 안내
3. ✅ **근접 알람**: 목표층 가까워지면 "곧 도착합니다"
4. ✅ **도착 알람**: 21층 도착 시 알람
5. ✅ **외부 트리거 방지**: 월패드 버튼 등 외부 호출 시 알람 안 울림

### 개선 기능
1. ✅ **변수 중앙화**: 모든 설정값을 도우미 엔티티로 관리 - UI에서 실시간 수정 가능
2. ✅ **상태 머신**: 명시적 상태 관리 (idle → called → moving → arrived)
3. ✅ **시간대별 볼륨**: 주간/야간 볼륨 자동 조절 (변수로 관리)
4. ✅ **Floor 센서 트리거**: Direction + Floor 양쪽 센서 변화 감지로 정확한 상태 추적
5. ✅ **중복 알람 방지**: 동일 메시지 반복 방지 (last_announcement 추적)
6. ✅ **에러 핸들링**:
   - 30초 타임아웃 (응답 없으면 "다시 호출해주세요")
   - 센서 unavailable 즉시 감지 및 알람

## 🔧 설정 변경

**모든 설정은 UI 또는 `elevator_helpers.yaml`에서 수정**

### 방법 1: UI에서 변경 (추천)

1. Home Assistant → 설정 → 디바이스 및 서비스 → 도우미
2. 검색: "엘베" 또는 "elevator"
3. 원하는 항목 클릭하여 값 수정
4. **재시작 불필요** - 즉시 반영됨

### 방법 2: YAML 파일에서 변경

`elevator_helpers.yaml` 파일의 `initial` 값 수정 후 Home Assistant 재시작

### 주요 설정 항목

#### Entity IDs 변경
- **엘베 스위치**: `input_text.elevator_switch_id`
- **엘베 트리거**: `input_text.elevator_trigger_id`
- **Direction 센서**: `input_text.direction_sensor_id`
- **Floor 센서**: `input_text.floor_sensor_id`
- **주 스피커** (거실): `input_text.primary_speaker_id`
- **보조 스피커** (주방): `input_text.secondary_speaker_id`
- **TTS 서비스**: `input_text.tts_service_name`

#### 숫자 설정
- **목표층**: `input_number.elevator_target_floor` (기본: 21)
- **도착 임계값**: `input_number.elevator_arrival_threshold` (기본: 5층)
- **주간 볼륨**: `input_number.elevator_volume_day` (기본: 0.7)
- **야간 볼륨**: `input_number.elevator_volume_night` (기본: 0.3)

#### 메시지 커스터마이징
모든 `input_text.msg_*` 항목을 UI에서 자유롭게 수정 가능:
- `msg_call_initiated` - "엘베가 호출되었습니다"
- `msg_arriving_soon` - "엘베가 곧 도착합니다"
- `msg_arrived` - "엘베가 도착했습니다"
- `msg_direction_upward` - "올라가는"
- `msg_direction_downward` - "내려가는"
- 기타 에러 메시지 등

#### 시간 설정
- **조용한 시간 시작**: `input_datetime.elevator_quiet_start` (기본: 22:00)
- **조용한 시간 종료**: `input_datetime.elevator_quiet_end` (기본: 07:00)

## 🧪 테스트 가이드

### 1. 기본 동작 테스트

#### 테스트 1: 정상 호출 → 도착
```
1. 거실 엘베 스위치 누름
   예상: "엘베가 호출되었습니다" (livingroom_left 스피커)

2. direction 센서가 Upward로 변경
   예상: "엘베가 X층에서 올라가는 중입니다" (kitchen 스피커)

3. 21층 근처 도달 (16~20층)
   예상: "엘베가 곧 도착합니다" (kitchen 스피커)

4. direction 센서가 Arrival로 변경
   예상: "엘베가 도착했습니다" (kitchen 스피커)
```

#### 테스트 2: 이미 도착해있는 경우
```
1. direction이 이미 Arrival 상태에서 스위치 누름
   예상: "엘베가 이미 도착해있습니다" (kitchen 스피커)
   예상: 다른 알람 없음
```

#### 테스트 3: 외부 트리거 (월패드 버튼)
```
1. 스위치 누르지 않고 월패드에서 엘베 호출
2. direction/floor 센서 업데이트됨
   예상: 알람 없음 (정상 - 사용자가 스위치 누르지 않았으므로)
```

### 2. 에러 처리 테스트

#### 테스트 4: 타임아웃 (센서 응답 없음)
```
1. 스위치 누름 → "호출되었습니다" 알람
2. 30초 동안 direction이 Upward/Downward/Arrival로 변경되지 않음
   예상: "엘베 호출에 문제가 있습니다. 다시 호출해주세요" (kitchen)
   예상: 모든 상태 자동 리셋
```

#### 테스트 5: 센서 연결 끊김
```
1. 스위치 누름 → "호출되었습니다" 알람
2. direction 또는 floor 센서가 unavailable 상태가 됨
   예상: "엘베 센서 연결에 문제가 있습니다. 다시 호출해주세요" (kitchen)
   예상: 모든 상태 자동 리셋
```

### 3. 시간대별 볼륨 테스트

#### 테스트 6: 야간 볼륨 (22:00 - 07:00)
```
1. 현재 시간을 22:00 이후 또는 07:00 이전으로 설정
2. 스위치 누름
   예상: 볼륨 0.3으로 알람 재생
```

#### 테스트 7: 주간 볼륨 (07:00 - 22:00)
```
1. 현재 시간을 07:00 이후 ~ 22:00 이전으로 설정
2. 스위치 누름
   예상: 볼륨 0.7로 알람 재생
```

### 4. 임계값 경계 테스트

#### 테스트 8: 근접 알람 경계값 (임계값 5층 기준)
```
올라갈 때:
- 15층 → "15층에서 올라가는 중입니다" (일반 알람)
- 16층 → "곧 도착합니다" (근접 알람 시작)
- 20층 → "곧 도착합니다" (근접 알람)
- 21층 → "도착했습니다" (도착 알람)

내려갈 때:
- 26층 → "곧 도착합니다" (근접 알람)
- 22층 → "곧 도착합니다" (근접 알람)
- 21층 → "도착했습니다" (도착 알람)
- 27층 → "27층에서 내려가는 중입니다" (일반 알람)
```

## 🔍 문제 해결

### 문제 1: 알람이 안 울림

**확인 사항:**
1. Home Assistant가 재시작되었는지 확인
2. 모든 도우미 엔티티가 생성되었는지 확인
3. TTS 엔진이 작동하는지 확인 (개발자 도구 → 서비스 → `tts.cloud_say` 테스트)
4. 스피커가 연결되어 있는지 확인

### 문제 2: 외부 트리거에도 알람이 울림

**원인:** `input_boolean.elevator_user_called`가 제대로 작동하지 않음

**해결:**
1. 개발자 도구 → 상태 → `input_boolean.elevator_user_called` 확인
2. 스위치 누르면 `on`, 도착하면 `off`로 변경되는지 확인

### 문제 3: 타임아웃이 너무 빠르거나 늦음

**해결:** `elevator_helpers.yaml` → `timer.elevator_timeout` → `duration: "00:00:30"` 수정

### 문제 4: 볼륨이 너무 작거나 큼

**해결:** `elevator_config.yaml` → `package.volume_settings` → `volume_day`/`volume_night` 조절 (0.0 ~ 1.0)

### 문제 5: "곧 도착합니다"가 너무 빠르거나 늦게 나옴

**해결:** UI → 설정 → 도우미 → `엘베 도착 임계값` 조절 (1~10층)

## 📊 상태 머신 다이어그램

```
idle (대기)
  ↓ [스위치 누름]
called (호출됨) ← 타이머 30초 시작
  ↓ [Upward/Downward 감지]
moving (이동 중) ← 타이머 취소
  ↓ [Arrival 감지]
arrived (도착) → idle (대기)

[타임아웃 또는 unavailable]
  ↓
error (에러) → idle (대기)
```

## 🎨 Lovelace UI 카드 예시 (선택사항)

```yaml
type: entities
title: 엘리베이터 상태
entities:
  - entity: input_select.elevator_state
    name: 현재 상태
  - entity: sensor.kocom_ev_kocom_ev_0_direction
    name: 방향
  - entity: sensor.kocom_ev_kocom_ev_0_floor
    name: 현재 층
  - entity: input_number.elevator_arrival_threshold
    name: 도착 임계값
  - entity: timer.elevator_timeout
    name: 타임아웃 타이머
```

## 📝 향후 확장 계획

### 1단계 (현재)
✅ 스위치 기반 엘베 호출
✅ 음성 알람
✅ 에러 핸들링

### 2단계 (향후)
- [ ] iOS 단축어 통합
- [ ] 모바일 푸시 알림
- [ ] 호출 이력 로깅
- [ ] 평균 도착 시간 분석

### 3단계 (고급)
- [ ] 대시보드 UI
- [ ] 다국어 지원
- [ ] 학습 기반 알람 최적화

## 📞 지원

문제 발생 시:
1. Home Assistant 로그 확인 (설정 → 로그)
2. 자동화 트레이스 확인 (설정 → 자동화 → 해당 자동화 → 추적)
3. 개발자 도구로 센서 상태 확인

## 📄 라이선스

이 패키지는 Home Assistant 커뮤니티 라이선스를 따릅니다.

---

## 🔄 변경 이력

### v2.0 (최신)
- ✅ **변수 중앙화**: 모든 설정을 input_text/input_number 도우미로 관리
- ✅ **UI 실시간 수정**: 재시작 없이 대부분의 설정 변경 가능
- ✅ **Floor 센서 트리거 추가**: Direction + Floor 양쪽 센서 모니터링
- ✅ **중복 알람 방지**: 동일 메시지 반복 제거
- ✅ **TTS 서비스 동적 선택**: tts.cloud_say, tts.google_ai_tts_say 등 자유 변경
- ✅ **볼륨 변수화**: 주간/야간 볼륨을 UI에서 조절
- ⚠️ **Breaking Change**: elevator_config.yaml 사용 중단

### v1.0
- 기본 엘베 호출 및 음성 알람
- 상태 머신, 에러 핸들링, 시간대별 볼륨
