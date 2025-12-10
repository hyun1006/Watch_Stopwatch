# ⏱️ FPGA Smart Watch & Stopwatch

\<div align="center"\>

\<img src="[https://img.shields.io/badge/Language-Verilog-blue?style=for-the-badge\&logo=verilog](https://img.shields.io/badge/Language-Verilog-blue?style=for-the-badge&logo=verilog)" /\>
\<img src="[https://img.shields.io/badge/Tool-Vivado-red?style=for-the-badge\&logo=xilinx](https://img.shields.io/badge/Tool-Vivado-red?style=for-the-badge&logo=xilinx)" /\>
\<img src="[https://img.shields.io/badge/Board-Basys3-green?style=for-the-badge\&logo=fpga](https://img.shields.io/badge/Board-Basys3-green?style=for-the-badge&logo=fpga)" /\>

Basys 3 보드의 물리 버튼을 활용한 정밀 디지털 시계 및 스톱워치 설계

\</div\>

-----

## 📖 프로젝트 개요 (Project Overview)

이 프로젝트는 **Basys 3 FPGA 보드**를 활용하여 디지털 시계(Watch)와 스톱워치(Stopwatch) 기능을 구현한 결과물입니다.
외부 통신 없이 \*\*FPGA 내부 로직과 물리적 입출력 장치(Switch, Button, 7-Segment)\*\*만으로 동작하도록 설계되었습니다. 특히 기계적 스위치의 노이즈를 제거하는 디바운싱(Debouncing) 기술과 정확한 시간 계수를 위한 클럭 분주(Clock Division) 설계에 중점을 두었습니다.

-----

## 🚀 주요 기능 (Key Features)

### 1️⃣ 듀얼 모드 시간 관리 (Dual Mode Timekeeping)

  * **🕒 Digital Watch:** 시(Hour), 분(Min), 초(Sec) 단위의 실시간 시계 기능 (24시간제).
  * **⏱️ Stopwatch:** 1/100초(10ms) 단위의 정밀 타이머 기능.
  * **🔄 모드 전환:** 보드의 스위치(`sw1`) 조작을 통해 시계 모드와 스톱워치 모드 간 화면 전환 가능.

### 2️⃣ 하드웨어 제어 시스템 (Hardware Control System)

  * **시간 설정:** 시계 모드에서 버튼을 눌러 시/분/초를 개별적으로 조정 가능.
  * **스톱워치 제어:**
      * **Start/Stop:** 버튼을 눌러 카운팅 시작 및 일시 정지.
      * **Clear:** 정지 상태에서 시간을 00:00.00으로 초기화.
  * **입력 안정화:** 모든 버튼 입력에 Debounce 모듈을 적용하여 오작동 방지.

### 3️⃣ 📺 디스플레이 (FND Controller)

  * **Multiplexing:** 4자리 7-Segment Display를 고속으로 시분할 구동하여 잔상 효과를 이용한 숫자 표시.
  * **Visual Feedback:**
      * 시계 모드: `시:분` 표시 (버튼 입력 시 `분:초` 등으로 변경 가능하도록 확장성 고려).
      * 스톱워치 모드: `초.밀리초` 표시 및 동작 중 Dot(.) 점멸 기능.

-----

## 🛠️ 하드웨어 아키텍처 (H/W Architecture)

이 프로젝트는 크게 타이밍 생성부, 제어부, 그리고 데이터 처리부로 나뉩니다.

| 모듈명 (Module) | 역할 (Role) | 주요 상세 내용 |
| :--- | :--- | :--- |
| **top.v** | 최상위 모듈 | 시스템 클럭, 버튼 입력, FND 출력을 연결하는 Top Wrapper |
| **watch\_dp.v** | 시계 데이터패스 | 1Hz(1초) 틱을 생성하여 시/분/초 카운터 동작 및 오버플로우 처리 |
| **stopwatch\_dp.v** | 스톱워치 데이터패스 | 100Hz(10ms) 틱 기반 카운팅, 랩타임(임시 저장) 기능의 기초 마련 |
| **stopwatch\_cu.v** | 스톱워치 제어부 | FSM(Finite State Machine)을 이용한 `IDLE`, `RUN`, `CLEAR` 상태 전이 관리 |
| **combiner.v** | 데이터 MUX | `sw1` 신호에 따라 Watch와 Stopwatch의 출력 데이터 중 하나를 선택하여 FND로 전달 |
| **fnd\_controller.v** | 디스플레이 컨트롤러 | 입력된 BCD 데이터를 7-Segment의 Segment 신호와 Digit 선택 신호로 변환 (Dynamic Scanning) |
| **button\_debounce.v** | 입력 안정화 | 기계적 스위치의 채터링(Chattering) 현상을 제거하여 깨끗한 펄스 신호 생성 |

