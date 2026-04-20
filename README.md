# MIIIME Launcher Sweeper (MLS™)

MIIIMELauncherSweeper · 미메런처스위퍼<br>

![OS](https://img.shields.io/badge/Platform-Windows-0078D4?logo=windows&style=flat-square)
[![Language](https://img.shields.io/badge/Language-AutoIt-orange?logo=autoit&style=flat-square)](https://www.autoitscript.com/site/)
![License](https://img.shields.io/badge/License-Freeware-lightgrey?style=flat-square)

<br>
<img width="559" height="136" alt="001" src="https://github.com/miiimeworks/M4T/blob/main/4bit_Enhanced/Id/Neon/4b_136_0_G.png?raw=true" style="margin-top: 20px; margin-bottom: 20px;">
<br>

MIIIMEHybridLauncher uses NTFS junction points and creates registry backups and temporary session folders on the host.
MIIIMELauncherSweeper is a forensic cleanup utility for the MIIIME Hybrid Launcher.  

미메런처는 NTFS 정션 포인트를 사용하며,레지스트리 백업과 임시 세션 폴더를 호스트에 생성합니다.  
미메런처스위퍼는 미메런처의 비정상 종료 후 호스트에 잔류한 파일시스템 아티팩트를 탐지 · 제거하는 포렌식 정리 도구입니다.

<br>
<img width="402" height="197" alt="001" src="https://github.com/miiimeworks/MLS/blob/main/Preview/20260417_233754.png?raw=true" style="margin-top: 20px; margin-bottom: 20px;">  
<img width="402" height="197" alt="001" src="https://github.com/miiimeworks/MLS/blob/main/Preview/20260417_233813.png?raw=true" style="margin-top: 20px; margin-bottom: 20px;">  
<br>

---

## Features

### Scan

The utility first checks if the launcher is running; if active, it skips scanning the temporary folder.  
(This is because it is impossible to externally distinguish between active sessions and residual traces.)

런처가 실행 중인지 먼저 확인하고, 실행 중이면 임시 폴더 스캔을 건너뜀.  
(활성 세션과 잔류물을 외부에서 구분하는 것이 불가능하기 때문).

Four types of artifacts are detected.  
4종의 아티팩트를 탐지.

| Type       | Description                                                   | Scan Root                                                    |
| ---------- | ------------------------------------------------------------- | ------------------------------------------------------------ |
| `JUNCTION` | Residual NTFS junction points with a corresponding _bakk pair | `%AppData%`, `%LocalAppData%`, `%LocalLow%`, `%UserProfile%` |
| `BAKK`     | `*_bakk` *_bakk backup folders                                | Same                                                         |
| `TEMP`     | `MIIIME_*` temporary session folders                          | `%TEMP%`                                                     |
| `REGBAKK`  | `*.reg.bakk` registry snapshot files                          | `%TEMP%\MIIIME_*\RegBakk\`                                   |

### Cleanup

Detected artifacts are processed in the following order.  
탐지된 아티팩트를 다음 순서로 처리.

```
1. JUNCTION  →  2. BAKK  →  3. TEMP  →  4. REGBAKK
```

Junction points are removed using the kernel32.dll RemoveDirectoryW native API,  
with a fallback to cmd.exe rmdir upon failure.  
File and folder deletions include a retry logic (up to 10 times) to handle lock conflicts.

정션 포인트는 `kernel32.dll RemoveDirectoryW` 네이티브 API로 제거하며,  
실패 시 `cmd.exe rmdir` 폴백을 사용.  
파일·폴더 삭제는 재시도 로직(최대 10회)으로 잠금 충돌에 대응.

### Operation Modes

Three execution modes are supported.  
3가지 실행 모드를 지원.

| Mode          | Description                                                                     |
| ------------- | ------------------------------------------------------------------------------- |
| Default (GUI) | Displays detection results in a ListView and cleans up after user confirmation. |
| `--scan-only` | Read-only scan that displays detection results without performing cleanup.      |
| `--auto`      | Automatic cleanup without GUI (for schedulers or script integration).           |

---

## CLI Arguments

```bash
MIIIMELauncherSweeper.exe [Options]
```

| Argument      | Description                                                              |
| ------------- | ------------------------------------------------------------------------ |
| `--auto`      | **Auto Mode** : Executes cleanup immediately without a confirmation GUI. |
| `--scan-only` | **Scan Only** : Displays detection results only; no deletion occurs.     |

인수 없이 실행하면 탐지 결과를 표시하고 사용자 확인 후 정리.

---

## Configuration (`MIIIMELauncherSweeper.ini`)

### [Options]

| Key        | Type   | Default | Description                                                |
| ---------- | ------ | ------- | ---------------------------------------------------------- |
| `LogLevel` | Int    | `0`     | 0=Off, 1=All(no DEBUG), 2=DEBUG, 3=INFO+, 4=WARN+, 5=ERROR |
| `Local`    | String | ``      | `Kr` = Korean. Leave empty for English                     |

### [Advanced]

| Key                | Default                        | Description                                                          |
| ------------------ | ------------------------------ | -------------------------------------------------------------------- |
| `LogRotationSize`  | `5242880`                      | Maximum log file size in bytes. Rotates to `.log.old` when exceeded  |
| `LauncherSuffixes` | `_M`                           | Launcher identifier suffix. Multiple values separated by pipe (`\|`) |
| `BakkMarker`       | `_bakk`                        | Backup folder identifier marker                                      |
| `TempPrefix`       | `MIIIME_`                      | Temporary session folder identifier prefix                           |
| `LockFiles`        | `Cleanup.lock\|SysInject.lock` | Lock file name list                                                  |
| `StateFile`        | `State.ini`                    | Launcher state file name                                             |

---

## Scan Logic Details

### Launcher Running Check

- The utility checks for running launchers in the process list using the LauncherSuffixes + .exe pattern.  
- If a launcher is running, the entire TEMP scan is skipped.

**[런처 실행 중 처리]**  

- 프로세스 목록에서 `LauncherSuffixes` + `.exe` 패턴으로 런처 실행 여부를 확인.  
- 런처가 실행 중이면 **TEMP 스캔 전체를 건너뜀.**

### JUNCTION Detection

- Detected as a JUNCTION type if the original path paired with a _bakk folder is an NTFS junction point (attribute L).  
- If the original is missing, it is labeled as BAKK; if it is a normal folder, a Conflict status is shown.

**[JUNCTION 탐지]**  

- `_bakk` 폴더와 쌍을 이루는 원본 경로가 NTFS 정션 포인트(`L` 속성)인 경우 `JUNCTION` 유형으로 탐지.  
- 원본이 없으면 `BAKK`만, 원본이 일반 폴더이면 `Conflict` 정보를 표시.

### REGBAKK Detection

- RegBakk\*.reg.bakk files within temporary session folders are detected as individual items.  
- While they are removed during TEMP cleanup, they are re-processed as independent items if initial cleanup fails.

**[REGBAKK 탐지]** 

- 임시 세션 폴더 내의 `RegBakk\*.reg.bakk` 파일을 개별 항목으로 탐지.  
- TEMP 항목 삭제로 함께 제거되지만, 정리 실패 시 독립 항목으로 재처리.

---

## Directory Structure

```text
MIIIMELauncherSweeper/
  ├─ MIIIMELauncherSweeper.exe      # Executable / 실행 파일
  ├─ MIIIMELauncherSweeper.ini      # Configuration file / 설정 파일
  ├─ MIIIMELauncherSweeper.log      # Log file (Created when active) / 로그 파일
  └─ Loc/
      └─ Kr.ini                     # Korean language file (Optional) / 한국어 언어 파일
```

---

## 🛡️ Security & Anti-virus Info

### [✅ VirusTotal Analysis Report](https://www.virustotal.com/gui/file/5593ab1baa4180f52b912ee4c6911e1093325124124ff81fad4838822c29c67d?nocache=1)

| Status             | Details                                                                        |
|:------------------ |:------------------------------------------------------------------------------ |
| **Major Vendors**  | **Clean** (Passed by AhnLab V3, Kaspersky, Microsoft, Avast, ESET, etc.)       |
| **Detection Rate** | **8 / 72** (Mostly Heuristic/Generic/Trojan-type flags)                        |
| **Integrity**      | The source code is transparently available for verification in this repository |

> This Program was created with AutoIt. Some antivirus programs may incorrectly detect it as a virus.  
> 본 프로그램은 AutoIt으로 제작되었습니다. 일부 백신이 바이러스로 오진 할 수 있습니다. 

**File Checksum (SHA-256) :** `5593ab1baa4180f52b912ee4c6911e1093325124124ff81fad4838822c29c67d`

---

## Disclaimer

Provided **“AS IS”**, without warranty.  
This is a **private project**. No technical support is provided.

본 프로그램은 **“있는 그대로”** 제공되며, 사용 중 발생하는 문제에 대해 제작자는 책임을 지지 않습니다.  
기술 지원은 제공되지 않습니다.

---

## Project Information

**Developer** : MIIIME  
**Website** : https://www.miiime.com  
**GitHub** : [@miiime6248](https://github.com/miiime6248)  
**Last Update** : 2026-02-14  

<br>
<img width="64" height="64" alt="002" src="https://github.com/miiimeworks/M4T/blob/main/4bit_Brutal/Logo/Neon/4b_Mium_64_0_G.png?raw=true">  
<br>
<br>
<br>