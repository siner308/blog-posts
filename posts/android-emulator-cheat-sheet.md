---
title: "안드로이드 에뮬레이터 명령어 정리 - Android Emulator cheat sheet"
subtitle: "Android Studio 없이 CLI로만 에뮬레이터를 키우고 쓰는 법"
tags:
    - cheat sheet
    - android
date: 2026-09-15
image: https://raw.githubusercontent.com/siner308/blog-posts/master/assets/android-emulator-cheat-sheet-thumbnail.png
---

Android Studio를 켜지 않고도 에뮬레이터를 쓰고 싶었습니다. 띄우고 로그 보고 apk 하나 설치하면 끝인데 그걸 위해 IDE를 통째로 여는 게 아까웠습니다.
`sdkmanager`, `avdmanager`, `emulator`, `adb` 네 개면 충분합니다. 아래는 맥(Apple Silicon)에서 확인한 내용입니다.

## 설치와 PATH

Android Studio를 이미 깔았다가 지웠다면 SDK는 `~/Library/Android/sdk`에 남아있을 겁니다. 거기에 `cmdline-tools`만 얹으면 됩니다.

```bash
# https://developer.android.com/studio#command-line-tools-only 에서 mac용 zip 링크 확인
cd /tmp
curl -LO https://dl.google.com/android/repository/commandlinetools-mac-13114758_latest.zip
unzip -q commandlinetools-mac-13114758_latest.zip

SDK=~/Library/Android/sdk
mkdir -p $SDK/cmdline-tools
mv cmdline-tools $SDK/cmdline-tools/latest   # 반드시 latest 라는 이름이어야 합니다
```

`latest`가 아니면 `sdkmanager`가 자기 위치를 SDK 루트로 착각해서 엉뚱한 곳에 패키지를 받습니다.

`~/.zshrc`:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"
```

확인:

```bash
adb --version && emulator -version && avdmanager list target
```

## sdkmanager - 패키지 찾고 받기

[sdkmanager](https://developer.android.com/tools/sdkmanager)

```bash
sdkmanager --list                  # 받을 수 있는 전체 목록
sdkmanager --list_installed        # 지금 깔린 것만
sdkmanager --update                # 이미 깔린 패키지 최신화
```

system image 경로는 `system-images;<api>;<태그>;<abi>` 꼴입니다. 목록이 수백 줄이라 grep이 사실상 필수입니다.

```bash
# Apple Silicon에서 쓸 수 있는 최신 이미지만 추리기
sdkmanager --list | grep 'system-images' | grep 'arm64-v8a' | grep 'google_apis_playstore'
```

```
system-images;android-35;google_apis_playstore;arm64-v8a   | 9 | Google Play ARM 64 v8a System Image
system-images;android-36;google_apis_playstore;arm64-v8a   | 7 | Google Play ARM 64 v8a System Image
```

| 태그 | 내용 |
|------|------|
| `google_apis_playstore` | Play 스토어 포함. 대신 `adb root`가 막힙니다 |
| `google_apis` | Play 서비스는 있고 스토어는 없음. `adb root` 가능 |
| `default` | 순정 AOSP. 구글 서비스 없음 |
| `aosp_atd`, `google_atd` | 자동화 테스트용 경량 이미지 (API 30 계열에 있습니다) |

Apple Silicon이면 `arm64-v8a`를 받습니다. `x86_64` 이미지는 에뮬레이션이 한 겹 더 끼어 느린 것으로 보입니다.

설치:

```bash
sdkmanager "system-images;android-36;google_apis_playstore;arm64-v8a"
sdkmanager "platforms;android-36" "platform-tools" "emulator"
sdkmanager --licenses      # 라이선스 동의가 안 되어 있으면 설치가 막힙니다
```

## avdmanager - AVD 만들고 관리하기

[avdmanager](https://developer.android.com/tools/avdmanager)

AVD(Android Virtual Device)는 "어떤 기기 껍데기에 어떤 시스템 이미지를 올릴까"를 적어둔 설정 묶음입니다.

```bash
avdmanager list device     # 기기 프로파일 (pixel_7, pixel_tablet ...)
avdmanager list target     # 설치된 API 레벨
avdmanager list avd        # 내가 만든 AVD
emulator -list-avds        # 이름만 빠르게
```

생성:

```bash
avdmanager create avd \
  -n Pixel_7_API_36 \
  -k "system-images;android-36;google_apis_playstore;arm64-v8a" \
  -d pixel_7
