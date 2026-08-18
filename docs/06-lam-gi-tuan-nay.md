# 6. Làm gì ngay tuần này

> Phần hữu ích nhất trong ngắn hạn. Mọi script đã được sửa để chạy được — bản đầu có 16 lỗi khiến copy-paste sẽ fail.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

*Phần này hữu ích nhất trong ngắn hạn. Nếu bạn cần test app của chính mình tuần này, đọc bảng 10.1 rồi làm từ trên xuống. Mọi script dưới đây đã được viết lại để chạy được — phần trước của tài liệu có 16 lỗi khiến copy-paste sẽ fail.*

**Nguyên tắc sắp xếp:** xếp theo ROI thật (fidelity × chi phí × thời gian dựng), không theo độ "ngầu" kỹ thuật. Ba mục đầu tiên không cần VM, không cần hypervisor, không cần một dòng code virtualization nào — và chúng bắt được nhiều bug thật hơn phần còn lại cộng lại.

---

## 10.1. Bảng quyết định 30 giây

| Bạn cần gì | Làm gì | Chi phí | Thời gian dựng | Mục |
|---|---|---|---|---|
| **Bug THẬT trên máy THẬT của user THẬT** | Sentry/Crashpad + kênh beta 5-10 tình nguyện viên Windows/Linux | **$0** (free tier) | **2-3 giờ** | §10.2 |
| **Assertion thật** (không phải nhìn ảnh) về UI Electron | Playwright `_electron.launch()` — chạy ngay trên Mac | **$0** | **1 giờ** | §10.3 |
| Smoke test binary Linux **x86-64** ngay trên Mac | Docker/OrbStack `--platform linux/amd64`, hoặc Apple `container --arch amd64` | **$0** | **15 phút** | §10.4 |
| Build installer Windows/Linux + smoke test headless + Defender scan | **GitHub Actions** | $0 (public repo) / ~$0.17 mỗi lần chạy (private) | 30 phút | §10.5 |
| Test **.deb/.rpm/.AppImage cài được không**, nhiều distro, kiến trúc x86-64 thật | VPS Linux + `systemd-nspawn --ephemeral` | ~$10-24/tháng (SG) hoặc ~€4.3/tháng (EU) | 1 giờ | §10.6 |
| **Windows x64 THẬT** cho vài giờ, có API spin up/tear down | **Azure/AWS VM tính theo giờ** (Windows **Server**) | **~$2-12/tháng** ở mức 20 giờ | 45 phút | §10.7 |
| **Nhìn tận mắt** app trên Windows, click thử, debug | **UTM + Windows 11 ARM64** trên chính Mac | $0 (+ license Windows) | 1-2 giờ | §10.8 |
| Smoke test installer trong clean room trên máy Windows x64 | **Windows Sandbox** (Pro/Enterprise, không có trên Home) | $0 | 10 phút | §10.9 |
| Test **USB token, driver kernel-mode, anti-cheat, DX12, app 32-bit, WSL2** | **Mini PC N100 / laptop Windows cũ** — **kiểm tra edition trước khi mua** | ~$250 một lần (+ Pro nếu máy ship Home) | 1 giờ | §10.10 |
| **Tái hiện một bug user vừa báo** | Quy trình log/dump/symbol có sẵn công thức | $0 | 30 phút setup | §10.11 |
| **Build** target Windows (MAUI/WinUI3/Flutter desktop) | Bắt buộc có môi trường Windows — dùng `windows-latest` trên GHA | $0-0.010/phút | 30 phút | §10.5 |

**Đọc bảng này thế nào:** nếu app của bạn đã có người dùng, làm §10.2 trước tất cả. Nếu chưa có ai dùng, làm §10.3 + §10.4 hôm nay, §10.5 ngày mai.

---

## 10.2. ROI cao nhất: telemetry + kênh beta — fidelity 100%, $0

Đây là thứ **không VM nào trên Apple Silicon cho được**: hành vi thật, trên phần cứng thật, kernel thật, driver thật, Defender thật, độ phân giải thật, font thật, locale thật. Với một dev solo Mac-only, đây gần như chắc chắn là hành động đúng số 1 — trên cả mini PC.

### 10.2.1. Crash reporting (30 phút)

```javascript
// src/main/telemetry.js — chạy trong main process của Electron
const Sentry = require('@sentry/electron/main');
const { app } = require('electron');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  release: `myapp@${app.getVersion()}`,
  environment: app.isPackaged ? 'production' : 'dev',
  // Bật native crash (Crashpad) để bắt cả crash trong .node addon
  enableNative: true,
  beforeSend(event) {
    // REDACTION bắt buộc: đường dẫn chứa tên user, token trong URL, email khách
    const json = JSON.stringify(event)
      .replace(/([A-Za-z]:\\Users\\)[^\\"]+/g, '$1<user>')
      .replace(/(\/home\/)[^\/"]+/g, '$1<user>')
      .replace(/([?&](token|key|api_key)=)[^&"]+/gi, '$1<redacted>');
    return JSON.parse(json);
  },
});

// Tag quan trọng nhất: bạn cần biết bug xảy ra trên kiến trúc nào
Sentry.setTag('os', process.platform);
Sentry.setTag('arch', process.arch);          // x64 | arm64
Sentry.setTag('os_release', require('os').release());
```

```javascript
// src/renderer/telemetry.js
import * as Sentry from '@sentry/electron/renderer';
Sentry.init({});   // kế thừa config từ main process
```

Upload symbol/sourcemap trong CI để stack trace đọc được:

```bash
npx @sentry/cli sourcemaps upload --release "myapp@$(node -p 'require("./package.json").version')" ./dist
# Với native module / Crashpad:
npx @sentry/cli debug-files upload --include-sources ./dist
```

**Cộng thêm `electron-log` để user gửi log cho bạn bằng một cú click:**

```javascript
const log = require('electron-log/main');
log.initialize();
log.transports.file.level = 'info';
// Windows: %USERPROFILE%\AppData\Roaming\<app>\logs\main.log
// Linux  : ~/.config/<app>/logs/main.log
// Thêm một menu item "Mở thư mục log" — giảm 80% ma sát khi hỏi user
```

### 10.2.2. Kênh beta 5-10 tình nguyện viên (2 giờ)

Đây là cách phần lớn dev cross-platform solo thật sự đang làm. Cụ thể:

| Việc | Cách làm |
|---|---|
| Tìm người | Discord/Telegram/Facebook group của chính sản phẩm; đổi lại bằng license vĩnh viễn hoặc tên trong CREDITS |
| Phân phối bản beta | GitHub Releases với tag `-beta`, hoặc kênh `beta` của electron-updater (`channel: beta` trong `publish` config) |
| Thu thập có cấu trúc | Issue template bắt buộc 5 trường: OS + build number (`winver`), **kiến trúc** (x64/arm64), phiên bản app, file `main.log`, video/GIF |
| Cover đúng ma trận | Cố ý chọn: 2 người Windows 11 x64, 1 người Windows 11 ARM (Snapdragon X), 1 người Windows 10 (đã EOL 14/10/2025 nhưng vẫn còn user), 2 người Ubuntu/Fedora, 1 người màn 4K scaling 150%, 1 người locale không phải en-US |

**Chi phí: $0. Fidelity: 100%.** Đổi lại: chậm (chu kỳ vài ngày), không tái hiện được theo yêu cầu, và bạn nợ họ sự tôn trọng — trả lời mọi report, kể cả report tệ.

---

## 10.3. Playwright cho Electron — assertion thật, chạy ngay trên Mac, $0

Toàn bộ phần trước của tài liệu cược vào "poll screenshot + pixel-stability" để biết app đã lên. Đó là cơ chế yếu và flaky hơn nhiều so với thứ đã tồn tại miễn phí. Playwright điều khiển được Electron trực tiếp qua CDP, chạy trên macOS/Linux/Windows và trên mọi CI.

```bash
npm i -D @playwright/test
```

```typescript
// tests/smoke.spec.ts
import { test, expect, _electron as electron } from '@playwright/test';

test('app khởi động, cửa sổ chính hiện ra, không tràn chữ', async () => {
  const app = await electron.launch({
    args: ['.'],
    env: { ...process.env, NODE_ENV: 'test' },
  });

  const win = await app.firstWindow();
  await expect(win).toHaveTitle(/MyApp/);
  await expect(win.getByRole('button', { name: 'Lưu' })).toBeVisible();

  // Lớp bug tràn chữ / cắt chữ — bắt được 100% trên chính máy Mac,
  // không cần VM, không cần Windows, không cần license.
  const clipped = await win.evaluate(() =>
    [...document.querySelectorAll<HTMLElement>('button, label, .nav-item, th')]
      .filter(el => el.scrollWidth > el.clientWidth + 1)
      .map(el => el.textContent?.trim() ?? '')
  );
  expect(clipped, `Text bị cắt: ${clipped.join(' | ')}`).toEqual([]);

  await win.screenshot({ path: 'runs/mac-baseline.png' });
  await app.close();
});
```

