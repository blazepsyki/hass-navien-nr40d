# Home Assistant용 나비엔 NR-40D

[![HACS 사용자 지정 통합](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)
[![원본 저장소 릴리스](https://img.shields.io/github/v/release/ibuben/hass-navien-nr40d?include_prereleases)](https://github.com/ibuben/hass-navien-nr40d/releases)

나비엔 **NR-40D 실내온도조절기**를 Home Assistant에 연결하는 로컬 통합입니다. Deluxe ONE 및 호환 보일러에서 Home Assistant가 조절기의 연결을 직접 받아 Smart TOK이나 Sphere 클라우드 없이 보일러를 제어합니다.

Lovelace 카드 `custom:navien-nr40d-card`에서 실내 온도, 설정 온도, 난방 상태, 외출 모드, 난방 모드, 난방수 온도와 온수 온도를 확인할 수 있습니다.

## 대시보드 카드

카드의 JavaScript 모듈이 로드되면 **카드 추가** 목록에 표시됩니다. 통합을 업데이트한 뒤에는 Home Assistant를 재시작하고 브라우저에서 `Ctrl+F5`로 새로고침하세요. 목록에 카드가 없으면 다음 YAML로 직접 추가할 수 있습니다. `YOUR_DEVICE_ID`를 NR-40D 기기 ID로 바꾸세요.

```yaml
type: custom:navien-nr40d-card
device: YOUR_DEVICE_ID
```

통합은 JavaScript 모듈을 자동 등록합니다. 필요한 경우 **설정 → 대시보드 → 리소스 → 추가**에서 `/navien_nr40d/navien-nr40d-card.js`를 **JavaScript 모듈**로 등록할 수 있습니다.

온도조절기 엔티티는 목록에 **보일러**라는 이름으로 표시됩니다. 기기를 선택하거나 해당 기기의 `climate` 엔티티 ID를 지정할 수 있습니다.

```yaml
type: custom:navien-nr40d-card
device: YOUR_DEVICE_ID
entity: climate.navien_nr40d
```

카드는 기기에 연결된 난방수, 온수, 난방 모드, 연결 상태, 오류 엔티티를 자동으로 찾습니다. 필요한 경우 실제 엔티티 ID를 직접 지정하세요.

```yaml
type: custom:navien-nr40d-card
entity: climate.navien_nr40d
name: 보일러
water_entity: number.navien_nr40d_water_temp
dhw_entity: number.navien_nr40d_dhw_temp
mode_entity: select.navien_nr40d_heating_mode
away_entity: switch.navien_nr40d_away
```

카드 문구는 Home Assistant의 표시 언어를 따릅니다. YAML에 `name`을 지정하면 그 값이 카드 제목으로 표시됩니다.

- [한국어 카드 YAML 예제](lovelace/navien-nr40d-card.yaml)
- [영어 카드 YAML 예제](lovelace/navien-nr40d-card.en.yaml)

## Home Assistant에 추가되는 엔티티

| 엔티티 | 기능 |
| --- | --- |
| `climate` | 난방 켜기·끄기, 실내 온도 확인 및 설정, 외출 프리셋 |
| `select` 난방 모드 | 실내 온도 기준인 **실내** 모드와 난방수 온도 기준인 **온돌** 모드 전환 |
| `number` 난방수 온도 | 40–80 °C 설정 |
| `number` 온수 온도 | 30–60 °C 설정 |
| `switch` 외출 모드 | 외출 모드 전환 및 자동화 |
| `sensor` 온도, 운전 모드, 오류 | 상태 기록과 오류 코드 설명 표시 |
| `binary_sensor` 조절기 연결 | 조절기가 통합에 연결되어 있는지 표시 |

조절기는 온도 설정값을 1 °C 단위로 받습니다.

## 요구 사항

- Home Assistant 2024.2 이상
- Home Assistant와 같은 로컬 네트워크에 연결된 NR-40D 조절기
- 조절기의 클라우드 연결을 Home Assistant 호스트로 보내는 공유기 설정

## 공유기 설정

조절기는 나비엔 클라우드로 연결을 시작합니다. 공유기의 NAT 또는 로컬 DNS로 연결 목적지를 Home Assistant 호스트로 바꾸면 조절기가 기존 프로토콜과 목적지 포트로 Home Assistant에 연결합니다.

### 조절기의 연결 대상

- 호스트는 `rust2.naviensmartcontrol.com`, `rust.naviensmartcontrol.com`입니다. 지역에 따라 같은 `naviensmartcontrol.com` 도메인의 `krst2…`, `ukst…` 같은 이름을 사용할 수 있습니다.
- TCP 포트는 `7000`과 `21000–21010`입니다. 주로 `7000`, `21001`, `21002`, `21003`을 사용합니다. 조절기는 같은 서비스에 여러 포트로 접속을 시도하므로 설정에 빠진 포트로 연결하면 통합에 연결되지 않을 수 있습니다.

### 연결 목적지 변경 방법

1. **DNAT**
   조절기에서 위 클라우드 주소와 포트로 향하는 TCP 트래픽의 목적지를 Home Assistant의 IP와 같은 포트로 바꿉니다. 조절기가 호스트 이름을 직접 조회해 외부 IP로 연결하는 경우에도 적용할 수 있습니다.
2. **로컬 DNS**
   공유기의 정적 DNS 레코드, `hosts`, AdGuard 또는 Pi-hole을 이용해 클라우드 호스트 이름이 Home Assistant의 IP를 가리키도록 설정합니다. 조절기가 해당 DNS 서버를 사용해야 적용됩니다.
3. **DNAT와 로컬 DNS 함께 사용**
   새 DNS 조회는 Home Assistant를 가리키며, 이전 외부 IP를 캐시한 조절기의 트래픽은 DNAT로 처리할 수 있습니다.

공유기 펌웨어에 따라 메뉴 이름은 Destination NAT, Port Forward, Virtual Server, Address Mapping 등으로 표시됩니다. 조절기에서 나가는 연결의 목적지만 바꾸면 되므로 외부에서 들어오는 연결을 허용할 필요는 없습니다.

조절기의 트래픽이 클라우드나 다른 호스트로 향하는 동안에는 연결 센서가 꺼짐 상태로 표시됩니다.

## 설치

### HACS

1. **HACS → 통합 → 사용자 지정 저장소**를 엽니다.
2. 원본 저장소 `https://github.com/ibuben/hass-navien-nr40d`를 추가하고 유형으로 **Integration**을 선택합니다.
3. **Navien NR-40D**를 설치하고 Home Assistant를 재시작합니다.
4. **설정 → 기기 및 서비스 → 통합 추가**에서 **Navien NR-40D**를 선택합니다.

이 포크의 한국어 번역을 적용하려면 수동 설치 절차에 따라 이 저장소의 파일을 복사하세요.

### 수동 설치

1. `custom_components/navien_nr40d` 폴더를 `/config/custom_components/navien_nr40d`에 복사합니다.
2. Home Assistant를 재시작합니다.
3. Home Assistant 화면에서 **Navien NR-40D** 통합을 추가합니다.

기본 수신 주소는 `0.0.0.0`이며 기본 TCP 포트는 `7000,21001,21002,21003`입니다. 필요하면 `21000–21010` 범위에서 나머지 포트를 추가하세요.

### Docker / HAOS

Home Assistant 컨테이너가 호스트의 TCP 연결을 받을 수 있어야 합니다. `host` 네트워크 모드를 사용하거나 `7000`과 `21000–21010` 포트를 컨테이너에 매핑하세요.

## 보일러와 조절기 조합의 제한 사항

조절기가 다음 기능을 지원하지 않는다고 보고하면 해당 명령은 무시됩니다.

- 여름철 온수 전용 모드
- 온수 빠른 가열
- 0.5 °C 단위 설정(예를 들어 25.5 °C)

보일러 서비스 메뉴(`S.THE`, DIP, 실외 센서 `G.oN`)는 보일러 본체의 조작부에서만 사용할 수 있습니다. 이 버전에서는 조절기의 주간 일정을 저장할 수 없으므로 Home Assistant 자동화로 일정을 설정하세요.

## 문제 해결

- **로그**는 **설정 → 시스템 → 로그**에서 `navien_nr40d`로 필터링하세요.
- **진단 정보**는 기기 페이지의 **진단 정보 다운로드**에서 받을 수 있습니다.

원본 프로젝트 후원 주소(USDT ERC20)는 `0x0e807678C52E1BCAB291233266c7383da7a5f993`입니다.