```

| 옵션 | 내용 |
|------|------|
| `-n` | AVD 이름. 공백 대신 언더스코어를 씁니다 |
| `-k` | system image 경로. `sdkmanager --list`의 그 문자열 그대로 |
| `-d` | 기기 프로파일. 해상도와 dpi가 여기서 결정됩니다 |
| `-c 2G` | SD카드 크기 |
| `-f` | 같은 이름이 있으면 덮어쓰기 |
| `--abi` | 이미지에 ABI가 여럿일 때만 필요 |

이름 변경과 삭제:

```bash
avdmanager move avd -n Pixel_7_API_36 -r Pixel_7_Test
avdmanager move avd -n Pixel_7_Test -p ~/avds/Pixel_7_Test.avd   # 경로 이동
avdmanager delete avd -n Pixel_7_Test
```

## config.ini - AVD 설정 바꾸기

AVD의 실체는 `~/.android/avd/<이름>.avd/config.ini` 텍스트 파일 하나입니다. GUI 설정 화면이 하던 일은 전부 이 파일 편집입니다.

```bash
vi ~/.android/avd/Pixel_7_API_36.avd/config.ini
```

```ini
hw.ramSize = 4096                    # MB 또는 4G
vm.heapSize = 512                    # 앱당 힙 상한
hw.cpu.ncore = 4
disk.dataPartition.size = 16G        # /data 크기
sdcard.size = 2G
hw.lcd.width = 1080
hw.lcd.height = 2400
hw.lcd.density = 420                 # 420 = xhdpi 근처
hw.keyboard = yes                    # 맥 키보드로 입력. 켜두는 게 편합니다
hw.gpu.enabled = yes
hw.gpu.mode = auto
showDeviceFrame = no                 # 기기 테두리 그림 끄기
```

RAM은 명령줄에서도 덮어쓸 수 있습니다. `emulator -avd <이름> -memory 4096`.

## 디스크 키우기

`/data`가 꽉 차서 apk 설치가 실패하면 여기입니다. 현재 용량부터 봅니다.

```bash
adb shell df -h /data
```

```
Filesystem       Size Used Avail Use% Mounted on
/dev/block/dm-44 5.8G 986M  4.7G  17% /data/user/0
```

```bash
# 1) 영구 - config.ini 수정 후 데이터 초기화 부팅
#    disk.dataPartition.size = 16G 로 고친 다음
emulator -avd Pixel_7_API_36 -wipe-data