**Pseudolocalization — 20 dòng, bắt lớp bug đắt nhất cho app đa ngôn ngữ.** Thay mọi chuỗi UI bằng phiên bản dài hơn 40% và có dấu, chạy lại đúng test trên:

```typescript
// Trong app, khi NODE_ENV=test và PSEUDO=1:
export function t(key: string): string {
  const s = strings[key];
  if (process.env.PSEUDO !== '1') return s;
  const map: Record<string, string> = { a:'ä', e:'é', i:'ï', o:'ö', u:'ü', A:'Ä', E:'É' };
  const accented = [...s].map(c => map[c] ?? c).join('');
  return `[${accented}${'~'.repeat(Math.ceil(s.length * 0.4))}]`;
}
```

```bash
PSEUDO=1 npx playwright test tests/smoke.spec.ts
```

Nếu test pass với pseudolocale, layout của bạn sống sót được tiếng Đức, tiếng Việt có dấu, và font fallback. Đây là 2-3 tiếng công việc bắt được lớp bug mà cả một dàn VM không bắt nổi.

**Khi nào screenshot mới là công cụ đúng:** khi bạn KHÔNG điều khiển được app bằng automation — installer native, dialog hệ thống, tray icon, app không phải Electron/web. Lúc đó screenshot là phương án cuối, không phải phương án đầu.

| Framework | Công cụ automation | Chạy được trên Mac? |
|---|---|---|
| Electron | Playwright `_electron`, WebdriverIO + `wdio-electron-service` | Có |
| Flutter desktop | `integration_test` + `flutter drive` | Có (target macOS/Linux; target Windows cần host Windows) |
| Tauri | WebDriver qua `tauri-driver` | Chỉ Linux/Windows — **không** hỗ trợ macOS |
| .NET MAUI / WinUI 3 | Appium Windows Driver, FlaUI | Không — bắt buộc host Windows |

---

## 10.4. Container `linux/amd64` trên chính Mac — $0, 15 phút

Docker Desktop / OrbStack / colima trên Mac chạy được container `linux/amd64` qua Rosetta for Linux: bạn có **userspace x86-64 thật** ngay trên MacBook.

### 10.4.1. Cảnh báo phải đọc TRƯỚC khi tin kết quả

`--platform linux/amd64` cho bạn **userspace x86-64 chạy trên KERNEL arm64**. Cụ thể:

- `uname -m` sẽ trả về `x86_64` — **nó nói dối**. Kernel thật là arm64.
- Mọi hành vi syscall, kernel version, kernel module, ioctl là **arm64**.
- **Không có AVX/AVX2/AVX-512** — Rosetta for Linux không dịch chúng. Binary dùng AVX sẽ `SIGILL`.
- **Không chạy được binary 32-bit x86**.
- Container **chia sẻ kernel với host** → không phải security boundary (xem §10.12).

**Nó bắt được:** dependency thiếu, `.deb` không cài được, app không khởi động, crash lúc start, sai đường dẫn, packaging hỏng, thiếu font, layout cơ bản.
**Nó KHÔNG bắt được:** bug kernel/driver, bug phụ thuộc AVX, app 32-bit, tray icon (không có DE), portal, Wayland, DPI hệ thống, và bất kỳ số đo hiệu năng nào.

### 10.4.2. Script (đã sửa: `--shm-size`, D-Bus session, exit code thật, output ghi được)

```bash
mkdir -p runs
cat > runs/smoke-linux.sh <<'SCRIPT'
#!/bin/bash
# Chạy BÊN TRONG container. /artifacts = input read-only, /out = output read-write.
set -uo pipefail
export DEBIAN_FRONTEND=noninteractive

apt-get update -qq
apt-get install -y -qq --no-install-recommends \
  xvfb x11-utils imagemagick openbox dbus-x11 \
  libfuse2t64 fonts-noto-core ca-certificates
# Nếu UI của app có tiếng Trung/Nhật/Hàn, thêm: fonts-noto-cjk
# (fonts-noto-core đã đủ cho tiếng Việt có dấu)

if ! apt-get install -y -qq /artifacts/*.deb; then
  echo "verdict=INSTALL_FAILED"; exit 10
fi

Xvfb :99 -screen 0 1440x900x24 &
export DISPLAY=:99
sleep 1

# BẮT BUỘC bọc trong dbus-run-session: không có session bus thì nhiều app
# GTK/Electron treo lúc khởi động hoặc mất tray icon.
dbus-run-session -- bash -c '
  openbox &
  sleep 2
  myapp --no-sandbox --disable-gpu >/out/app.log 2>&1 &
  APP_PID=$!
  sleep 12
  if kill -0 "$APP_PID" 2>/dev/null; then
    import -window root /out/shot.png
    echo "verdict=RUNNING (app còn sống sau 12s, ảnh ở /out/shot.png)"
    kill "$APP_PID" 2>/dev/null
    exit 0
  fi
  # exit code của APP, không phải của import — đây là dòng dùng làm verdict
  wait "$APP_PID"; RC=$?
  echo "verdict=CRASHED exit_code=$RC (xem /out/app.log)"
  exit 1
'
SCRIPT
chmod +x runs/smoke-linux.sh
```

```bash
# --shm-size=2g là BẮT BUỘC: /dev/shm mặc định của Docker là 64MB,
# Chromium/Electron sẽ crash và bạn sẽ tưởng app của mình hỏng.
docker run --rm --platform linux/amd64 --shm-size=2g \
  -v "$PWD/dist:/artifacts:ro" \
  -v "$PWD/runs:/out" \
  ubuntu:24.04 bash /out/smoke-linux.sh

echo "docker exit code = $?"
open runs/shot.png
```

Bật Rosetta cho Docker Desktop: Settings → General → "Use Rosetta for x86/amd64 emulation". OrbStack bật sẵn mặc định.

**So sánh hai kiến trúc trong 30 giây** — cách nhanh nhất phát hiện bug phụ thuộc kiến trúc:

```bash
for ARCH in amd64 arm64; do
  docker run --rm --platform "linux/$ARCH" --shm-size=2g \
    -v "$PWD/dist:/artifacts:ro" -v "$PWD/runs:/out" \
    ubuntu:24.04 bash /out/smoke-linux.sh | tail -1 | sed "s/^/[$ARCH] /"
done
```

### 10.4.3. Apple `container` — lựa chọn thay thế Docker Desktop

Apple có runtime container Linux first-party, **Apache-2.0**, mỗi container một micro-VM riêng trên Virtualization.framework. Phiên bản mới nhất **v1.2.2 (08/08/2026)** — đã qua 1.0 nhưng repo vẫn ghi rõ minor version có thể có breaking change.

```bash
brew install --cask container      # hoặc tải .pkg từ github.com/apple/container/releases
container system start

container run --rm --arch amd64 \
  --volume "$PWD/dist:/artifacts:ro" \
  --volume "$PWD/runs:/out" \
  ubuntu:24.04 bash /out/smoke-linux.sh
```

| | Đánh giá |
|---|---|
| **Được** | First-party Apple, Apache-2.0, có `--arch amd64` chạy qua Rosetta (xác nhận trực tiếp trong `docs/multiplatform-images.md`), isolation mạnh hơn (VM riêng cho mỗi container), không cần license thương mại như Docker Desktop |
| **Mất** | **Yêu cầu macOS 26** để đủ tính năng (macOS 15 chạy được nhưng container không nói chuyện được với nhau và có bug subnet làm mất mạng); chưa có GUI; không có Compose first-party; VM-per-container tốn RAM hơn shared-VM khi chạy nhiều container; tương thích Docker socket API kém OrbStack |
| **Boot time** | Apple nói "comparable to containers running in a shared VM" nhưng **không công bố con số** — đừng trích số cụ thể. Độ tin cậy: Thấp |
| **`--shm-size`** | Cờ CLI của `container` bám sát Docker nhưng **không phải 1:1** — chạy `container run --help` để đối chiếu trước khi copy nguyên script Docker. Độ tin cậy tên cờ: Trung bình |

**Kết luận thực dụng:** nếu bạn đang ở macOS 26 và chỉ cần chạy container Linux, `container` là lựa chọn hợp lý. Nếu team còn máy macOS 15 hoặc bạn phụ thuộc Compose/Docker socket, ở lại OrbStack.

---

## 10.5. GitHub Actions — build + smoke test + Defender scan

### 10.5.1. Chi phí thật (đọc trước khi commit workflow)

Bảng giá per-minute hiện hành (đọc từ tài liệu billing của GitHub, 17/08/2026):

| Runner | $/phút |
|---|---|
| Linux 1-core (`ubuntu-slim`) | $0.002 |
| Linux 2-core x64 | $0.006 |
| **Linux 2-core arm64** | **$0.005** (rẻ hơn x64) |
| Windows 2-core x64 | $0.010 |
| Windows 2-core arm64 | $0.010 (bằng giá x64) |
| macOS 3-4 core | $0.062 |