-----

## ⚡ 기술적 도전 & 트러블슈팅 (Troubleshooting)

프로젝트 진행 중 발생한 하드웨어적 문제점과 이를 해결한 과정입니다.

### 🛑 문제점 1: 스위치 채터링 (Switch Bouncing)

  * **현상:** 버튼을 한 번 눌렀음에도 불구하고 카운터가 여러 번 증가하거나 스톱워치가 시작하자마자 멈추는 현상 발생.
  * **원인:** 기계식 버튼 내부의 접점이 붙거나 떨어질 때 미세한 진동으로 인해 수십 ms 동안 High/Low 신호가 반복됨.
  * **해결책 (Debouncing Logic):**
      * 샘플링 기법을 적용하여 일정 시간(약 20ms) 동안 신호가 안정적으로 유지될 때만 유효한 입력으로 간주하는 `button_debounce` 모듈을 구현하여 해결.

### ⏱️ 문제점 2: 타이밍 오차 (Timing Inaccuracy)

  * **현상:** 스톱워치를 장시간 동작시켰을 때, 실제 시간(스마트폰 스톱워치)과 미세한 차이가 발생함.
  * **원인:** 100MHz 시스템 클럭을 단순히 카운터로 나누는 과정에서 발생한 미세한 틱(Tick) 오차 누적.
  * **해결책 (Precise Prescaler):**
      * 100MHz 클럭에서 정확히 1초(1Hz)와 0.01초(100Hz)를 생성하기 위한 카운터 상수를 정확하게 계산하여 적용 (`cnt == 100_000_000 - 1`).
      * 조건문(`>=` 대신 `==`)을 명확히 사용하여 불필요한 클럭 사이클 낭비를 방지.

-----

## 💡 배운 점 (Lessons Learned)

1.  **FSM 설계의 중요성:** 스톱워치의 동작(시작, 정지, 초기화)을 명확한 상태(State)로 정의하고 제어함으로써 복잡한 제어 로직을 체계적으로 구현하는 방법을 익혔습니다.
2.  **동적 디스플레이 원리:** 인간의 눈이 인식하지 못하는 속도로 FND를 빠르게 스위칭(Scanning)하여 적은 수의 I/O 핀으로 다수의 디스플레이를 제어하는 원리를 체득했습니다.
3.  **하드웨어 타이밍 제어:** 클럭 분주(Clock Division)와 디바운싱(Debouncing)을 직접 구현하며 디지털 회로에서의 타이밍 무결성(Timing Integrity)의 중요성을 배웠습니다.

-----

## 📂 폴더 구조 (Project Structure)

```bash
📦 FPGA_Watch_Project
 ├── 📂 src
 │    ├── 📜 top.v                # 최상위 모듈 (Watch + Stopwatch 통합)
 │    ├── 📜 watch_dp.v           # 시계 카운터 및 시간 로직
 │    ├── 📜 stopwatch_dp.v       # 스톱워치 데이터패스 (1/100초 카운터)
 │    ├── 📜 stopwatch_cu.v       # 스톱워치 FSM 제어기 (Run/Stop/Clear)
 │    ├── 📜 fnd_controller.v     # 7-Segment 디스플레이 스캐닝 제어
 │    ├── 📜 button_debounce.v    # 버튼 채터링 방지 모듈
 │    └── 📜 combiner.v           # 모드에 따른 출력 데이터 선택 (MUX)
 ├── 📂 constraint
 │    └── 📜 Basys-3-Master.xdc   # FPGA 핀 맵핑 파일
 └── 📜 README.md
```