# 2) 일회성 - 실행할 때만 크게 (MB 단위)
emulator -avd Pixel_7_API_36 -partition-size 16384
```

`-wipe-data`는 앱과 데이터를 전부 날립니다. 파티션을 늘리려면 이미지를 다시 만들어야 해서 피할 수 없으니, 지우기 싫으면 먼저 백업해 둡니다.

```bash
adb backup -f pre-resize.ab -apk -all       # deprecated 경고가 뜨고, 기기에서 확인을 눌러야 진행됩니다
adb shell pm list packages -3                # 최소한 설치 목록이라도 남겨둡니다
```

## 실행

[Start the emulator from the command line](https://developer.android.com/studio/run/emulator-commandline)

```bash
emulator -avd Pixel_7_API_36                              # 기본
emulator -avd Pixel_7_API_36 -no-snapshot-load            # 스냅샷 무시하고 풀부팅
emulator -avd Pixel_7_API_36 -no-window -gpu swiftshader_indirect   # headless, CI용
emulator -avd Pixel_7_API_36 -memory 4096 -cores 4
emulator -avd Pixel_7_API_36 -writable-system             # adb remount 하려면 필요
emulator -avd Pixel_7_API_36 -http-proxy http://127.0.0.1:8888   # 프록시로 트래픽 보기
emulator -avd Pixel_7_API_36 -dns-server 8.8.8.8
emulator -avd Pixel_7_API_36 -netdelay gprs -netspeed edge       # 느린 망 흉내
```

부팅이 끝났는지 기다리는 한 줄입니다.

```bash
adb wait-for-device shell 'while [ "$(getprop sys.boot_completed)" != 1 ]; do sleep 1; done'
```

### 여러 대 동시에

```bash
emulator -avd A -port 5554 &
emulator -avd B -port 5556 &
adb devices                       # emulator-5554, emulator-5556
adb -s emulator-5556 shell        # 대상 지정
export ANDROID_SERIAL=emulator-5556   # 이후 adb 명령의 기본 대상
```

같은 AVD를 두 개 띄우려면 `-read-only`를 붙입니다. 대신 스냅샷 저장이 막힙니다.

### 스냅샷

```bash
emulator -avd Pixel_7_API_36 -snapshot-list
emulator -avd Pixel_7_API_36 -snapshot clean-state
emulator -avd Pixel_7_API_36 -no-snapshot-save      # 종료 시 상태를 버립니다
```

## adb - 앱 설치와 디버깅

[Android Debug Bridge (adb)](https://developer.android.com/tools/adb)

```bash
adb devices -l                    # 연결 목록
adb install -r app.apk            # 재설치 (데이터 유지)
adb install -t app.apk            # test-only 플래그 붙은 빌드
adb install -d app.apk            # 버전 다운그레이드 허용
adb install-multiple base.apk split_config.arm64_v8a.apk   # app bundle 결과물
adb uninstall com.example.app
adb shell pm clear com.example.app          # 앱 데이터만 초기화
adb shell pm list packages -3               # 직접 설치한 앱만
adb shell pm path com.example.app           # 설치된 apk 경로
adb shell dumpsys package com.example.app | grep versionName
```

설치된 apk 빼오기:

```bash
adb pull $(adb shell pm path com.example.app | sed 's/package://' | tr -d '\r') ./app.apk
```

액티비티와 딥링크:

```bash
adb shell am start -n com.example.app/.MainActivity
adb shell am start -a android.intent.action.VIEW -d "myapp://promotion/1234"
adb shell am force-stop com.example.app
```

로그:

```bash
adb logcat -c                               # 버퍼 비우기
adb logcat                                  # 전부
adb logcat MyTag:D '*:S'                    # MyTag만 남기고 나머지 침묵
adb logcat --pid=$(adb shell pidof -s com.example.app)   # 특정 앱만
adb logcat -v time -e 'HTTP|api' | tee run.log           # 정규식 필터
adb logcat -b crash                         # 크래시 버퍼
```

화면:

```bash
adb exec-out screencap -p > shot.png
adb shell screenrecord /sdcard/demo.mp4     # ctrl-c 로 중단
adb pull /sdcard/demo.mp4
```

입력과 파일:

```bash
adb shell input text "hello"
adb shell input keyevent KEYCODE_BACK       # 4=BACK, 3=HOME, 66=ENTER
adb shell input tap 540 1200
adb shell input swipe 540 1800 540 600
adb push local.json /sdcard/Download/
adb pull /sdcard/Download/result.json .
```

권한과 루트:

```bash
adb shell pm grant com.example.app android.permission.CAMERA
adb shell pm revoke com.example.app android.permission.CAMERA
adb root                                    # playstore 이미지에서는 막혀 있습니다
adb remount                                 # -writable-system 으로 띄웠을 때만
```

## 네트워크

에뮬레이터 안에서 맥의 `localhost`는 `10.0.2.2`입니다. 맥에서 8080 서버를 띄웠다면 앱에서는 `http://10.0.2.2:8080`으로 호출합니다.