**Free tier cho private repo:** Free 2.000 phút/tháng, Pro 3.000, Team 3.000, Enterprise Cloud 50.000.

**Về hệ số nhân:** mô hình cũ (1x Linux / 2x Windows / 10x macOS) đã được thay bằng bảng giá per-minute. Tính theo tỷ lệ giá thực tế so với Linux 2-core: **Windows ≈ 1,67x**, **macOS ≈ 10,3x**. Nếu trang billing của tài khoản bạn vẫn hiển thị quota theo hệ số cũ (2x/10x), dùng con số đó để tính an toàn — **tự kiểm tra trên tài khoản của bạn**, đây là chỗ hai mô hình có thể còn lệch.

Ví dụ cụ thể với workflow dưới đây (Linux ~8 phút + Windows ~12 phút mỗi lần push):

| | Private repo, GitHub Free | Public repo |
|---|---|---|
| Chi phí mỗi lần chạy | 8×$0.006 + 12×$0.010 = **$0.168** | **$0** |
| Quy đổi Linux-equivalent | 8 + 12×1,67 ≈ **28 phút** | không tính quota |
| Số lần chạy vừa 2.000 phút free | **~71 lần/tháng** | không giới hạn quota |

**Public repo thì standard GitHub-hosted runner là miễn phí** — nhưng "miễn phí" ≠ "unlimited" tuyệt đối: vẫn có giới hạn concurrency và job timeout, và **larger runner trên public repo vẫn tính tiền**.

### 10.5.2. Workflow (đã sửa: `timeout-minutes`, độ phân giải thật, D-Bus, exit code, sample consent)

```yaml
name: smoke
on: [push, workflow_dispatch]

jobs:
  linux:
    runs-on: ubuntu-24.04
    timeout-minutes: 20          # BẮT BUỘC: app GUI treo sẽ chạy tới mặc định 6 GIỜ
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: npm ci
      - run: npx electron-builder --linux AppImage deb

      - name: Cài công cụ ảo hoá màn hình
        run: |
          sudo apt-get update
          sudo apt-get install -y xvfb x11-utils imagemagick openbox \
                                  libfuse2t64 fonts-noto-core dbus-x11
          # thêm fonts-noto-cjk nếu UI có tiếng Trung/Nhật/Hàn

      - name: Kiểm tra .deb cài được không
        run: sudo apt-get install -y ./dist/*.deb

      - name: Chạy thử và chụp màn hình
        timeout-minutes: 5
        env:
          DISPLAY: ":99"
        run: |
          set -uo pipefail
          APP=$(ls dist/*.AppImage | head -1)
          chmod +x "$APP"
          export APP
          Xvfb :99 -screen 0 1440x900x24 &
          sleep 1
          dbus-run-session -- bash -c '
            openbox & sleep 2
            # runner không có FUSE → BẮT BUỘC --appimage-extract-and-run
            "$APP" --appimage-extract-and-run --no-sandbox --disable-gpu \
              > /tmp/app.log 2>&1 &
            PID=$!
            sleep 15
            if kill -0 "$PID" 2>/dev/null; then
              import -window root /tmp/shot.png
              echo "verdict=RUNNING"; kill "$PID"; exit 0
            fi
            wait "$PID"; echo "verdict=CRASHED exit_code=$?"; exit 1
          '

      - name: Chụp thêm ở mức scale 150% (XEM CẢNH BÁO §10.5.3)
        timeout-minutes: 5
        continue-on-error: true
        env:
          DISPLAY: ":99"
          GDK_DPI_SCALE: "1.5"
        run: |
          set -uo pipefail
          APP=$(ls dist/*.AppImage | head -1); export APP
          dbus-run-session -- bash -c '
            "$APP" --appimage-extract-and-run --no-sandbox --disable-gpu \
              --force-device-scale-factor=1.5 > /tmp/app150.log 2>&1 &
            PID=$!; sleep 15
            kill -0 "$PID" 2>/dev/null && import -window root /tmp/shot-150.png
            kill "$PID" 2>/dev/null || true
          '

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: linux-smoke
          path: |
            /tmp/shot*.png
            /tmp/app*.log

  windows:
    runs-on: windows-latest      # = Windows SERVER 2025 (nhãn tương đương windows-2025)
    timeout-minutes: 25
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: npm ci
      - run: npx electron-builder --win nsis

      - name: Kiểm tra chữ ký Authenticode
        shell: powershell
        run: Get-ChildItem dist\*.exe | Get-AuthenticodeSignature | Format-List

      - name: BẬT LẠI Defender rồi quét installer
        shell: powershell
        continue-on-error: true
        run: |
          # Runner image của GitHub CHỦ ĐỘNG TẮT Defender vì hiệu năng
          # (images/windows/scripts/build/Configure-WindowsDefender.ps1 trong
          #  actions/runner-images: DisableRealtimeMonitoring, DisableScriptScanning,
          #  ExclusionPath C:\ và D:\, và set ForceDefenderPassiveMode nếu có ATP path).
          # Không bật lại thì verdict "sạch" là VÔ NGHĨA.
          Set-MpPreference -DisableRealtimeMonitoring $false
          Set-MpPreference -ExclusionPath @()
          Set-MpPreference -MAPSReporting Advanced
          # QUAN TRỌNG: SendAllSamples sẽ UPLOAD installer CHƯA RELEASE của bạn
          # lên Microsoft. Với sản phẩm thương mại / có NDA, dùng NeverSend.
          Set-MpPreference -SubmitSamplesConsent NeverSend
          Update-MpSignature
          $exe = (Get-ChildItem dist\*.exe | Select-Object -First 1).FullName
          & "$env:ProgramFiles\Windows Defender\MpCmdRun.exe" -Scan -ScanType 3 -File $exe
          Get-MpThreatDetection | Format-List

      - name: Cài im lặng, chạy, chụp màn hình
        shell: powershell
        continue-on-error: true
        run: |
          $exe = (Get-ChildItem dist\*.exe | Select-Object -First 1).FullName
          Start-Process -FilePath $exe -ArgumentList "/S" -Wait   # NSIS silent
          Start-Sleep -Seconds 5
          $app = Get-ChildItem "$env:LOCALAPPDATA\Programs" -Recurse -Filter *.exe |
                 Select-Object -First 1
          if (-not $app) { throw "Không tìm thấy app sau khi cài" }
          $p = Start-Process $app.FullName -PassThru
          Start-Sleep -Seconds 20
          if ($p.HasExited) { throw "verdict=CRASHED exit_code=$($p.ExitCode)" }

          Add-Type -AssemblyName System.Windows.Forms,System.Drawing
          # KHÔNG hardcode 1920x1080 — desktop của runner có thể là 1024x768
          $b = [System.Windows.Forms.Screen]::PrimaryScreen.Bounds
          $bmp = New-Object Drawing.Bitmap $b.Width, $b.Height
          $gfx = [Drawing.Graphics]::FromImage($bmp)
          $gfx.CopyFromScreen($b.X, $b.Y, 0, 0, $bmp.Size)
          $bmp.Save("$env:RUNNER_TEMP\shot.png")
          $gfx.Dispose(); $bmp.Dispose()
          Stop-Process -Id $p.Id -Force

      - uses: actions/upload-artifact@v4
        if: always()
        with: { name: windows-smoke, path: "${{ runner.temp }}/shot.png" }
```

### 10.5.3. Năm cảnh báo trung thực về workflow này

