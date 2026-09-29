# Lv.2 모듈 1: OpenCR · 다이나믹셀 위치 P 제어

## 장비 · 환경
- 라즈베리파이: Ubuntu Server 22.04 LTS (aarch64). PC는 SSH 접속에만 사용하고, 빌드·업로드·시리얼 수신·로그 저장은 모두 라즈베리파이에서 수행
- 제어기: OpenCR 1.0, 라즈베리파이에 USB 연결 (`/dev/ttyACM0`)
- 모터: 다이나믹셀 ID 1, Model 1030, 1 Mbps, Protocol 2.0
- 예제: `opencr_position_p` (과제 저장소 제공 소스)
- 코드 수정: 기본 대상(ID 12, Model 1020)을 스캔 결과(ID 1, Model 1030)로 변경 → [opencr_position_p.ino](opencr_position_p/opencr_position_p.ino). 모델 검사 로직은 유지

| 도구 | 버전 |
|---|---|
| Arduino CLI | 1.5.1 (Linux ARM64) |
| OpenCR 코어 | 1.5.3 (수동 설치, FQBN `ROBOTIS:OpenCR:OpenCR`) |
| 컴파일러 | `arm-none-eabi-g++` 【TODO: `arm-none-eabi-g++ --version` 결과】 (Ubuntu 패키지) |
| Dynamixel2Arduino | commit `cfbbaf7` |
| 업로더 `opencr_ld` | OpenCR 저장소 commit `68ec75d`, 라즈베리파이에서 소스 빌드 |
| 시리얼 모니터 | `python3 -m serial.tools.miniterm`, 115200 baud |

## 환경변수
SSH에 새로 접속할 때마다 다시 설정한다.

```bash
export BASE="$HOME/pa-opencr-build"   # 빌드 작업 폴더
set -o pipefail                        # tee 사용 시 앞 명령 실패를 놓치지 않도록
PORT=/dev/ttyACM0                      # ls -l /dev/ttyACM* 로 확인한 OpenCR 포트
UPLOADER="$BASE/uploader-src/arduino/opencr_develop/opencr_ld/opencr_ld"
```

`dialout` 그룹 추가(`sudo usermod -aG dialout "$USER"`) 후 SSH에 다시 접속해야 포트에 접근할 수 있다.

## 실행 방법
설치 과정(CLI·코어·라이브러리·업로더)은 과제 저장소의 라즈베리파이 빌드 가이드를 따랐다.

1. 수정한 소스를 빌드한다.
   ```bash
   "$BASE/bin/arduino-cli" --config-file "$BASE/arduino-cli.yaml" \
     compile --fqbn ROBOTIS:OpenCR:OpenCR --jobs 1 \
     --output-dir "$BASE/output" \
     "$BASE/sketches/opencr_position_p" 2>&1 | tee "$BASE/build.log"
   ```
2. 직접 빌드한 `.bin`을 업로드하고 출력에서 `CRC OK`, `[OK] Download`를 확인한다.
   ```bash
   "$UPLOADER" "$PORT" 115200 "$BASE/output/opencr_position_p.ino.bin" 1 \
     2>&1 | tee "$BASE/upload.log"
   ```
3. 시리얼 모니터에 접속해 `?`를 입력하고 `READY` 상태를 확인한다. (종료: Ctrl+])
   ```bash
   python3 -m serial.tools.miniterm "$PORT" 115200 --eol LF -e
   ```
4. 명령을 입력한다. 형식: `s <Kp> <속도 상한 deg/s> <목표각 deg>`
   - 실행 A: `s 10 30 30`
   - 실행 B: `s 1 30 30`
5. 출력 로그를 `results/`에 저장한다.

## 결과 파일 위치
| 파일 | 내용 |
|---|---|
| [report.md](report.md) | 문제 1~4 보고서 |
| [results/환경확인.txt](results/환경확인.txt) | OS, 아키텍처, 포트, 다이나믹셀 스캔 결과 |
| [results/build.log](results/build.log) | 펌웨어 빌드 로그 |
| [results/upload.log](results/upload.log) | 펌웨어 업로드 로그 |
| [results/실행A.log](results/실행A.log) | 실행 A (Kp=10) 로그 |
| [results/실행B.log](results/실행B.log) | 실행 B (Kp=1) 로그 |