에뮬레이터의 포트를 맥에서 열고 싶으면 `forward`, 에뮬레이터가 맥의 포트를 자기 `localhost`로 보게 하려면 `reverse`입니다.

```bash
adb forward tcp:9222 tcp:9222     # 에뮬레이터의 9222를 맥의 9222로
adb reverse tcp:8080 tcp:8080     # 에뮬레이터의 localhost:8080 을 맥으로
adb forward --list
adb reverse --remove-all
```

`reverse`를 쓰면 앱 코드의 `localhost:8080`을 안 고치고도 맥의 서버에 붙습니다.

## 에뮬레이터 콘솔 (adb emu)

[Send emulator console commands](https://developer.android.com/studio/run/emulator-console)

```bash
adb emu avd name                  # 지금 이 에뮬레이터의 AVD 이름
adb emu kill                      # 종료
adb emu geo fix 127.0276 37.4979  # 경도 위도 순서입니다
adb emu sms send 01012345678 "인증번호 123456"
adb emu power capacity 15         # 배터리 15%
adb emu power ac off              # 충전 해제
adb emu network speed gsm         # 느린 망
adb emu rotate                    # 화면 회전
adb emu finger touch 1            # 지문 인식 (등록된 finger id)
```

## 자주 막히는 곳

```bash
adb kill-server && adb start-server         # 기기가 안 잡힐 때 첫 번째 시도
adb devices                                  # offline 이면 에뮬레이터를 재시작합니다
```

| 증상 | 원인과 처방 |
|------|------------|
| `avdmanager`가 실행이 안 됨 | JDK 17 이상이 필요합니다. `java -version` 확인 |
| `sdkmanager`가 엉뚱한 곳에 설치 | `cmdline-tools/latest` 이름을 확인합니다 |
| 부팅이 한없이 느림 | `x86_64` 이미지를 받았을 수 있습니다. `arm64-v8a`로 다시 받습니다 |
| `adb root`가 거부됨 | playstore 이미지의 정책입니다. `google_apis` 이미지로 만듭니다 |
| `INSTALL_FAILED_INSUFFICIENT_STORAGE` | 위의 디스크 키우기 |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | 서명이 다른 같은 패키지입니다. 먼저 `adb uninstall` |
| 화면이 검은 채로 멈춤 | `-gpu swiftshader_indirect`로 띄워봅니다 |
| 스냅샷 복구 후 상태가 이상함 | `-no-snapshot-load`로 풀부팅 |

## 한 번에 쓰는 함수

`.zshrc`에 넣어두고 씁니다.

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"

emu() {  # emu           -> AVD 목록
         # emu <이름>    -> 백그라운드로 부팅하고 준비될 때까지 대기
  if [ -z "$1" ]; then
    emulator -list-avds
    return
  fi
  nohup emulator -avd "$1" > /tmp/emulator-$1.log 2>&1 &
  adb wait-for-device shell 'while [ "$(getprop sys.boot_completed)" != 1 ]; do sleep 1; done'
  echo "booted: $(adb emu avd name | head -1)"
}

applog() {  # applog com.example.app
  adb logcat --pid=$(adb shell pidof -s "$1")
}
```

IDE를 안 열어도 되니 부팅이 빠르고, 무엇보다 스크립트에 넣을 수 있어서 좋았습니다. CI에서 `-no-window`로 띄우는 것도 같은 명령어 그대로입니다.

## references

- [sdkmanager](https://developer.android.com/tools/sdkmanager)
- [avdmanager](https://developer.android.com/tools/avdmanager)
- [Start the emulator from the command line](https://developer.android.com/studio/run/emulator-commandline)
- [Send emulator console commands](https://developer.android.com/studio/run/emulator-console)
- [Android Debug Bridge (adb)](https://developer.android.com/tools/adb)
- [Logcat command-line tool](https://developer.android.com/tools/logcat)