1. **`windows-latest` = Windows Server 2025**, không phải Windows 11 client. Hành vi SmartScreen, Defender UI, MSIX, shell integration và một số chính sách khác nhau. (`windows-2022` vẫn còn khả dụng nếu bạn cần.)
2. **Screenshot trên Windows runner có thể ra ảnh đen** (session 0 / không có desktop tương tác). Phải thử mới biết — nếu đen, bỏ bước đó, giữ Defender scan và `Get-AuthenticodeSignature`.
3. **`--force-device-scale-factor=1.5` KHÔNG phải test DPI 150% thật.** Nó chỉ scale device pixel ratio trong renderer; **không** đổi system metrics, **không** đổi non-client area/title bar, **không** đổi font hệ thống, **không** kích hoạt `WM_DPICHANGED`, **không** ảnh hưởng native dialog/menu/tray. Dùng để bắt bug layout trong webview, **không** dùng để kết luận "app OK ở 150%".
4. **Defender verdict `!ml` (cloud ML) phụ thuộc prevalence/reputation.** Installer mới của bạn có prevalence = 0 trên máy user thật; CI không tái hiện được điều đó. Verdict "sạch" trên CI **không đảm bảo** user không bị SmartScreen chặn.
5. **Đừng lấy thời gian build trên CI làm chuẩn** để suy ra thời gian build trên máy Windows thật của user — runner đã tắt Defender và loại trừ toàn bộ `C:\`, `D:\`.

**Về `windows-11-arm`:** đã mở cho **cả public VÀ private repo** — không còn giới hạn public-only như thông tin cũ. Biến thể `windows-11-vs2026-arm` đang ở private preview. Số core: 4 cho public repo, 2 cho private repo (áp dụng chung cho ubuntu/windows).

---

## 10.6. VPS Linux x86-64 — fidelity cao nhất trên mỗi đồng cho Linux

### 10.6.1. Chọn nhà cung cấp — bối cảnh Việt Nam

**Độ tin cậy của toàn bộ bảng giá dưới đây: THẤP.** Không xác minh được bằng nguồn trực tuyến trong lần soạn này (trang giá của Vultr/Linode/Hetzner trả 403 hoặc render giá bằng JavaScript). Coi đây là ước lượng và **tự kiểm tra trên trang giá trước khi mở ví**.

| Nhà cung cấp | Cấu hình ~2 vCPU / 4 GB | Giá ước tính | Latency từ VN | Ghi chú cho khách VN |
|---|---|---|---|---|
| **Hetzner CX22** (Falkenstein/Nuremberg/Helsinki) | 2 vCPU Intel shared, 4 GB, 40 GB NVMe | ~€3,79/th + ~€0,50 IPv4 ≈ **€4,3/th (~120-130k VND)** | **230-300 ms**, vọt 350 ms+ khi đứt cáp quang biển | Rẻ nhất. **Nhưng Hetzner nổi tiếng khó với khách VN**: yêu cầu xác minh danh tính (ảnh hộ chiếu/CCCD, đôi khi selfie), hay từ chối thẻ tín dụng phát hành tại VN, ưu tiên PayPal/SEPA |
| **Hetzner Singapore** | Chỉ có dòng **CPX** (AMD shared) và **CCX** (dedicated vCPU) — **CX/CAX là EU-only** | CPX21 (3 vCPU, 4 GB) ~**€8-10/th** do phụ phí vị trí | 25-50 ms | Vẫn vướng rào cản thanh toán/KYC như trên |
| **Vultr Singapore** | Cloud Compute Regular | ~**$20/th** (High Frequency ~$24) | **25-50 ms** | Thẻ VN dễ qua nhất trong nhóm quốc tế |
| **DigitalOcean Singapore** | Basic Droplet | ~**$24/th** (Premium Intel/AMD ~$28) | 25-50 ms | Thẻ VN dễ qua |
| **Akamai/Linode Singapore** | 4 GB shared | ~**$24/th** | 25-50 ms | Có thể có phụ phí vùng nhẹ |
| **Vietnix / AZDIGI** | 2 vCPU / 4 GB | ~**200-450k VND/th** | **<10-20 ms** | Thanh toán VND, hỗ trợ tiếng Việt |
| **VNG Cloud / FPT Cloud / Viettel IDC** | 2 vCPU / 4 GB | ~**400-800k VND/th** | <10-20 ms | Đắt hơn nhưng có hoá đơn VAT — quan trọng nếu bạn cần hạch toán |
| **Oracle Cloud Always Free** | 2 instance AMD x86 + 4 core ARM/24 GB | **$0 vĩnh viễn** | tuỳ region | Đáng thử nhưng khâu duyệt tài khoản với thẻ VN nổi tiếng khó; đừng dựa vào nó cho việc quan trọng. Độ tin cậy: Thấp |

**Khuyến nghị:**

- Latency **chỉ quan trọng khi bạn tương tác** (SSH gõ lệnh, VNC/RDP xem màn hình, `rsync` file 200 MB). Với batch build/CI thì 250 ms không ảnh hưởng gì.
- **Nếu bạn muốn xem màn hình app hoặc gõ lệnh nhiều:** chọn **Vultr hoặc DigitalOcean Singapore**. Đắt hơn Hetzner ~4-5 lần nhưng thẻ VN qua được và 25-50 ms là khác biệt cảm nhận rõ ràng.
- **Nếu chỉ chạy smoke test không người trực:** Hetzner EU rẻ nhất — nếu bạn vượt được khâu KYC.
- **Nếu cần hoá đơn VAT hoặc latency nội địa:** VPS trong nước.

Bắt đầu bằng tier nhỏ nhất (1 vCPU / 2 GB thường rẻ hơn đáng kể) — smoke test headless không cần 4 GB. Nâng cấp sau nếu chật.

### 10.6.2. Setup một lần (~20 phút) — đã sửa btrfs subvolume, DNS, FUSE, pre-bake

```bash
# ===== Trên VPS, chạy 1 lần dưới quyền root =====
apt-get update
apt-get install -y systemd-container debootstrap btrfs-progs rsync \
                   podman ufw x11vnc xvfb imagemagick openbox dbus-x11

mkdir -p /var/lib/machines /srv/in /srv/out /srv/bin

# --- BƯỚC BỊ BỎ QUÊN #1: /var/lib/machines PHẢI là btrfs ---
# `--ephemeral` chỉ nhanh (~2s) khi rootfs là SUBVOLUME btrfs.
# Nếu không, systemd âm thầm fallback sang COPY TOÀN BỘ rootfs: 30-90 giây.
if [ "$(stat -f -c %T /var/lib/machines)" != "btrfs" ]; then
  fallocate -l 20G /var/lib/machines.raw
  mkfs.btrfs -q /var/lib/machines.raw
  echo "/var/lib/machines.raw /var/lib/machines btrfs loop,compress=zstd 0 0" \
    >> /etc/fstab
  mount /var/lib/machines
fi
stat -f -c %T /var/lib/machines    # phải in ra: btrfs
```

```bash
# --- BƯỚC BỊ BỎ QUÊN #2: tạo SUBVOLUME trước khi debootstrap ---
btrfs subvolume create /var/lib/machines/ubuntu2404
debootstrap --arch=amd64 noble /var/lib/machines/ubuntu2404 \
  http://archive.ubuntu.com/ubuntu/

# --- BƯỚC BỊ BỎ QUÊN #3: PRE-BAKE dependency vào rootfs ---
# Nếu không, mỗi lần `--ephemeral` bạn phải `apt-get update` lại: 20-40 giây,
# và lời hứa "máy sạch trong 2 giây" là không có thật ở end-to-end.
cat > /var/lib/machines/ubuntu2404/etc/apt/sources.list <<'EOF'
deb http://archive.ubuntu.com/ubuntu noble main universe
deb http://archive.ubuntu.com/ubuntu noble-updates main universe
deb http://security.ubuntu.com/ubuntu noble-security main universe
EOF

systemd-nspawn -q -D /var/lib/machines/ubuntu2404 --resolv-conf=copy-host \
  /bin/bash -c '
    export DEBIAN_FRONTEND=noninteractive
    apt-get update -qq
    apt-get install -y -qq --no-install-recommends \
      xvfb x11-utils imagemagick openbox dbus-x11 libfuse2t64 \
      fonts-noto-core ca-certificates
    apt-get clean'
```

**Ubuntu 26.04 (và mọi codename mới):** `debootstrap` trên host sẽ **fail** nếu nó chưa biết codename đó. Thêm symlink script:

```bash
# TỰ VERIFY codename của Ubuntu 26.04 trước khi chạy — tài liệu này KHÔNG
# xác minh được tên codename đó, đừng tin con số/tên nào ở đây.
CODENAME=<codename-thật-của-26.04>
ln -sf /usr/share/debootstrap/scripts/gutsy \
       /usr/share/debootstrap/scripts/"$CODENAME"

btrfs subvolume create /var/lib/machines/ubuntu2604
debootstrap --arch=amd64 "$CODENAME" /var/lib/machines/ubuntu2604 \
  http://archive.ubuntu.com/ubuntu/
