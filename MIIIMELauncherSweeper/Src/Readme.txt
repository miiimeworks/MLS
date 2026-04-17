========================================================================
		MIIIME Launcher Sweeper (MLS™)
========================================================================

MIIIMEHybridLauncher uses NTFS junction points and creates registry backups and temporary session folders on the host. 
MIIIMELauncherSweeper is a forensic cleanup utility for the MIIIME Hybrid Launcher.

미메런처는 NTFS 정션 포인트를 사용하며,레지스트리 백업과 임시 세션 폴더를 호스트에 생성합니다.
미메런처스위퍼는 미메런처의 비정상 종료 후 호스트에 잔류한 파일시스템 아티팩트를 탐지 · 제거하는 포렌식 정리 도구입니다.

========================================================================
[Features]
========================================================================

1. Artifact Detection
   - Scan Roots :
     %AppData%, %LocalAppData%, %AppData%\..\LocalLow, %UserProfile%
     Scans for *_bakk folders left by AppDat and Usr injection.
   - Junction Check : 
     If the original path of a _bakk pair has the L (Reparse Point)
     attribute, it is flagged as a stranded Junction.
   - Temp Sessions : 
     Enumerates %TEMP%\MIIIME_* directories from previous sessions.
   - False-Positive Guard :
     Only directory-type entries are processed.
     Plain files matching the suffix are excluded.

   [아티팩트 탐지]
   - 스캔 범위 : 
     %AppData%, %LocalAppData%, %AppData%\..\LocalLow, %UserProfile%
     AppDat 및 Usr 주입이 남긴 *_bakk 폴더 탐색.
   - Junction 확인 : 
     _bakk 쌍의 원본 경로에 L (Reparse Point) 속성이 있으면 잔류 Junction으로 표시.
   - 임시 세션 : 
     이전 세션의 %TEMP%\MIIIME_* 폴더를 열거.
   - 오탐 방지 : 
     폴더 유형 항목만 처리함. 
     접미사가 일치하더라도 일반 파일은 제외.

2. Cleanup Sequence
   Processing is ordered to prevent dependency conflicts.
   처리는 의존성 충돌을 방지하기 위해 순서가 보장됩니다.

   Step 1 [JUNCTION] : Native removal via RemoveDirectoryW -> cmd.exe rmdir fallback
   Step 2 [BAKK] : Restore to original path if absent; delete if original already exists
   Step 3 [TEMP] : Delete entire MIIIME_* session directory
   Step 4 [REGBAKK] : Delete orphaned *.reg.bakk files (if not removed in step 3)

3. Launcher Conflict Prevention
   If a _M.exe process is detected at startup, the entire MIIIME_* temp folder scan is skipped.
   The _bakk scan continues regardless, as those artifacts are safe to process.

   런처(_M.exe) 프로세스가 감지되면 MIIIME_* 임시 폴더 스캔 전체를 건너뜁니다.
   _bakk 스캔은 런처 실행 여부와 관계없이 안전하므로 계속 진행됩니다.

4. Operation Modes
   세 가지 실행 모드를 지원합니다.

   Default (GUI) : Displays detection results in a ListView and cleans up after user confirmation.
   --scan-only : Read-only scan; displays results without performing cleanup.
   --auto : Automatic cleanup without GUI (for schedulers or script integration).

   기본 (GUI) : 탐지 결과를 ListView에 표시하고 사용자 확인 후 정리.
   --scan-only : 읽기 전용 스캔. 탐지 결과만 표시하며 삭제는 수행하지 않음.
   --auto : GUI 없이 자동 정리 (스케줄러·스크립트 연동용).

========================================================================
[Configuration]
========================================================================

1. File Placement
   Place the optional .ini in the same directory as the executable.
   .ini는 실행 파일과 동일한 디렉토리에 배치.

2. Key INI Options (MIIIMELauncherSweeper.ini)

   [Options]
   LogLevel : 0=Off  1=All  2=DEBUG  3=INFO+  4=WARN+  5=ERROR
   Local : Kr = Korean. Leave empty for English.

   [Advanced]
   LauncherSuffixes : Launcher identifier suffix. Default: _M  (pipe-separated for multiple)
   BakkMarker : Backup folder marker. Default: _bakk
   TempPrefix : Temp session folder prefix. Default: MIIIME_

3. Directory Structure (상세 폴더 구조) 

   MIIIMELauncherSweeper/
       ├─ MIIIMELauncherSweeper.exe    	# Sweeper Executable / 스위퍼 실행 파일
       ├─ MIIIMELauncherSweeper.ini    	# Configuration (optional) / 설정 파일 (선택)
       ├─ MIIIMELauncherSweeper.log    	# Runtime Log / 실행 로그 (활성 시 생성)
       └─ Loc/
           └─ Kr.ini                   			# Korean language file (optional) / 한국어 언어 파일 (선택)

______________________________________________________________________________________________________________________

This program was created with AutoIt. Some antivirus programs may incorrectly detect it as a virus.
본 프로그램은 AutoIt으로 제작되었습니다. 일부 백신이 바이러스로 오진 할 수 있습니다.

========================================================================
[Disclaimer]
========================================================================

Provided "AS IS", without warranty.
This is a private project. No technical support is provided.
본 프로그램은 "있는 그대로" 제공되며, 사용 중 발생하는 문제에 대해 제작자는 책임을 지지 않습니다.
기술 지원은 제공되지 않습니다.

Developer	: MIIIME 
Website		: https://www.miiime.com 
GitHub		: @miiime6248 
Update		: 2026.03.15