# rồi lặp lại bước pre-bake ở trên với sources.list dùng $CODENAME
```

**Fedora: đừng dùng `dnf --installroot` từ host Ubuntu** (thiếu dnf, thiếu GPG key, rất khó chịu). Dùng podman:

```bash
podman run --rm --shm-size=2g \
  -v /srv/in:/artifacts:ro -v /srv/out:/out:z \
  fedora:latest bash -c '
    dnf install -y -q xorg-x11-server-Xvfb ImageMagick openbox dbus-x11 \
                      google-noto-sans-fonts fuse-libs
    dnf install -y -q /artifacts/*.rpm || { echo verdict=INSTALL_FAILED; exit 10; }
    Xvfb :99 -screen 0 1440x900x24 & export DISPLAY=:99; sleep 1
    dbus-run-session -- bash -c "
      openbox & sleep 2
      myapp --no-sandbox --disable-gpu >/out/app-fedora.log 2>&1 &
      PID=\$!; sleep 12
      if kill -0 \$PID 2>/dev/null; then
        import -window root /out/shot-fedora.png; echo verdict=RUNNING; kill \$PID; exit 0
      fi
      wait \$PID; echo \"verdict=CRASHED exit_code=\$?\"; exit 1
    "'
```

### 10.6.3. Script chạy mỗi lần test

```bash
# ===== Trên VPS, tạo 1 lần =====
cat > /srv/bin/smoke.sh <<'SCRIPT'
#!/bin/bash
# Chạy trong nspawn container. /artifacts = ro, /out = rw.
set -uo pipefail
DEB=$(ls /artifacts/*.deb 2>/dev/null | head -1)
[ -n "$DEB" ] || { echo "verdict=NO_ARTIFACT"; exit 20; }

# Dependency đã pre-bake → không cần apt-get update. Nếu .deb có dep thiếu,
# bỏ comment dòng dưới (mất thêm 20-40s):
# apt-get update -qq
apt-get install -y -qq "$DEB" || { echo "verdict=INSTALL_FAILED"; exit 10; }

Xvfb :99 -screen 0 1440x900x24 &
export DISPLAY=:99
sleep 1

dbus-run-session -- bash -c '
  openbox & sleep 2
  myapp --no-sandbox --disable-gpu > "/out/app-$DISTRO.log" 2>&1 &
  PID=$!
  sleep 12
  if kill -0 "$PID" 2>/dev/null; then
    import -window root "/out/shot-$DISTRO.png"
    echo "verdict=RUNNING distro=$DISTRO"; kill "$PID"; exit 0
  fi
  wait "$PID"; echo "verdict=CRASHED distro=$DISTRO exit_code=$?"; exit 1
'
SCRIPT
chmod +x /srv/bin/smoke.sh
```

```bash
# ===== Từ Mac =====
rsync -az --delete dist/ root@<ip>:/srv/in/

ssh root@<ip> 'for D in ubuntu2404 ubuntu2604; do
  systemd-nspawn --ephemeral -q \
    -D "/var/lib/machines/$D" \
    --resolv-conf=copy-host \
    --setenv=DISTRO="$D" \
    --bind-ro=/srv/in:/artifacts \
    --bind=/srv/out:/out \
    --bind=/srv/bin:/bin-host:ro \
    --bind=/dev/fuse \
    --property=DeviceAllow="/dev/fuse rwm" \
    /bin/bash /bin-host/smoke.sh
done'

mkdir -p runs && rsync -az root@<ip>:/srv/out/ runs/
open runs/shot-ubuntu2404.png
```

Bốn sửa lỗi quan trọng so với bản cũ, đừng bỏ:

| Cờ | Vì sao bắt buộc |
|---|---|
| `--bind=/srv/out:/out` | Bản cũ dùng `--bind-ro=/artifacts` rồi ghi `shot.png` vào đó → **fail chắc chắn**. Input ro và output rw phải là hai bind khác nhau |
| `--resolv-conf=copy-host` | Không có nó thì container **không có DNS**, mọi `apt-get` fail |
| `--bind=/dev/fuse` + `--property=DeviceAllow` | Không có nó thì `.AppImage` không mount được (fallback: `--appimage-extract-and-run`) |
| `--setenv=DISTRO=...` | Để mỗi distro ghi ra file riêng, không đè lên nhau |

### 10.6.4. "Nhìn tận mắt" — không cần xrdp, không cần XFCE

Bản cũ khuyên cài `xrdp` + XFCE. Không cần, và còn sai môi trường: bạn sẽ nhìn một desktop XFCE khác hẳn cái Xvfb mà test đang chạy.

```bash
# ===== Trên VPS: chạy Xvfb + app ở SESSION KHÔNG ephemeral =====
# (x11vnc trên host không nhìn thấy được X server bên trong container ephemeral —
#  muốn xem thì chạy trực tiếp trên host, hoặc bỏ --ephemeral cho phiên debug.)
Xvfb :99 -screen 0 1440x900x24 &
export DISPLAY=:99
openbox &
myapp --no-sandbox --disable-gpu &

x11vnc -display :99 -forever -localhost -nopw -shared &
```

```bash
# ===== Trên Mac =====
ssh -N -L 5900:localhost:5900 root@<ip>
# Cửa sổ khác: Finder → Cmd+K → vnc://localhost:5900
# (macOS có sẵn Screen Sharing, không cần cài gì)
```

`-localhost` là bắt buộc: nó khiến x11vnc **chỉ** lắng nghe trên loopback, nên VNC không bao giờ lộ ra internet. Xem §10.12.

---

## 10.7. Windows x64 THẬT tính theo GIỜ — đường rẻ nhất cho nhu cầu tương tác

Đây là lựa chọn bị bỏ sót lớn nhất, và bị gộp nhầm vào rổ "per-user-per-month" của Windows 365/Shadow. Thực tế: **có VM Windows tính theo giờ, license gộp trong giá, API spin-up/tear-down/snapshot đầy đủ, hợp pháp 100%, không dính QMTH.**

### 10.7.1. Ràng buộc licensing phải biết trước

Cái bạn thuê được theo giờ là **Windows Server 2022/2025**, **không phải Windows 11 client**. EC2/Azure VM/GCE không cho bạn chạy Windows 11 desktop hợp lệ trừ khi qua Windows 365 / Azure Virtual Desktop (cần entitlement M365 E3/E5/Business Premium), hoặc nhà cung cấp QMTH, hoặc dedicated host BYOL.

| Nhu cầu của bạn | Windows Server có đủ không? |
|---|---|
| Kiểm chứng Win32/WinUI cơ bản, installer NSIS/Inno, Authenticode, Defender scan, native `.node` addon trên x64 thật | **Đủ** |
| Shell integration của Win11, MSIX/packaged app, driver, UI scaling kiểu Win11, SmartScreen đúng như user thấy | **Không đủ** — ràng buộc licensing là có thật, phải dùng máy thật (§10.10) hoặc Windows 365 |

### 10.7.2. Giá (độ tin cậy: Trung bình-Cao cho Azure/AWS, Thấp cho GCP)

| VM | Linux $/h | **Windows $/h** | 20 giờ/tháng |
|---|---|---|---|
| AWS t3.medium (2 vCPU / 4 GB) | $0,0416 | **~$0,08** | ~$1,6 |
| Azure B2s (2 vCPU / 4 GB) | $0,0416 | **~$0,09-0,10** | ~$2 |
| Azure B2ms (2 vCPU / 8 GB) | $0,0832 | **~$0,14-0,15** | ~$3 |
| GCP e2-medium + Windows Server license | — | ~$0,11 (**độ tin cậy Thấp**) | ~$2,2 |

Cộng disk OS persistent: ~$5-10/tháng cho 128 GB, hoặc **~$0 nếu bạn snapshot rồi xoá cả VM lẫn disk** (snapshot ~$1-3/tháng). **Tổng thực tế ~$4-12/tháng.**

**So với Windows 365 Business** (Basic $28 / Standard $36 / Premium $56 mỗi user/tháng): ở mức ~20 giờ/tháng, VM theo giờ **rẻ hơn 3-7 lần**. Điểm hoà vốn rơi vào khoảng **250-300 giờ/tháng** — tức gần như chạy 24/7 mới khiến Cloud PC đáng giá. Đổi lại, Windows 365 cho bạn Windows 11 **client** hợp lệ và zero ops.

### 10.7.3. Azure — spin up, test, tear down

```bash
brew install azure-cli
az login

RG=wintest
LOC=southeastasia          # Singapore: ~25-50 ms từ VN
MYIP=$(curl -s https://ifconfig.me)

az group create -n "$RG" -l "$LOC"

# Verify URN image trước (URN thay đổi theo thời gian):
#   az vm image list --publisher MicrosoftWindowsServer --offer WindowsServer \
#     --all --output table | head
az vm create -g "$RG" -n win1 \
  --image MicrosoftWindowsServer:WindowsServer:2025-datacenter-azure-edition:latest \
  --size Standard_B2s \
  --admin-username azureuser \
  --admin-password '<MatKhauRatManh#2026>' \
  --public-ip-sku Standard \
  --nsg-rule NONE

# Mở RDP CHỈ cho IP của bạn — đừng bao giờ mở 3389 ra 0.0.0.0/0
az network nsg rule create -g "$RG" --nsg-name win1NSG -n allow-rdp-me \
  --priority 1000 --access Allow --protocol Tcp \
  --destination-port-ranges 3389 --source-address-prefixes "$MYIP"
```

```bash
# Chạy lệnh từ xa, KHÔNG cần RDP — đây là thứ biến nó thành CI target
az vm run-command invoke -g "$RG" -n win1 \
  --command-id RunPowerShellScript \
  --scripts "Get-AuthenticodeSignature 'C:\dist\MyApp-Setup.exe' | Format-List"

# Snapshot ổ OS để lần sau dựng lại từ máy đã cấu hình sẵn
OSDISK=$(az vm show -g "$RG" -n win1 --query "storageProfile.osDisk.name" -o tsv)
az snapshot create -g "$RG" -n win1-clean --source "$OSDISK"

# Tắt tính tiền compute (giữ disk):
az vm deallocate -g "$RG" -n win1

# Xoá sạch mọi thứ (dừng tính tiền hoàn toàn):
az group delete -n "$RG" --yes --no-wait
```

Đẩy file lên bằng SCP (Windows Server 2025 có sẵn OpenSSH Server, nhưng phải bật) hoặc đơn giản nhất là mount ổ qua RDP client, hoặc `az storage blob upload` rồi `run-command` tải về.

### 10.7.4. AWS — tương đương

```bash
AMI=$(aws ssm get-parameter \
  --name /aws/service/ami-windows-latest/Windows_Server-2025-English-Full-Base \
  --query 'Parameter.Value' --output text)

IID=$(aws ec2 run-instances \
  --image-id "$AMI" --instance-type t3.medium \
  --key-name my-key --security-group-ids sg-xxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=wintest}]' \
  --query 'Instances[0].InstanceId' --output text)

aws ec2 wait instance-running --instance-ids "$IID"
aws ec2 get-password-data --instance-id "$IID" --priv-launch-key ~/.ssh/my-key.pem

# Đóng băng thành AMI để lần sau dựng lại trong 2 phút:
aws ec2 create-image --instance-id "$IID" --name "wintest-clean-$(date +%s)"

# Tear down:
aws ec2 terminate-instances --instance-ids "$IID"
```

---

## 10.8. Chạy Windows 11 ARM64 miễn phí trên Mac bằng UTM (1-2 giờ)

Đây là cách $0 tốt nhất để **nhìn tận mắt** app của bạn trên Windows.

1. **Cài UTM**: `brew install --cask utm` (hoặc tải từ mac.getutm.app — miễn phí; bản Mac App Store $9.99 chỉ để ủng hộ).
2. **Lấy ISO Windows 11 ARM64**:
   - Bản chính chủ có key: `microsoft.com/software-download/windows11arm64` (25H2, ~5-7 GB, đa phiên bản).
   - **Hoặc — cách không cần key, hợp pháp, dùng thử 90 ngày:** Windows 11 **Enterprise ARM64 Evaluation** từ Microsoft Evaluation Center (`microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise`). Trang ghi rõ có bản **Arm64 ISO**, đánh giá **90 ngày**, và "**A product key is not required for this software**" — chỉ cần đăng nhập bằng tài khoản Microsoft. Hết 90 ngày phải dựng lại VM; không convert được sang production. **Đây là cách tốt nhất để thử trước khi bỏ tiền mua Windows 11 Pro retail.**
   - **Phương án thứ hai:** Windows Insider Preview ARM64 — có bản **VHDX** dùng trực tiếp cho VM, miễn phí, chỉ cần tài khoản đã đăng ký Windows Insider. Build có ngày hết hạn và là license pre-release chỉ để đánh giá/test. (Độ tin cậy: Trung bình — không xác minh lại được trang tải trong lần soạn này.)
   - Verify SHA-256 trước khi dùng.
3. **Tải virtio-win driver ARM64**: bản Fedora build, hoặc bộ ISO ARM64 rút gọn (`qemus/virtiso-arm`).
4. **Tạo VM trong UTM**: chọn **Virtualize** → Windows → gắn ISO Windows + tick "Install drivers and SPICE tools". Cấu hình khuyến nghị: **4 vCPU, 8-12 GB RAM**, đĩa trên SSD nhanh. UTM 4.7.5 đóng gói sẵn flow QEMU+HVF+TPM này.
5. **Trong lúc cài**, khi Setup không thấy ổ đĩa → Load driver → trỏ tới `viostor\w11\ARM64`.
6. **Sau khi cài**: chạy `spice-tools` installer để có clipboard + shared folder. Tắt visual effects.
7. **Chia sẻ file từ Mac**: UTM có shared directory (SPICE WebDAV). Nếu bí, cách chắc chắn nhất là bật **File Sharing** trên macOS rồi từ Windows guest mount `\\10.0.2.2\<share>` (hoặc IP của Mac trên mạng NAT).

**Cấu hình app khi test trong VM (bắt buộc):** không có GPU acceleration → Electron/Chromium hay tự tắt renderer hoặc crash. Chạy với `--disable-gpu` hoặc `--use-gl=swiftshader`.

### Cái bạn sẽ KHÔNG test được trong VM này — dán checklist này lên tường

- Driver kernel-mode (USB token chữ ký số, VPN client cũ, ASIO audio, print driver, anti-cheat)
- DirectX 12, Vulkan, compute shader hiện đại (không hypervisor nào trên Mac có DX12)
- Hiệu năng bất kỳ (AVX2 qua emulation trên M2 = **0.66x** so với SSE2 — ngược dấu với x64 thật)
- App 32-bit và app x64 có helper process 32-bit
- WSL2, Hyper-V, Windows Sandbox, VBS/HVCI/Memory Integrity, Docker Desktop, Android Emulator (không có nested virt; và **M1/M2 vĩnh viễn không có**)
- Bug do memory ordering (ARM yếu hơn x86 TSO → bạn sẽ thấy bug ma, và bug x86 thật bị Prism che)
- Bug native `.node` addon / DLL x64 (fail theo cách khác, hoặc bị Prism che)

**Nếu bất kỳ dòng nào ở trên ảnh hưởng app của bạn: không có phần mềm nào cứu bạn.** Đi thẳng xuống §10.10.

---

## 10.9. Windows Sandbox — clean room 10 giây cho smoke test installer

Trên **bất kỳ máy Windows 11 Pro/Enterprise x64 nào** (mini PC của bạn, VM cloud ở §10.7, máy của bạn bè), Windows Sandbox đơn giản hơn nhiều so với vòng Hyper-V checkpoint: một desktop Windows sạch tinh, tự xoá sạch khi đóng cửa sổ.

**Không có trên Windows 11 Home.** Kiểm tra edition trước (xem §10.10.1).

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName "Containers-DisposableClientVM" -All
# Khởi động lại máy
```

```xml
<!-- smoke.wsb — double-click file này là xong -->
<Configuration>
  <MappedFolders>
    <MappedFolder>
      <HostFolder>C:\dist</HostFolder>
      <SandboxFolder>C:\dist</SandboxFolder>
      <ReadOnly>true</ReadOnly>
    </MappedFolder>
  </MappedFolders>
  <LogonCommand>
    <Command>powershell -ExecutionPolicy Bypass -Command "Start-Process 'C:\dist\MyApp-Setup.exe' -ArgumentList '/S' -Wait; Start-Sleep 5; Start-Process (Get-ChildItem $env:LOCALAPPDATA\Programs -Recurse -Filter *.exe | Select -First 1).FullName"</Command>
  </LogonCommand>
  <MemoryInMB>4096</MemoryInMB>
</Configuration>
```

**Bắt được:** installer chạy trên máy chưa từng có app của bạn, thiếu VC++ redistributable, thiếu .NET runtime, đường dẫn hardcode, registry key mồ côi, per-user vs per-machine install.
**KHÔNG bắt được:** SmartScreen đúng như user thấy (Sandbox có chính sách riêng), Defender với cấu hình mặc định của user, và mọi thứ cần persistence giữa các lần chạy.

---

## 10.10. Mini PC N100 — câu trả lời trung thực nhất cho nhiều trường hợp

Mua một mini PC Intel N100 (16 GB/512 GB, ~$240-270 quốc tế, ~5-6 triệu VND ở VN). Cân nhắc thêm hai phương án thường tốt hơn ở VN:

| Phương án | Ưu | Nhược |
|---|---|---|
| Mini PC N100 mới | Nhỏ, mới, ít điện (~7-15 W idle, ~25 W tải → **~15-35k VND điện/tháng**) | Thường ship **Windows 11 Home** — xem cảnh báo bên dưới |
| **Laptop Windows cũ 2-4 triệu VND** | Có sẵn màn hình + bàn phím, và **pin = UPS tích hợp** (rất đáng giá ở VN), thường kèm license OEM | Cồng kềnh hơn, pin chai |
| **Dell OptiPlex / HP EliteDesk micro refurb ~1,5-3 triệu** | Mạnh hơn N100 đáng kể, thường có Windows Pro OEM sẵn | Ồn hơn, tốn điện hơn |

### 10.10.1. KIỂM TRA EDITION TRƯỚC KHI MUA — đây là chỗ kế hoạch cũ sụp

`Enable-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V` **KHÔNG có trên Windows 11 Home**, mà phần lớn mini PC $250 ship Home. Windows Sandbox cũng không có. Toàn bộ câu chuyện "reset bằng Hyper-V checkpoint" sụp ở đây.

```powershell
# Chạy ngay khi máy về, TRƯỚC khi lên kế hoạch gì
Get-ComputerInfo -Property WindowsProductName, WindowsEditionId, OsArchitecture
# WindowsEditionId = "Core"/"CoreSingleLanguage" → HOME, không có Hyper-V/Sandbox
# WindowsEditionId = "Professional"/"Enterprise"  → OK
```

**Nếu máy là Home, ba đường ra:**

| Đường | Chi phí | Đánh giá |
|---|---|---|
| Nâng lên Pro (Microsoft Store, giữ nguyên dữ liệu) | ~$99 quốc tế / VN thường ~2,3-2,9 triệu VND (**độ tin cậy Thấp — tự kiểm tra giá**) | Đơn giản nhất, mở khoá cả Hyper-V lẫn Windows Sandbox. Cộng con số này vào giá mua ngay từ đầu |
| **Image restore thay cho checkpoint**: Macrium Reflect / Veeam Agent Free, chụp image ổ C: lúc sạch, restore bằng USB boot | $0 | Chạy trên Home. Nhược: restore mất 5-15 phút thay vì 15 giây, và phải cắm USB — không automate từ xa được |
| Ổ SSD thứ hai với bản Windows sạch, đổi boot order | ~500k-1 triệu VND | Thô nhưng chắc, và cho bạn hai môi trường độc lập |

### 10.10.2. Biến nó thành test rig điều khiển được từ Mac

```powershell
# ===== Trên mini PC =====
# 1. Bật OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic

# 2. BƯỚC BỊ BỎ QUÊN: default shell của OpenSSH trên Windows là cmd.exe.
#    Không đổi thì mọi lệnh PowerShell qua ssh sẽ FAIL.
New-Item -Path "HKLM:\SOFTWARE\OpenSSH" -Force | Out-Null
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" `
  -Name DefaultShell `
  -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -PropertyType String -Force
Restart-Service sshd

# 3. Key-only auth (xem §10.12) — dán public key của Mac vào:
#    C:\Users\<user>\.ssh\authorized_keys
#    (nếu user là admin: C:\ProgramData\ssh\administrators_authorized_keys)
```

```powershell
# ===== CHỈ khi máy là Windows 11 Pro/Enterprise =====
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
# sau khi tạo VM "TestWin11" bên trong:
Checkpoint-VM -Name "TestWin11" -SnapshotName "clean"
# reset về máy sạch trước mỗi lần test installer (~15 giây):
Restore-VMCheckpoint -VMName "TestWin11" -Name "clean" -Confirm:$false
```

```bash
# ===== Trên Mac =====
brew install --cask tailscale
# Cài Tailscale ở CẢ HAI máy → truy cập từ bất cứ đâu, không cần port forward,
# không mở cổng nào ra internet.

# ĐƯỜNG DẪN PHẢI LÀ ĐƯỜNG DẪN WINDOWS — không phải /Users/you/
scp dist/MyApp-Setup.exe minipc:C:/Users/dat/Desktop/

ssh minipc 'Start-Process -FilePath "C:\Users\dat\Desktop\MyApp-Setup.exe" -ArgumentList "/S" -Wait'
ssh minipc 'Get-AuthenticodeSignature "C:\Users\dat\Desktop\MyApp-Setup.exe" | Format-List'

# Nhìn tận mắt: RDP bằng app "Windows App" của Microsoft trên Mac
# (bản kế nhiệm của Microsoft Remote Desktop), kết nối qua địa chỉ Tailscale.
```

Bạn vừa có: **Windows x64 thật + reset về máy sạch + điều khiển từ terminal trên Mac + nhìn được bằng mắt.** Với ~$250 và một buổi chiều.

Bật **Wake-on-LAN** để không phải chạy máy 24/7 — điện không đáng kể nhưng nhiệt và tuổi thọ quạt thì có.

---

## 10.11. Quy trình tái hiện bug user báo

Đây là lý do phổ biến nhất khiến một dev Mac-only cần Windows, nhưng không ai viết ra công thức. Đây là công thức:

**Bước 1 — Thu thập (làm trước, đừng đoán).** Yêu cầu user 6 thứ, đúng thứ tự này:

```text
1. winver                → build number chính xác (vd 26100.xxxx)
2. Cài đặt → Hệ thống → Giới thiệu → "Loại hệ thống"  → x64 hay ARM
3. Phiên bản app (Help → About)
4. File log:  %APPDATA%\<app>\logs\main.log
5. Tỷ lệ hiển thị (Cài đặt → Màn hình → Tỷ lệ)  → 100/125/150/175%
6. Video 10 giây hoặc GIF thao tác dẫn tới lỗi
```

Nếu đã có Sentry (§10.2), 1-3 đã có sẵn trong event; chỉ cần hỏi 4-6.

**Bước 2 — Bật logging chi tiết cho lần tái hiện tiếp theo:**

```powershell
# Electron/Chromium: log đầy đủ ra stderr + file
& "C:\Users\<user>\AppData\Local\Programs\MyApp\MyApp.exe" `
    --enable-logging=file --log-file=C:\temp\chrome_debug.log --v=1
```

**Bước 3 — Bật WER LocalDumps để lần crash sau có dump thật:**

```powershell
$p = "HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps\MyApp.exe"
New-Item -Path $p -Force | Out-Null
New-ItemProperty -Path $p -Name DumpFolder -Value "C:\crashdumps" -PropertyType ExpandString -Force
New-ItemProperty -Path $p -Name DumpCount  -Value 10 -PropertyType DWord -Force
New-ItemProperty -Path $p -Name DumpType   -Value 2  -PropertyType DWord -Force  # 2 = full dump
```

**Bước 4 — Chọn môi trường tái hiện theo phân loại bug:**

| Triệu chứng | Tái hiện ở đâu | Vì sao |
|---|---|---|
| Layout vỡ, chữ bị cắt, sai locale | **Ngay trên Mac** bằng Playwright + pseudolocale (§10.3) | Không cần Windows chút nào |
| Installer bị Defender/SmartScreen chặn | GitHub Actions có bật lại Defender (§10.5) **+** máy Windows thật | Cloud ML verdict phụ thuộc prevalence, CI không tái hiện đủ |
| App không khởi động, thiếu runtime/DLL | **Windows Sandbox** (§10.9) — máy sạch tuyệt đối | Máy dev của bạn đã có sẵn mọi runtime |
| Crash trong native `.node` addon, bug x64 | **VM cloud theo giờ** (§10.7) hoặc **mini PC** (§10.10) | VM ARM sẽ fail theo cách KHÁC, dẫn bạn đi sai hướng |
| Driver, USB token, DX12, app 32-bit | **Chỉ mini PC / máy thật** | Không có đường nào khác |
| Bug chỉ xảy ra ở 150% scaling | **Máy Windows thật** đổi scale trong Settings | `--force-device-scale-factor` là fallback tự lừa dối |
| Bug chỉ xảy ra ở một user duy nhất | Sentry breadcrumb + log của chính user (§10.2) | Đừng cố tái hiện — hỏi thêm dữ liệu |

**Bước 5 — Bug auto-update là lớp đắt nhất và không có ở bảng nào phía trên.** Squirrel/NSIS updater, delta update, file association, deep link, per-user vs per-machine, gỡ cài đặt còn sót, migration khi nâng version. Cách duy nhất test được: cài **bản CŨ** trước, rồi để nó tự update lên bản mới, trên máy Windows thật hoặc VM có snapshot. Đưa nó vào checklist release, không phải vào CI.

---

## 10.12. Bảo mật — ba điều không được bỏ qua

**Container KHÔNG phải security boundary.** `systemd-nspawn` và Docker **chia sẻ kernel host**. Chạy một installer chưa ký / không rõ nguồn gốc trong đó = đặt cả VPS (và mọi artifact của khách trên đó) vào rủi ro. Quy tắc: binary của **chính bạn** → container thoải mái; binary **không tin cậy** → VM dùng một lần, rồi xoá.

**VPS: key-only, firewall, không expose gì ngoài SSH.**

```bash
# ===== Trên VPS, ngay sau khi dựng =====
# 1. Chỉ cho phép key, tắt password
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
systemctl reload ssh

# 2. Firewall: chặn tất cả trừ SSH
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw --force enable

# 3. TUYỆT ĐỐI KHÔNG mở 5900 (VNC) hay 3389 (RDP) ra internet.
#    Cổng 3389 mở sẽ bị quét trong vài phút. Dùng:
#    - x11vnc với cờ -localhost + SSH tunnel (§10.6.4), HOẶC
#    - Tailscale/WireGuard cho mọi truy cập tương tác
curl -fsSL https://tailscale.com/install.sh | sh && tailscale up
```

**Token và dữ liệu nhạy cảm.**

| Rủi ro | Cách xử lý |
|---|---|
| PAT của GitHub với scope `workflow` = quyền ghi repo | Dùng **fine-grained token** giới hạn đúng repo, hoặc GitHub App. Không dùng classic PAT |
| Screenshot và log rất dễ chứa API key, email khách, token trong URL | Chạy redaction **trước** khi lưu vào `runs/` — xem `beforeSend` ở §10.2.1. Thêm `runs/` vào `.gitignore` |
| `curl \| sh` để cài agent/tool | Pin checksum hoặc chữ ký. Tối thiểu: tải về, đọc, rồi mới chạy |
| `SubmitSamplesConsent SendAllSamples` trong CI | Upload installer chưa release lên Microsoft. Với sản phẩm thương mại/NDA: dùng `NeverSend` (§10.5.2) |
| Secret trong Actions log | Dùng `::add-mask::` hoặc GitHub Secrets; đừng `echo` biến môi trường để debug |

---

## 10.13. Ngân sách RAM và ổ cứng — bỏ qua mục nào nếu máy bạn nhỏ

Đây là bảng quyết định thật sự, quan trọng hơn mọi thứ kỹ thuật ở trên nếu máy bạn 8 GB.

| Máy Mac của bạn | Làm được | **BỎ QUA hoàn toàn** |
|---|---|---|
| **8 GB RAM** | §10.2 (Sentry), §10.3 (Playwright), §10.4 (container — nhưng đặt Docker Desktop limit 3 GB), §10.5 (CI), §10.6 (VPS), §10.7 (cloud VM), §10.10 (mini PC) | **§10.8 (UTM + Windows 11 ARM)** — guest cần 6-8 GB riêng, máy sẽ swap chết. Đừng thử |
| **16 GB RAM** | Tất cả, nhưng chật | Đừng chạy UTM Windows VM **cùng lúc** với Docker Desktop (mặc định ăn 4-8 GB) + Xcode + Node dev server. Đóng bớt trước khi bật VM |
| **24-32 GB RAM** | Tất cả, thoải mái | — |

| Ổ cứng Mac | Hệ quả |
|---|---|
| **256 GB** | **Loại khỏi mọi phương án VM local.** Windows 11 ARM VM cần 30 GB tối thiểu / 64 GB thực tế; +VS Build Tools 10-20 GB. Dùng VPS/cloud/mini PC |
| **512 GB** | Một VM Windows là được, hai thì chật |
| **1 TB+** | Thoải mái |

**Hai điều ít ai nói:**

- **SSD Mac hàn cứng, không thay được.** Mọi phương án dùng `savevm`/suspend VM liên tục ghi hàng chục GB mỗi ngày — một VM 8 GB suspend mỗi 5 phút = ~160 GB ghi/ngày ≈ **~50 TB/năm**. Đừng làm thế.
- **Pin và nhiệt.** Chạy VM Windows làm pin MacBook tụt nhanh 2-4x; MacBook Air không quạt sẽ throttle trong các run dài. **Kết luận hành động: mọi thứ chạy lâu nên đẩy sang VPS / cloud VM / mini PC, không chạy trên máy đang code.**

---

## 10.14. Những thứ ĐỪNG làm

| Đừng | Vì sao |
|---|---|
| **Đừng dùng CrossOver/Wine để verify build của bạn** | Nó không phải Windows thật. Registry, DPI scaling, font rendering, native dialog, MSI installer, Windows API edge case đều khác. Bug tìm thấy/không tìm thấy đều **không đáng tin**. Nó là sản phẩm cho **người dùng cuối**, không phải cho dev verify build |
| **Đừng thử emulate x86-64 Windows toàn hệ thống** | Parallels x86 emulator sau 1,5 năm vẫn là "early preview": boot 2-7 phút, 1 vCPU, không USB, không audio. QEMU TCG còn tệ hơn (đo được 7,2x-18,7x chậm) |
| **Đừng tin bất kỳ số đo hiệu năng nào trong VM ARM** | AVX2 = 0.66x so với SSE2 trên M2, ngược dấu hoàn toàn với x64 thật |
| **Đừng kết luận "chạy tốt trên Windows" sau khi test trên ARM64** | Đây là cách mất user nhanh nhất. Luôn nói đủ: *"PASS trên Windows 11 ARM64 (app x64 chạy qua Prism). CHƯA verify: [danh sách §10.8]"* |
| **Đừng hardcode `1920,1080` khi chụp màn hình** | Desktop của CI runner có thể là 1024x768 → ảnh đen viền hoặc exception. Luôn đọc `Screen.PrimaryScreen.Bounds` |
| **Đừng dùng `SubmitSamplesConsent SendAllSamples` trong CI của sản phẩm thương mại** | Bạn vừa upload installer chưa release lên Microsoft |
| **Đừng expose VNC (5900) hay RDP (3389) ra internet** | Sẽ bị quét trong vài phút. Dùng `-localhost` + SSH tunnel, hoặc Tailscale |
| **Đừng bỏ `timeout-minutes` trong GitHub Actions** | App GUI treo sẽ chạy tới mặc định **6 giờ** và ăn sạch quota tháng |
| **Đừng chạy `--platform linux/amd64` rồi tin `uname -m`** | Nó nói dối. Kernel vẫn là arm64, không có AVX, không chạy binary 32-bit |
| **Đừng để `--ephemeral` chạy trên rootfs không phải btrfs subvolume** | Nó âm thầm fallback sang copy toàn bộ: 30-90 giây thay vì 2 giây, và bạn sẽ tưởng systemd-nspawn chậm |
| **Đừng mua mini PC trước khi kiểm tra edition Windows** | Home không có Hyper-V, không có Windows Sandbox → mất cả hai cơ chế reset |
| **Đừng dùng key OEM giá rẻ cho VM** | OEM gắn vĩnh viễn với máy vật lý đầu tiên, không được chuyển sang VM. Chỉ retail/FPP mới chuyển được hợp pháp |
| **Đừng hứa chơi game Windows trên Mac** | Vanguard/EAC/BattlEye chủ động phát hiện và chặn VM. Không có Boot Camp trên Apple Silicon để lách |

---

## 10.15. Checklist tuần này

Sắp theo ROI. Nếu chỉ làm được ba dòng đầu, bạn đã lấy được 70% giá trị của cả phần này.

```text
NGÀY 1 (buổi sáng, ~3 giờ) — fidelity 100%, chi phí $0
  □ Cài Sentry + electron-log vào app, có redaction         §10.2.1   2 giờ
  □ Thêm menu item "Mở thư mục log"                         §10.2.1   15 phút
  □ Nhắn tìm 5-10 tình nguyện viên beta trong cộng đồng     §10.2.2   30 phút
    (cố ý chọn: 2 Win11 x64, 1 Win11 ARM, 2 Linux, 1 màn 4K@150%, 1 locale khác)

NGÀY 1 (buổi chiều, ~2 giờ) — assertion thật, chạy ngay trên Mac
  □ Viết 1 test Playwright: app lên + không tràn chữ        §10.3     1 giờ
  □ Thêm pseudolocalization, chạy lại test                  §10.3     1 giờ

NGÀY 2 (~1 giờ)
  □ Chạy §10.4 (container linux/amd64) trên artifact hiện tại        20 phút
  □ Chạy lại với --platform linux/arm64 để so hai kiến trúc          10 phút
  □ Mở checklist §10.8 "KHÔNG test được", đánh dấu dòng nào ảnh
    hưởng app của bạn — quyết định này chi phối mọi thứ còn lại      20 phút

NGÀY 3 (~1 giờ)
  □ Commit workflow §10.5, xem Defender có flag installer không      30 phút
  □ Kiểm tra quota/giá trên trang billing của chính tài khoản bạn    15 phút
  □ Thêm timeout-minutes vào MỌI job đang có sẵn                     10 phút

NGÀY 4 (~1,5 giờ) — chọn MỘT trong hai theo nhu cầu
  □ Nếu app có bản Linux: thuê VPS (Vultr/DO Singapore nếu bạn cần
    tương tác; Hetzner EU nếu chỉ batch) + dựng §10.6                1,5 giờ
  □ Nếu bug đang gặp là bug Windows x64: dựng VM Azure/AWS
    theo giờ (§10.7), test, tear down ngay — tốn ~$0,5               45 phút

NGÀY 5
  □ Nếu §10.8 checklist có dòng nào bị đánh dấu (native module,
    driver, DX12, app 32-bit, USB token, WSL2):
    → đặt mua mini PC / laptop cũ, KIỂM TRA EDITION trước (§10.10.1)
  □ Nếu không dòng nào bị đánh dấu:
    → dựng UTM + Windows 11 Enterprise Evaluation ARM64 (§10.8)      1-2 giờ

CUỐI TUẦN
  □ Rà lại §10.12: VPS đã key-only + ufw chưa? runs/ đã vào
    .gitignore chưa? PAT đã fine-grained chưa?                       30 phút
  □ Viết quy trình §10.11 thành một issue template trong repo        30 phút
```

**Một câu để nhớ:** mọi thứ trong §10.3-§10.10 chỉ là cách *đoán trước* điều gì sẽ xảy ra trên máy user. §10.2 là cách *biết chắc*. Làm §10.2 trước.

---

[← 5. Rủi ro và mô hình kinh doanh](05-rui-ro-va-kinh-doanh.md) · [Mục lục](README.md) · [Phụ lục A — Kết quả kiểm chứng lại →](phu-luc-a-kiem-chung.md)
