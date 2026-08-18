# 4. Kiến trúc, MVP và roadmap

> Kiến trúc kỹ thuật cụ thể, hợp đồng `run`, 16 rule của `scan`, ba spike bắt buộc, MVP 4 tuần và roadmap 12 tháng.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

---

## 5. KIẾN TRÚC KỸ THUẬT

## 5.1. Sơ đồ tổng thể

```
╔════════════════════════ macOS (Apple Silicon, arm64) ════════════════════════╗
║                                                                              ║
║  firstrun ── CLI viết bằng Go, single static binary, không dependency runtime║
║  │                                                                           ║
║  ├── scan/        STATIC ANALYZER (Lớp 0 — không cần mạng, không cần target) ║
║  │     ├─ detect/    nhận diện framework (package.json, tauri.conf.json,     ║
║  │     │             pubspec.yaml, *.csproj, go.mod, Cargo.toml)             ║
║  │     ├─ rules/     16 rule (§5.7) — nhóm mới: AUTO-UPDATE & LIFECYCLE      ║
║  │     └─ kb/        fidelity knowledge base (YAML, embed vào binary,        ║
║  │                   cập nhật qua manifest ký Ed25519)                       ║
║  │     → blindspot.json + report.html + exit code                            ║
║  │                                                                           ║
║  ├── run/         ORCHESTRATOR (Lớp 1)                                       ║
║  │     ├─ plan     đọc firstrun.yaml → danh sách (target × lane)             ║
║  │     ├─ sync     đẩy artifact (MVP: rsync-over-ssh; v2: FastCDC+zstd)      ║
║  │     ├─ exec     gọi agent, stream NDJSON về terminal                      ║
║  │     ├─ redact   LỌC secret khỏi log/ảnh TRƯỚC khi ghi xuống đĩa (§5.8)    ║
║  │     └─ collect  runs/<ts>/<target>/<lane>/{shot.png, app.log, result.json}║
║  │                                                                           ║
║  ├── targets/     DRIVER INTERFACE — điểm mở rộng duy nhất của sản phẩm      ║
║  │     ├─ ssh        VPS / mini PC / laptop cũ / máy công ty   ◄── MVP tuần 2║
║  │     ├─ winmetal   Windows x64 THẬT, agent dial-out          ◄── tháng 2   ║
║  │     ├─ gha        GitHub Actions (workflow_dispatch)        ◄── tháng 3   ║
║  │     ├─ cloudwin   Azure B2s / EC2 t3.medium tính theo GIỜ   ◄── tháng 4   ║
║  │     │             (Windows SERVER, KHÔNG phải Win11 client)               ║
║  │     ├─ local-vz   Linux arm64 VM trên Virtualization.fw     ◄── tháng 5   ║
║  │     └─ qemu-win   Windows 11 ARM64 (QEMU+HVF)               ◄── tháng 7+  ║
║  │                                                  (CHỈ nếu số liệu ép phải)║
║  └── report/      HTML self-contained + JUnit XML + JSON                     ║
║                                                                              ║
║  firstrun-vzd ── Swift, CHỈ tồn tại từ tháng 5, chỉ cho Linux guest trên VZ. ║
║                  Tách process vì VZVirtualMachine là process-local và không  ║
║                  NSSecureCoding → không truyền qua XPC được. Daemon SỞ HỮU VM║
║                  và expose IPC riêng (Lima/VirtualBuddy/apple-container đều  ║
║                  làm vậy) — đây KHÔNG phải rào cản kiến trúc.                ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
                │
                │  MỘT protocol (NDJSON), BA transport:
                │    • SSH exec + stdio        (ssh, winmetal, cloudwin)
                │    • vsock                   (local-vz)
                │    • WebSocket dial-out      (gha, winmetal sau NAT)
                ▼
╔═══════ target: Linux x86-64 | Linux arm64 | Windows x64 | Windows ARM ═══════╗
║                                                                              ║
║  firstrun-agent ── Go, CGO_ENABLED=0, cross-compile TỪ Mac ra 4 target:      ║
║                    linux/amd64, linux/arm64, windows/amd64, windows/arm64.   ║
║                    Đây là lý do chọn Go — và là bằng chứng sống rằng         ║
║                    "không cross-compile được từ macOS" là SAI (§5.4).        ║
║                                                                              ║
║  ┌─ provision ─ chuẩn bị rootfs / máy sạch                                   ║
║  │    Linux  : debootstrap | podman → /var/lib/machines/<distro>             ║
║  │             mỗi run = systemd-nspawn --ephemeral trên btrfs subvolume     ║
║  │    Windows: Hyper-V checkpoint revert | Windows Sandbox | runner mới      ║
║  │                                                                           ║
║  ├─ install ── nhận diện định dạng → package manager đúng                    ║
║  │    .deb → apt-get install -y ./x.deb       .rpm → dnf install -y ./x.rpm  ║
║  │    .AppImage → chmod +x; thiếu FUSE → --appimage-extract-and-run          ║
║  │    .exe → NSIS /S | Inno /VERYSILENT       .msi → msiexec /i /qn /l*v     ║
║  │                                                                           ║
║  ├─ drive ──── ƯU TIÊN 1: DRIVER AUTOMATION THẬT (§5.3)                      ║
║  │    Electron  : Playwright _electron.launch() | WebdriverIO electron-svc   ║
║  │    Flutter   : integration_test / flutter drive                           ║
║  │    WinUI/MAUI: Appium Windows Driver | FlaUI                              ║
║  │    → assertion trên cây UI thật, không suy đoán bằng pixel                ║
║  │                                                                           ║
║  ├─ launch ─── ƯU TIÊN 2: chạy artifact như user (khi KHÔNG có driver)       ║
║  │    lane=clean   Xvfb :99 + openbox + dbus-run-session  (X server của TA)  ║
║  │                 biến thể Wayland: sway/labwc --headless + grim            ║
║  │    lane=desktop session GNOME/KDE thật; capture qua RDP client-side       ║
║  │                                                                           ║
║  ├─ observe ── "app đã lên chưa?"                                            ║
║  │    CÓ driver     : chờ selector/điều kiện — xác định, không flaky         ║
║  │    KHÔNG driver  : poll ảnh 250ms + pixel-stability N=3  (FALLBACK)       ║
║  │    escape hatch  : --wait <giây> | --wait-for-log <regex>                 ║
║  │                                                                           ║
║  ├─ capture ── X11 (Xvfb ta tự dựng) : import -window root (ImageMagick)     ║
║  │             wlroots headless      : grim                                  ║
║  │             Windows               : GDI BitBlt (kbinani/screenshot, MIT)  ║
║  │             QEMU (v2)             : QMP screendump — lấy được cả khi treo ║
║  │                                                                           ║
║  └─ collect ── stdout/stderr (stream), exit code, journalctl --user,         ║
║                Windows Event Log (wevtapi) + WER LocalDumps,                 ║
║                Defender: MpCmdRun.exe -Scan + Get-MpThreatDetection,         ║
║                Authenticode: Get-AuthenticodeSignature                       ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

**Ba thay đổi so với bản trước, đều là sửa lỗi thật:**

1. Thêm tầng `drive` **trên** tầng `launch`. Screenshot bị hạ từ "cơ chế phát hiện chính" xuống "bằng chứng + fallback". Lý do ở §5.3.
2. Thêm bước `redact` vào đường ghi kết quả. Không phải tính năng — là điều kiện để công cụ này không trở thành nơi rò rỉ API key của khách (§5.8).
3. Bỏ ghi chú "GNOME đã xoá X11 nên không chụp được màn hình". Sai. `lane=clean` dựng **X server của chính mình** (Xvfb) bên trong container — nó không dính dáng gì tới GNOME, GDM, hay phiên bản X11 session của distro. Trạng thái X11 trong GNOME 49/50 **không ảnh hưởng** tới đường này (độ tin cậy Cao về cơ chế; độ tin cậy Trung bình về mốc phiên bản GNOME cụ thể).

## 5.2. Hợp đồng `run` — đây LÀ sản phẩm

Mọi thứ khác là chi tiết triển khai. Hợp đồng phải ổn định và độc lập với backend:

```
INPUT : một file artifact bất kỳ (.deb .rpm .AppImage .exe .msi .dmg .zip .tar.gz)
        + một target descriptor
        + một lane descriptor (dpi, locale, theme, session)
        + (tuỳ chọn) một test script cho driver automation

OUTPUT: runs/<timestamp>/<target>/<lane>/
        ├── result.json
        ├── shot-t3.png, shot-t10.png, shot-final.png
        ├── app.log            (stdout + stderr, có timestamp, ĐÃ redact)
        ├── system.log         (journalctl / Event Log, ĐÃ redact)
        ├── install.log        (output của package manager / msiexec /l*v)
        ├── driver.log         (trace của Playwright/Appium nếu có)
        └── crash.dmp          (nếu có)
```

```jsonc
// result.json — schema ổn định, versioned
{
  "schema": 1,
  "verdict": "PASS" | "FAIL" | "NOT-COVERED",   // ba trạng thái, KHÔNG nhị phân
  "evidence": "driver" | "screenshot" | "exit-code-only",  // MỚI: bằng chứng
  "fidelity": {                                            // đến từ đâu
    "label": "linux-x86_64-native" | "linux-arm64-native"
           | "linux-x86_64-rosetta" | "win-x64-native"
           | "win-arm64-prism" | "container-shared-kernel"
           | "win-server-not-win11-client",
    "not_covered": [                 // LUÔN xuất hiện, KHÔNG tắt được
      "kernel-mode drivers",
      "GPU/DirectX/Vulkan behaviour (software rendering)",
      "SmartScreen reputation (phụ thuộc prevalence, không tái hiện được trên CI)",
      "different kernel version (container chia sẻ kernel với host)",
      "auto-update từ bản đã cài sẵn (chỉ test cài mới)"
    ]
  },
  "install": { "ok": true, "duration_ms": 8300, "manager": "apt" },
  "launch":  { "ok": true, "window_stable_at_ms": 4120, "exit_code": null },
  "checks":  [
    {"rule":"no-clipped-text","status":"fail","detail":"nút 'Lưu' scrollWidth 118 > clientWidth 84"},
    {"rule":"contrast","status":"pass"}
  ],
  "artifacts": {"screenshots":["shot-t3.png"],"logs":["app.log"]}
}
```

Trường `evidence` là bổ sung bắt buộc sau khi thêm driver automation: một verdict `PASS` dựa trên `driver` và một verdict `PASS` dựa trên `screenshot` **không cùng độ tin cậy**, và công cụ phải nói ra điều đó thay vì để người đọc tự đoán. `exit-code-only` nghĩa là ta chỉ biết tiến trình không chết — yếu nhất, và nên hiển thị màu xám chứ không màu xanh.

**Vì sao hợp đồng này là tài sản:** cùng một lệnh chĩa được vào VM Linux local, VPS x86-64, mini PC, VM Windows Server tính theo giờ, Windows ARM VM, hay GitHub Actions runner. Nó **không bị vô hiệu hoá bởi giới hạn Apple Silicon**, không bị Apple/Parallels copy trong một release, và nếu sản phẩm chết nó vẫn sống được dưới dạng một GitHub Action hoặc một CLI open-source.

## 5.3. Automation UI THẬT vs screenshot — quyết định kiến trúc quan trọng nhất

Bản kiến trúc trước đặt cược vào "poll screenshot mỗi 250ms + pixel-stability" làm cơ chế biết app đã khởi động và đã đúng. Đó là một **sai lầm về thứ tự ưu tiên**: nó tự viết lại, bằng heuristic pixel, một việc mà các framework automation đã giải xong, miễn phí, xác định, và chạy được ngay trên chính chiếc Mac này.

### 5.3.1. Cái đã tồn tại và miễn phí

| Framework app | Công cụ drive UI thật | Chạy trên Mac? | Headless? | Chi phí |
|---|---|---|---|---|
| Electron | **Playwright `_electron.launch()`** (chỉ định `executablePath` tới binary đã cài) | Có | Có (Linux cần Xvfb làm display) | $0 |
| Electron | **WebdriverIO + `wdio-electron-service`** | Có | Có | $0 |
| Flutter desktop | `integration_test` / `flutter drive` | Có | Có | $0 |
| .NET MAUI / WinUI 3 | **Appium Windows Driver** (WinAppDriver), **FlaUI** (UIAutomation) | Không — cần host Windows | Không (cần session tương tác) | $0 |
| Tauri | WebDriver qua `tauri-driver` + `webkit2gtk-driver` (Linux) / Edge driver (Windows) | Một phần | Một phần | $0 |

Độ tin cậy: **Cao** cho Playwright/WebdriverIO Electron và Flutter `integration_test` (đây là API có tài liệu, dùng rộng rãi). **Trung bình** cho mức độ hoàn thiện hiện tại của `tauri-driver` trên macOS host — chưa xác minh lại trong 8/2026.

Hai sắc thái phải nói thẳng, vì chúng quyết định kiến trúc:

- API Electron của Playwright **hẹp hơn** API browser của nó và về mặt lịch sử được đánh dấu là thử nghiệm. Nó ổn định đủ để làm smoke test và assertion luồng, **không** nên coi là nền tảng cho một sản phẩm QA đầy đủ.
- Trên Linux, driver **không xoá bỏ Xvfb**. Electron vẫn cần một display để mở cửa sổ. Xvfb chuyển vai trò từ "cơ chế chụp ảnh" thành "màn hình để driver làm việc" — đây là tin tốt: cùng một hạ tầng, ít việc hơn, và spike S2 (§5.10) không còn là điểm chết duy nhất.

### 5.3.2. Khi nào dùng cái nào

| Loại bug | Công cụ ĐÚNG | Vì sao |
|---|---|---|
| Luồng người dùng sai (bấm Lưu không lưu, dialog không đóng) | **Driver** | Assertion trên trạng thái thật; screenshot chỉ thấy "có gì đó thay đổi" |
| Nội dung DOM/cây UI sai, i18n string không load | **Driver** | Đọc được text thật, không phải OCR ngược từ pixel |
| App treo, không phản hồi | **Driver** (timeout xác định) | Pixel-stability không phân biệt được "đã ổn định" với "đã đơ" — đây là điểm yếu chí mạng của cơ chế cũ |
| Regression logic | **Driver** | Chạy được mỗi push, trên Mac, không cần target |
| **Installer bị Defender flag, MSI/NSIS fail, thiếu dependency runtime** | **Screenshot + log + exit code** | Driver chạy sau khi app đã cài xong — nó mù hoàn toàn với bước cài |
| **Tray icon / StatusNotifierItem biến mất trên Wayland** | **Screenshot** | Tray không nằm trong process app; không driver nào thấy nó |
| **DPI hệ thống 125%/150%, non-client area, `WM_DPICHANGED`** | **Screenshot** | Driver đọc giá trị app khai báo, không đọc cái Windows thật sự vẽ |
| **Font fallback, thiếu font CJK/Việt, chữ vẽ thành ô vuông** | **Screenshot** | CSSOM trả về font **đã khai báo**, không trả về font **đã dùng** |
| **Native dialog của OS, window chrome, CSD, splash screen** | **Screenshot** | Ngoài phạm vi renderer |
| **Gỡ cài đặt còn sót, file association, deep link** | **Log + script hệ thống** | Không thuộc về cả hai |
| Kernel driver, DX12, USB token | **KHÔNG CÔNG CỤ NÀO** → `NOT-COVERED` | Cần máy x64 vật lý |

### 5.3.3. Hệ quả kiến trúc

1. `firstrun run` **chạy driver trước nếu có**, và chỉ rơi về pixel-stability khi project không có test script hoặc framework không có driver.
2. Screenshot **không bị bỏ**. Nó chuyển từ "cơ chế phán quyết" sang hai vai trò mà nó làm tốt nhất: **bằng chứng đính kèm** (ảnh app trên Ubuntu để dán vào PR) và **công cụ duy nhất** cho nhóm bug ở nửa dưới bảng trên — đúng nhóm bug mà một dev Mac-only không bao giờ thấy.
3. Điều này **làm mạnh** luận điểm sản phẩm chứ không làm yếu: `firstrun` không cạnh tranh với Playwright, nó **chở** Playwright tới những nơi Playwright không tự tới được (một máy Linux x86-64 sạch, một máy Windows thật có Defender bật), rồi thu về những thứ Playwright không thấy được.
4. Với dev Electron/Flutter, đây cũng là câu trả lời trung thực cho câu hỏi "tôi có cần công cụ này không": **nếu bạn chưa có test Playwright, hãy viết nó trước** — nó rẻ hơn, nhanh hơn, và bắt được nhiều bug hơn bất cứ thứ gì trong tài liệu này. `firstrun` có giá trị từ dòng thứ hai trở đi.

## 5.4. Ngôn ngữ, thư viện — và sự thật về cross-compile

| Thành phần | Ngôn ngữ | Lý do |
|---|---|---|
| `firstrun` CLI | **Go** (bản ổn định hiện hành — không pin version trong tài liệu) | Single static binary; cross-compile ra 4 target agent bằng cùng toolchain; `spf13/cobra`, `charmbracelet/bubbletea` (TUI ma trận), `gopkg.in/yaml.v3` |
| `firstrun-agent` | **Go**, cùng codebase, `CGO_ENABLED=0` | `golang.org/x/sys/windows/svc`, `mdlayher/vsock` (MIT), `kbinani/screenshot` (MIT) |
| `firstrun scan` rules | **Go** | Embed toàn bộ KB vào binary; `npx` wrapper mỏng chỉ để phân phối |
| So sánh ảnh | Go, `disintegration/imaging` + pixelmatch tự viết | Tránh **AGPL** (không dùng `dssim`) vì sản phẩm closed-source |
| `firstrun-vzd` (tháng 5+) | **Swift** | Truy cập 100% API VZ ngay khi Apple release. Entitlement: chỉ `com.apple.security.virtualization` |
| Phân phối | Homebrew tap + npm + `.pkg` notarized (Developer ID) | KHÔNG Mac App Store. Lưu ý chi phí: Apple Developer Program $99/năm (~2,5 triệu VND) là **điều kiện bắt buộc** trước khi có đồng doanh thu đầu tiên |
| Thanh toán | **Paddle / Lemon Squeezy** (Merchant of Record) | Họ gánh VAT/sales tax toàn cầu |
| License key | Ed25519, verify **offline** | Không tài khoản, không server, không telemetry bắt buộc |

### 5.4.1. Cross-compile từ macOS: bảng phân loại đúng

Khẳng định "không cross-compile được từ macOS ra Windows" là **quá rộng và sai về mặt kỹ thuật**. Chính agent của sản phẩm này là phản ví dụ. Phân loại đúng gồm bốn nhóm:

| Nhóm | Trường hợp | Trạng thái từ macOS Apple Silicon | Độ tin cậy |
|---|---|---|---|
| **Sạch, không cần Windows** | Go: `GOOS=windows GOARCH=amd64/arm64` + `CGO_ENABLED=0` | Chạy hoàn hảo, ra `.exe` native | **Cao** |
| | Rust: target `*-pc-windows-gnu` + mingw-w64 (`brew install mingw-w64`) | Chạy được | Cao |
| | Ký Authenticode `.exe`/`.dll`/`.msi` bằng **`osslsigncode`** (`brew install osslsigncode`), có timestamp RFC3161 | Chạy được từ Mac | Cao |
| **Được nhưng có điều kiện** | Electron → NSIS installer qua `electron-builder` trong Docker `electronuserland/builder:wine` | Được; đường chính thống là chạy image này **trên Linux/CI**, không phải Wine trên macOS ARM | **Thấp** cho trạng thái khuyến nghị 2026 — chưa verify lại |
| | Tauri → `*-pc-windows-msvc` qua `cargo-xwin` (kể cả `aarch64`) | Hoạt động, cộng đồng dùng, **nhưng không phải luồng chính thức**; EULA của MSVC chỉ cho dùng trên máy có giấy phép Visual Studio → **vùng xám pháp lý cho sản phẩm thương mại** | **Thấp** cho trạng thái 2026 |
| | Go/Rust có CGO, UI toolkit (Fyne, Wails, webview) | Cần mingw-w64; một số lib vẫn không build được | Trung bình |
| **Bắt buộc cần Windows** | .NET MAUI target `net*-windows` (WinUI 3 / WinAppSDK) | Cần Windows SDK + MSBuild trên Windows. `dotnet build -f net9.0-windows` trên macOS báo thiếu workload | **Cao** |
| | `flutter build windows` | Cần Visual Studio + C++ desktop workload, chỉ chạy trên host Windows | **Cao** |
| | MSIX/APPX, MSI đầy đủ (WiX), driver/kernel | Cần Windows | Cao |
| | Native module Node (`better-sqlite3`, `keytar`, `node-canvas`) cho win32 | node-gyp cần MSVC; `link.exe` không có trên macOS | Cao |
| **Không phải "chặn", mà là thiếu tooling** | — | Microsoft/Google **không cấm bằng chính sách**; đây là giới hạn engine/toolchain. Phân biệt này quan trọng vì nó có thể thay đổi, còn "chính sách cấm" thì không | Cao |

**Hệ quả cho kiến trúc:** kết luận "phải build **bên trong** guest" (lý do đã giết concept Rig) chỉ đúng cho **native module + MAUI + Flutter desktop + MSIX/MSI**. Với Electron thuần JS và Tauri, cross-compile hoặc build trên CI Linux/Windows là đường đi bình thường. Điều này gỡ bỏ yêu cầu "cài VS Build Tools ~10GB vào golden image" khỏi MVP — nó chỉ quay lại ở nhóm framework thứ ba.

**Khuyến nghị thực dụng cho dev Mac-only:** dùng GitHub Actions `windows-latest` làm máy build+sign cho **mọi** target Windows. Miễn phí với repo public, rẻ với private ($0.010/phút, độ tin cậy Cao), và nó giải quyết luôn cả nhóm "bắt buộc cần Windows". Cross-compile local chỉ dùng để lặp nhanh, **không dùng cho bản release**.

**Cảnh báo ký số:** `osslsigncode` ký được từ macOS bằng file PFX — nhưng nếu chứng chỉ OV/EV của bạn nằm trong HSM/token theo yêu cầu CA/B Forum (từ 2023), bạn phải ký qua dịch vụ cloud signing (Azure Trusted Signing, DigiCert KeyLocker), không phải bằng file. Đây là ràng buộc thật và ảnh hưởng tới rule #10 của `scan`.

## 5.5. Đồng bộ file — ba mức, chọn theo backend

| Backend | Cơ chế | Ghi chú |
|---|---|---|
| `ssh` (VPS/mini PC/cloudwin) | `rsync -az --partial` qua SSH; v2: FastCDC (`restic/chunker`, BSD) + zstd → installer 200MB, lần thứ hai chỉ đẩy 5-20MB | Đơn giản, tin cậy |
| `local-vz` | **virtiofs** (`VZVirtioFileSystemDeviceConfiguration(tag:"ws")`). Với Linux guest, bake tag vào `/etc/fstab` hoặc một systemd `.mount` unit trong golden image là đủ — mount tự động lúc boot, **không cần agent để mount**. Giới hạn Apple nêu là về việc script mount vào một guest **tuỳ ý đang chạy** | Agent vẫn cần cho việc khác, nhưng không phải cho mount |
| `qemu-win` | **KHÔNG dùng virtiofs.** Dùng: agent làm HTTP/gRPC file server trên virtio-net + `hostfwd`; hoặc bật macOS File Sharing rồi mount `\\10.0.2.2\share` từ guest qua NAT | Xem cảnh báo dưới |
| `gha` | `actions/upload-artifact` + `actions/download-artifact` bằng token của **chính user** | Không đi qua hạ tầng của bạn → không chi phí biên, không DPA, không trách nhiệm pháp lý với binary của khách |

> **Cảnh báo giữ nguyên, không được giấu:** **virtiofs từ macOS host vào Windows ARM64 guest qua QEMU KHÔNG TỒN TẠI.** `virtiofsd` là Linux-only; QEMU không build virtfs/9p trên macOS; virtiofs của Apple chỉ có trong VZ (mà VZ không boot được Windows); driver virtio-fs ARM64 còn thiếu trong virtio-win. **Bằng chứng xác nhận mạnh nhất: Lima v2.2 — codebase tham chiếu hợp pháp gần nhất — đánh dấu template Windows là `plain: true`, tức TẮT file mount và port forwarding, chính xác vì thứ này không làm được.** Đừng vay mượn một blueprint mà bản thân blueprint đó có sẵn dòng chữ "phần này không chạy".

## 5.6. Snapshot / reset — con số END-TO-END, không phải con số của bước rẻ nhất

| Backend | Cơ chế reset | **Thời gian end-to-end tới lúc chạy được artifact** |
|---|---|---|
| Linux qua ssh | `systemd-nspawn --ephemeral` trên **btrfs subvolume** | ~2s cho container; **+20-40s nếu `apt-get update` mỗi lần** → bắt buộc pre-bake dependency vào rootfs, nếu không lời hứa "2 giây" là không có thật |
| Linux qua ssh (không root/systemd) | Podman/Docker container từ image đã build sẵn | ~1-3s |
| `local-vz` | APFS `clonefile(2)` (`COPYFILE_CLONE_FORCE`) trên golden disk khi VM đã tắt | **~15-20s tổng.** Bước clone ~5ms là **không đáng kể**; thời gian bị chi phối hoàn toàn bởi **boot VM**. Ghi "5ms" là gây hiểu nhầm và tài liệu này không ghi nữa |
| Windows trên mini PC | Hyper-V checkpoint revert | ~10-20s. **Điều kiện: Windows 11 Pro.** Phần lớn mini PC N100 giá rẻ ship **Home**, không có Hyper-V → phải kiểm tra edition trước khi mua, hoặc dùng Macrium Reflect image restore |
| Windows trên máy x64 thật (smoke test installer) | **Windows Sandbox** — clean room tức thì, reset khi đóng | ~10-20s. Đơn giản hơn nhiều so với vòng Hyper-V checkpoint. Cũng cần Pro/Enterprise |
| `cloudwin` (Azure/AWS theo giờ) | Xoá VM + tạo lại từ image tuỳ chỉnh (`az vm create` / `aws ec2 run-instances`) | ~2-5 phút. Đắt về thời gian, rẻ về tiền |
| `gha` | Runner mới mỗi job | ~30-60s queue |
| `qemu-win` | qcow2 backing chain: `qemu-img create -b golden -F qcow2 overlay.qcow2` + **cold boot** | ~40-60s |

> **Cảnh báo giữ nguyên:** `savevm`/`loadvm` với `-accel hvf` trên aarch64 **chưa có bằng chứng công khai là hoạt động** (không có call HVF trong migration path của target arm; đây là giới hạn đã nêu trong series patch Apple Silicon của QEMU, không phải bug sẽ được sửa). **Đừng bao giờ hứa "warm resume 3-8 giây" cho Windows guest trước khi tự đo.** Đây là lỗi đã giết concept Ferry. Reset Windows guest = **cold boot**, chấm hết, cho tới khi S5 chứng minh ngược lại.

**Ghi chú về ngân sách ghi SSD:** mọi phương án dựa trên snapshot VM đều ghi lượng lớn dữ liệu lên SSD **hàn cứng, không thay được** của MacBook. Một vòng suspend 8GB mỗi 5 phút ≈ 160GB/ngày ≈ **~50TB/năm**. Đây là lý do kỹ thuật — không phải lý do tiết kiệm — để đẩy mọi thứ chạy lâu sang VPS/mini PC thay vì chạy trên chính máy đang code.

## 5.7. `firstrun scan` — 16 rule (Lớp 0)

Mỗi rule = một construct trong source code → một lớp bug cụ thể → một câu trả lời "setup hiện tại của bạn có bắt được không".

| # | Rule | Phát hiện | Lớp bug bị che |
|---|---|---|---|
| 1 | `native-addon-no-prebuilt` | dependency có `.node` (better-sqlite3, keytar, node-canvas, sharp, serialport, node-pty, robotjs) không có prebuilt `win32-arm64`/`linux-arm64` | node-gyp mặc định không tải `node.lib` arm64; build ARM64 fail theo cách **khác hoàn toàn** lỗi thật trên x64. Đây cũng là nhóm **duy nhất** thật sự bắt buộc build trong guest Windows (§5.4) |
| 2 | `ia32-target` | `electron-builder` config có target `ia32`/`win32-ia32` | Prism chỉ expose AVX/AVX2/BMI/FMA/F16C cho **x64**; app 32-bit không thấy. (Lưu ý sửa: Windows 10 on ARM **có** x64 emulation từ preview 12/2020; luận điểm đúng và mạnh hơn là **Windows 10 đã EOL 14/10/2025** nên không còn là target) |
| 3 | `hardcoded-dll` | chuỗi `.dll` hardcode, `ffi-napi`/`koffi`/`LoadLibrary` | DLL x64 bên thứ ba không có bản ARM64 → Prism che giấu hoặc fail vì lý do khác |
| 4 | `arch-detection` | `process.arch`, `os.arch()`, `GetNativeSystemInfo`, `RtlGetVersion` | Với process x64 chạy dưới emulation, `GetNativeSystemInfo` trả về `PROCESSOR_ARCHITECTURE_AMD64` — **cố tình nói dối để tương thích**. Chỉ `IsWoW64Process2`/`GetMachineTypeAttributes` lộ ARM64 |
| 5 | `kernel-mode` | file `.sys`, `.inf`, WDK project, driver signing config | **Không bao giờ** test được trên VM ARM. Prism chỉ dịch user-mode |
| 6 | `dx12-vulkan` | `d3d12`, `--use-angle=d3d12`, `wgpu` backend, `vulkan` | Trên **stack bạn tự xây** (VZ, QEMU+HVF) không có 3D accel. Parallels/VMware có driver paravirtual riêng nhưng trần ở DX11.1 |
| 7 | `wayland-tray` | dùng tray/`Tray` API mà không có fallback StatusNotifierItem/AppIndicator | Tauri #14234: tray mất trên Wayland ở `.deb` nhưng bình thường ở AppImage. Electron #34801, #11866, #21445 |
| 8 | `deeplink-windows` | có handler `open-url` (macOS) nhưng không có `second-instance` | Windows không có `open-url` → Electron mở instance thứ hai |
| 9 | `long-path` | độ sâu `node_modules` + tên file dài → nguy cơ MAX_PATH 260 trong NSIS | Chỉ vỡ khi cài thật trên Windows |
| 10 | `unsigned-installer` | không có `certificateFile`/cloud signing config | SmartScreen warning; chuỗi issue electron-builder #6334 #6347 #550 #5759 #4878: NSIS installer bị Defender báo trojan **kể cả khi đã ký EV**. Cảnh báo kèm: chứng chỉ OV/EV sau 2023 phải nằm trong HSM/token → không ký được bằng file PFX |
| 11 | `webview2-bootstrap` | `tauri.conf.json` dùng `downloadBootstrapper` | User offline/enterprise không cài được; WebView2 evergreen ≠ WKWebView khoá theo macOS ≠ WebKitGTK khoá theo distro |
| 12 | `no-arm64-windows-build` | CI matrix không có `windows-11-arm` hoặc không build arm64 | App chạy dưới Prism trên máy Snapdragon → chậm và có thể crash. **Cập nhật 2026: `windows-11-arm` đã mở cho cả public VÀ private repo**, không còn giới hạn public-only — nên rule này không còn "lời khuyên bất khả thi" |
| 13 | `string-expansion` | chuỗi UI không có ngân sách giãn nở | Tiếng Đức/Việt dài hơn tiếng Anh 30-40% → tràn nút. **Chạy pseudolocalization ngay trên Mac để chứng minh** |
| 14 | `fractional-dpi` | dùng đơn vị pixel cứng, `devicePixelRatio` giả định 1/2 | macOS dùng hệ số **nguyên** (2x); Windows mặc định 125%/150% trên laptop 1080p/1440p. Qt QTBUG-53022 |
| 15 | `maui-flutter-crossbuild` | có target WinUI3/`windows/` mà build chạy trên macOS | Thiếu **tooling**, không phải chính sách cấm: MAUI cần Windows SDK + MSBuild trên Windows; `flutter build windows` cần Visual Studio C++ workload |
| **16** | **`update-lifecycle`** *(MỚI — nhóm gây thảm hoạ lớn nhất, trước đây không rule nào chạm tới)* | Xem phân rã bên dưới | Xem phân rã bên dưới |

### 5.7.1. Rule 16 phân rã — AUTO-UPDATE & INSTALL LIFECYCLE

Đây là nhóm bug **đắt nhất** của desktop app: nó không làm app fail lúc bạn test, nó làm app fail **ở lần cập nhật thứ hai, trên máy của user đã cài bản cũ** — trạng thái mà không setup nào trong tài liệu này tái tạo được nếu không cố tình.

| 16.x | Kiểm tra | Lớp bug bị che |
|---|---|---|
| 16.1 | `electron-updater`/Squirrel.Windows/NSIS updater có cấu hình nhưng **không có test cập nhật từ version cũ** | Update từ v1.0 → v1.1 fail vì đổi `appId`, đổi publisher name, hoặc đổi chữ ký → Squirrel từ chối, user kẹt vĩnh viễn ở bản cũ |
| 16.2 | Delta update (blockmap) bật mà artifact cũ không được giữ lại | Delta tính sai → tải full mỗi lần (mất băng thông) hoặc apply fail |
| 16.3 | Cài **per-user** (`%LOCALAPPDATA%`) vs **per-machine** (`Program Files`) — `oneClick`/`perMachine` của electron-builder | Cài đè lẫn nhau → **hai bản app cùng tồn tại**; updater ghi vào Program Files không có quyền admin → im lặng thất bại |
| 16.4 | File association (`.myext`) khai báo nhưng không có bước gỡ đăng ký khi uninstall | Registry còn sót, mở file ra app đã bị xoá |
| 16.5 | Deep link / custom protocol (`myapp://`) đăng ký per-user vs per-machine không nhất quán | Deep link chết sau khi update; hoặc bản cũ cướp protocol của bản mới |
| 16.6 | Không có migration cho schema dữ liệu người dùng khi tăng version | Update xong app crash lúc mở, mất dữ liệu — lớp bug **không thể phục hồi** và giết uy tín nhanh nhất |
| 16.7 | Uninstaller không xoá scheduled task / service / autostart entry | Máy user còn sót tiến trình sau khi gỡ; bị anti-virus đánh dấu |

**Ràng buộc trung thực phải ghi vào `not_covered`:** `firstrun run` ở MVP chỉ test **cài mới trên máy sạch**. Test auto-update đúng cách đòi hỏi một target **có trạng thái** (cài bản cũ → chạy → update → verify), tức là ngược hoàn toàn với kiến trúc "ephemeral state" của Lớp 1. Đây là một **giới hạn cấu trúc**, và nó phải xuất hiện trong `not_covered[]` của mọi run cho tới khi có backend hỗ trợ. Rule 16 vì vậy là **rule tĩnh** — nó chỉ ra bạn có nguy cơ, nó không chứng minh bạn an toàn.

### 5.7.2. Output — bảng ba cột

```
LỚP BUG                          SETUP HIỆN TẠI CỦA BẠN         CÁCH BẮT ĐƯỢC
────────────────────────────────────────────────────────────────────────────────
Logic UI, luồng người dùng       ✓ bắt được ngay trên Mac        Playwright _electron
Installer bị Defender flag       ✗ KHÔNG BAO GIỜ                 run --on win-x64
                                   (CI tắt Defender)              (Defender bật thật)
native .node arm64 ABI           ✗ KHÔNG BAO GIỜ                 windows-11-arm trong CI
tray icon trên Wayland           ✗ KHÔNG BAO GIỜ                 run --on linux:desktop
DPI 150% layout                  ✗ KHÔNG BAO GIỜ (macOS chỉ 2x)  run --lane dpi=150
font fallback CJK thiếu          ✗ KHÔNG BAO GIỜ                 run + screenshot
auto-update v1.0 → v1.1          ✗ KHÔNG BAO GIỜ, KỂ CẢ CÓ VM    cần target CÓ TRẠNG THÁI
                                   (mọi test đều máy sạch)        — chưa có trong v1
gỡ cài đặt còn sót               ✗ KHÔNG BAO GIỜ                 cần target có trạng thái
driver kernel-mode               ✗ KHÔNG BAO GIỜ, KỂ CẢ CÓ VM    máy Windows x64 vật lý
DX12 / GPU driver                ✗ KHÔNG BAO GIỜ, KỂ CẢ CÓ VM    máy Windows x64 vật lý
tràn chữ khi dịch tiếng Đức      ✗ chưa bật                      scan --pseudoloc ← trên Mac
```

Hai dòng "cần target CÓ TRẠNG THÁI" là chỗ công cụ này **tự thú nhận giới hạn của chính nó**. Chúng phải nằm trong bảng, không được xoá cho đẹp.

## 5.8. Bảo mật — bốn thứ phải làm từ dòng code đầu tiên

Đây là góc độ hoàn toàn thiếu trong bản trước, và nó không phải hạng mục "làm sau".

| # | Vấn đề | Bắt buộc |
|---|---|---|
| **1** | **Container không phải security boundary** | `systemd-nspawn`/Docker **chia sẻ kernel host**. Chạy một installer chưa ký/không rõ nguồn gốc trong đó = đặt cả VPS (và mọi artifact của khách trên đó) vào rủi ro. Quy tắc: **binary không tin cậy → VM dùng một lần, không phải container.** Tài liệu trước chỉ nói "chia sẻ kernel" ở góc độ *fidelity*, không ở góc độ *bảo mật* — đó là thiếu sót nghiêm trọng |
| **2** | **VPS chạy agent với quyền root, SSH public IP** | `PasswordAuthentication no`, key-only, ufw chặn mọi thứ trừ 22. **Tuyệt đối không expose xrdp/VNC ra internet** — 3389 mở sẽ bị quét trong vài phút. Dùng Tailscale/WireGuard cho mọi truy cập tương tác. Agent chạy dưới user không đặc quyền, chỉ nâng quyền cho bước `provision` |
| **3** | **Token và quyền của backend `gha`** | `workflow_dispatch` cần PAT scope `workflow` — mà scope đó **là quyền ghi repo**. Không được yêu cầu PAT classic. **Bắt buộc: fine-grained personal access token** (chỉ repo được chọn, chỉ quyền Actions) **hoặc GitHub App** với permission tối thiểu. Ghi rõ trong onboarding token đó làm được gì, vì user đang trao cho công cụ này quyền ghi vào repo của họ |
| **4** | **Screenshot và log là nơi rò rỉ dữ liệu** | Screenshot của một app đang chạy rất dễ chứa **API key, email khách hàng, token trong URL, tên file nội bộ**; log còn dễ hơn. Bắt buộc có bước **`redact`** trước khi ghi xuống `runs/`: regex cho các pattern token phổ biến (`sk-`, `ghp_`, `Bearer `, JWT, chuỗi base64 dài), lọc biến môi trường, và tuỳ chọn `--redact-file` cho pattern riêng của user. Với screenshot, tối thiểu là cảnh báo rõ ràng trước lần chia sẻ đầu tiên. **Mặc định phải là an toàn, không phải là tiện** |
| **5** | **Cài agent bằng `curl \| sh`** | Đường cài một dòng là bắt buộc về mặt UX nhưng phải **pin checksum SHA256 và verify chữ ký Ed25519** trong chính script, và script phải phục vụ từ domain có TLS pinning nếu có thể. Một công cụ được cài bằng `curl \| sh` lên máy Windows của công ty khách hàng mà không verify gì là một sự cố supply-chain đang chờ xảy ra |

## 5.9. Quy trình tái hiện bug user báo

Đây là **lý do phổ biến nhất** khiến một dev Mac-only cần Windows — phổ biến hơn nhiều so với "test trước khi release" — và bản trước không có một dòng công thức nào. Nó cũng là quy trình mà `firstrun` phục vụ tốt nhất, vì nó là quy trình **có target rõ ràng, có bằng chứng, và lặp lại nhiều lần**.

| Bước | Việc | Công cụ |
|---|---|---|
| 1 | **Lấy đủ ngữ cảnh từ user trước khi động vào code** | Template issue bắt buộc: version app chính xác, `winver` (build number Windows), kiến trúc (x64 vs ARM64 — hỏi bằng Task Manager cột "Architecture", không hỏi "máy anh chip gì"), DPI scaling %, ngôn ngữ hệ thống, cài per-user hay per-machine, có phải máy vừa update từ bản cũ không |
| 2 | **Bật log của chính app** | Electron: `--enable-logging` (ghi ra stderr / `%LOCALAPPDATA%\<app>\logs`); `electron-log` với file transport bật sẵn ở production. Đây là thứ phải có **trước khi** bug xảy ra, không phải sau |
| 3 | **Lấy crash dump thật** | Bật **WER LocalDumps** trên máy user: `HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps` với `DumpFolder`, `DumpType=2` (full). Hướng dẫn user bằng một file `.reg` ký sẵn, không bắt họ gõ regedit |
| 4 | **Đọc được dump đó** | Cần **symbol server** riêng cho bản build của bạn (Electron có symbol công khai cho binary của nó; symbol phần native của bạn thì phải tự lưu **mỗi lần build**, theo build ID). Nếu bạn chưa lưu symbol theo version, dump của user là vô dụng — đây là việc phải làm **hôm nay**, không phải khi có bug |
| 5 | **Crash reporter tự động** | Sentry/Crashpad gửi crash + breadcrumb + version + OS build về ngay, không cần user làm gì. Xem §7 — đây là hạng mục ROI cao nhất trong toàn bộ roadmap |
| 6 | **Tái hiện có kiểm soát** | `firstrun run <artifact> --on <target> --lane <đúng DPI/locale user báo>` — và nếu bug thuộc nhóm auto-update, phải dựng target **có trạng thái**: cài bản cũ, chạy, rồi update |
| 7 | **Khi không tái hiện được, nói ra** | Đối chiếu với `blindspot.json`: nếu lớp bug đó nằm trong `NOT-COVERED`, trả lời user bằng sự thật ("đây là bug kernel-mode driver, tôi không có môi trường tái hiện, cần bạn chạy giúp bước X") thay vì đoán. Đây cũng là chính sách support: **từ chối chịu trách nhiệm cho ca mà `scan` đã báo trước là NOT-COVERED** |

## 5.10. Ba spike bắt buộc — làm TRƯỚC khi viết dòng code sản phẩm nào

| Spike | Câu hỏi | Nếu FAIL thì sao |
|---|---|---|
| **S1** (2 ngày) | `systemd-nspawn --ephemeral` trên VPS có cho máy sạch < 5s **end-to-end** không (đã pre-bake dependency, đã tạo btrfs **subvolume** trước khi debootstrap)? `apt-get install ./x.deb` bên trong có giải được dependency offline không? | Đổi sang Podman/Docker container — vẫn OK, chỉ mất systemd. Rủi ro thấp |
| **S2** (3 ngày) | **(a)** Playwright `_electron.launch({executablePath: ...})` có drive được một app Electron **đã cài từ `.deb`** bên trong `xvfb-run` trên VPS x86-64 không? **(b)** Xvfb + openbox + `import -window root` có chụp được một `.AppImage` thật (`--no-sandbox --disable-gpu --appimage-extract-and-run`) không? | **Không còn là spike sống-chết như bản trước.** (a) fail → mất tầng driver, quay về pixel-stability (yếu hơn nhưng vẫn ship được). (b) fail → mất bằng chứng ảnh cho nhóm bug tray/DPI/font. **Chỉ khi CẢ HAI fail** thì Lớp 1 mới không có tính năng bán hàng. Đây chính là giá trị của việc thêm driver: nó phá thế điểm-chết-đơn |
| **S3** (1 ngày) | Chụp màn hình trên GitHub Actions `windows-latest` (Windows Server 2025, session 0) có ra **ảnh đen** không? | Nếu đen: backend `gha` chỉ làm được Defender scan + install/uninstall verify + driver automation headless, không làm được ảnh → phải dùng `winmetal` cho ảnh Windows |

**Ba spike này trả lời được trong 6 ngày và quyết định concept sống hay chết** — thay vì phát hiện ở tháng thứ 6 như 5 concept trong panel.

**Hai spike hoãn lại (không cần cho MVP, nhưng phải làm trước tháng 5 và tháng 7):**

- **S4:** Capture `VZVirtualMachineView` offscreen ra PNG có ra ảnh thật hay ảnh trắng? (View backed bởi `CAMetalLayer` thường trả khung trắng với `cacheDisplay`; **không có API đọc framebuffer trong VZ** — chưa kiểm chứng được, phải thử. Đường vòng: chạy Xvfb + `import` **bên trong** guest, giống hệt backend `ssh` — nếu S2(b) pass thì S4 gần như không còn quan trọng.)
- **S5:** `savevm`/`loadvm` với `-accel hvf` trên aarch64 có hoạt động không? Và reset Windows VM về sạch mất bao nhiêu giây khi **không** có savevm?

---

## 6. MVP 4 TUẦN

**Nguyên tắc:** mỗi tuần kết thúc bằng một thứ **chạy được và kiểm chứng được bằng số**. Không tuần nào là "làm hạ tầng". Cắt tàn nhẫn.

## Tuần 0 (6 ngày trước tuần 1) — SPIKE

Chạy S1, S2, S3 ở §5.10. **Nếu cả S2(a) lẫn S2(b) fail, dừng lại và thiết kế lại.** Không viết code sản phẩm.

Song song, làm hai việc chi phí gần bằng 0 nhưng phải bắt đầu sớm nhất có thể (xem §7, hàng tháng 1): **bật crash reporting cho app của chính bạn** và **mở form đăng ký beta**. Cả hai không tiêu tốn thời gian phát triển và cả hai đều cần thời gian để tích luỹ dữ liệu.

## Tuần 1 — `firstrun scan` (sản phẩm miễn phí, độc lập, có thể ship riêng)

**Làm:**
- CLI Go: `firstrun scan [path]` → `blindspot.json` + `report.html` + exit code
- Framework detection: Electron (`package.json` + electron-builder), Tauri (`tauri.conf.json`), Flutter (`pubspec.yaml`), .NET MAUI (`*.csproj`)
- **11 rule** trong 16 rule ở §5.7: các rule 1-10 + **rule 16 (`update-lifecycle`, ít nhất 16.1/16.3/16.6)**. Rule 16 vào MVP **không phải để đủ số** — nó là rule có tỷ lệ "finding đúng trên repo thật" cao nhất trong cả bộ, vì gần như mọi app Electron đều cấu hình `perMachine`/`oneClick` mà không có test update
- Rule 11-15 để tuần sau
- Report HTML self-contained, bảng ba cột như §5.7.2
- npm wrapper: `npx firstrun-scan`

**KHÔNG làm:** cấu hình rule tuỳ ý, plugin system, web app, tài khoản, telemetry, đa ngôn ngữ.

**Deliverable kiểm chứng được:** chạy trên **10 repo Electron/Tauri open-source thật** (gợi ý: Joplin, Bruno, Standard Notes, Beekeeper Studio, Notesnook, RustDesk, Lens, Insomnia, AppFlowy, Bitwarden desktop). **Tiêu chí pass: ≥7/10 repo cho ra ít nhất 2 finding mà maintainer sẽ đồng ý là đúng.** Nếu không đạt, rule engine sai — sửa trước khi đi tiếp.

## Tuần 2 — `firstrun run --on ssh://` (Linux x86-64 thật)

**Làm:**
- `firstrun-agent` Go, `CGO_ENABLED=0`, cross-compile `linux/amd64` + `linux/arm64` từ Mac (5 phút, không cần Docker, không cần Linux — đây là bằng chứng cho §5.4)
- `firstrun target add vps --ssh root@1.2.3.4` → cài agent qua SSH **có verify checksum + chữ ký**, provision rootfs bằng `debootstrap` trên btrfs subvolume, pre-bake dependency
- `firstrun run ./dist/app.AppImage --on vps`:
  - rsync artifact
  - `systemd-nspawn --ephemeral` (fallback: Podman container), `--resolv-conf=copy-host`, bind `/dev/fuse`
  - nhận diện `.deb`/`.rpm`/`.AppImage` → cài đúng cách
  - `xvfb-run` + openbox + **`dbus-run-session`** (thiếu D-Bus là nguyên nhân treo phổ biến nhất với app GTK/Electron)
  - **nếu project có test script: chạy driver (Playwright `_electron`) và lấy verdict từ đó**
  - **nếu không: fallback poll `import -window root` + pixel-stability N=3**
  - thu stdout/stderr/exit code thật của **app** (không phải của lệnh chụp ảnh)/`journalctl --user`
  - **redact** rồi kéo `shot.png` + `result.json` về Mac
- Terminal output stream real-time

**KHÔNG làm:** ma trận, song song, GUI, VM local, Windows, reset ngoài `--ephemeral`, delta sync (rsync là đủ).

**Deliverable:** trên một VPS x86-64 sạch, lệnh `firstrun run ./dist/MyApp.AppImage --on vps` trả về screenshot + exit code + log **trong dưới 60 giây**, cho **cả `.AppImage` lẫn `.deb`**, và `result.json` ghi đúng `evidence: "driver"` khi có test script. Quay video 30 giây làm demo.

> **Ghi chú chọn VPS (độ tin cậy Thấp cho mọi con số giá — cần tự kiểm tra):** Hetzner CX22 (2 vCPU, 4 GB) ~€3,79/tháng **+ ~€0,50 IPv4 ≈ €4,3/tháng**, nhưng dòng **CX/CAX chỉ có ở châu Âu**; Hetzner Singapore chỉ có dòng CPX/CCX với phụ phí vị trí. Latency VN → Falkenstein **~230-300ms** (vọt lên 350ms+ khi đứt cáp quang biển) so với VN → Singapore **~25-50ms**. Thêm nữa, Hetzner **nổi tiếng khó với khách Việt Nam**: yêu cầu xác minh danh tính, hay từ chối thẻ phát hành tại VN (độ tin cậy Trung bình). Kết luận thực dụng: **với build/CI chạy nền thì latency không quan trọng — Hetzner EU vẫn rẻ nhất và đáng dùng.** Với phiên tương tác thì lấy Vultr/DigitalOcean/Linode Singapore (2 vCPU/4 GB ~$20-24/tháng, độ tin cậy Thấp), hoặc VPS trong nước (Vietnix/AZDIGI ~200-450k VND/tháng) nếu cần hoá đơn VAT và hỗ trợ tiếng Việt.

## Tuần 3 — Ma trận, lane, verdict, report

**Làm:**
- `firstrun.yaml`: khai báo targets × lanes
- Nhiều rootfs trên cùng một VPS: `ubuntu2404`, `ubuntu2604`, `debian13`, `fedora44` → `--on ubuntu2604,fedora44` **chạy song song** (rootfs Fedora dựng bằng `podman run fedora:44` export ra, không phải `dnf --installroot` từ host Ubuntu)
- Lane: `dpi` (100/150 qua `GDK_SCALE`+`GDK_DPI_SCALE`/`QT_SCALE_FACTOR`/`--force-device-scale-factor`), `locale` (`LANG=vi_VN.UTF-8`), `pseudoloc`
  - **BẮT BUỘC:** nếu DPI được set bằng cờ app thay vì system metric, lane phải mang nhãn **`simulated-dpi — không đại diện`** trong `result.json` và trên report. Đây là lỗi đã giết concept Triptych: fallback im lặng khiến công cụ báo PASS sai. `--force-device-scale-factor` chỉ scale DPR trong renderer, **không** đổi system metric, **không** đổi non-client area, **không** kích hoạt `WM_DPICHANGED` — tức là bỏ sót chính xác lớp bug đang hứa bắt
- Verdict **PASS / FAIL / NOT-COVERED** + khối `not_covered[]` **luôn xuất hiện, không tắt được** + trường `evidence`
- 3 rule đầu tiên chạy trên chính screenshot/DOM: `no-clipped-text`, `no-out-of-bounds`, `contrast`
- Report HTML: cột theo target, ảnh cạnh nhau, click finding → highlight box
- `--junit` + exit code ≠ 0 khi FAIL
- Bổ sung font: `fonts-noto-core` đủ cho tiếng Việt, **`fonts-noto-cjk` bắt buộc nếu app có UI Trung/Nhật/Hàn** — thiếu nó thì screenshot ra ô vuông và bạn sẽ báo bug sai cho chính mình

**KHÔNG làm:** pixel regression baseline (v2), visual diff cheo OS (**không bao giờ** — bài toán ill-posed), video, crash dump parsing.

**Deliverable:** `firstrun run ./dist/app.deb --on ubuntu2604,fedora44 --lane dpi=100,dpi=150` cho ra **4 screenshot + 1 report HTML trong dưới 3 phút**, và bắt được ít nhất **1 bug thật** trong app của chính bạn.

## Tuần 4 — Phân phối và ĐO LƯỜNG

**Làm:**
- Đóng gói: Homebrew tap + npm + `.pkg` notarized (Developer ID, hardened runtime)
- Landing page **một trang**, phần trên là bảng trung thực §5.7.2 (không phải feature list)
- Doc: quickstart 5 phút + trang **"Cái gì `firstrun` KHÔNG test được"** (đây là trang quan trọng nhất) + trang "Nếu bạn chưa có test Playwright, hãy làm việc đó trước" (§5.3.3 — nói thẳng, nó xây lòng tin nhanh hơn mọi feature)
- Free: `scan` vĩnh viễn + `run` trên 1 target
- **Telemetry opt-in đo ĐÚNG MỘT SỐ: số lần `run` mỗi user mỗi tháng.** Không đo gì khác
- Phân phối: PR vào doc của electron-builder / `tauri-action`; post vào Tauri Discord, r/electronjs, r/tauri, r/rust, HN Show; demo là **ảnh side-by-side app trên macOS vs Ubuntu Wayland với tray icon biến mất** — không phải ảnh terminal in "PASS"

**Deliverable:** public release; **20 beta user thật**; số đo `run`/user/tháng bắt đầu chảy về.

## Tiêu chí GIẾT (định nghĩa trước, không đàm phán sau)

| Mốc | Đo gì | Ngưỡng giết |
|---|---|---|
| Tuần 1 | `scan` cho finding đúng trên repo thật | < 7/10 repo có ≥2 finding đúng |
| Tuần 8 | Số cài `firstrun scan` | < 300 lượt trong 4 tuần sau launch |
| Tháng 3 | **Median `run`/user/tháng** | **< 4** → mô hình subscription sai; xoay trục sang credit-based hoặc one-time, HOẶC pivot hẳn sang GitHub Action |
| Tháng 6 | Khách trả tiền | < 30 → thị trường đã trả lời. Giữ `scan` như OSS, dừng đầu tư Lớp 1 |

---

## 7. ROADMAP 6-12 THÁNG

| Mốc | Nội dung | Vì sao thứ tự này | Gate để đi tiếp |
|---|---|---|---|
| **Tháng 1** — Telemetry, crash reporting, kênh beta *(chạy song song MVP, KHÔNG phải sau)* | Cho **app của chính bạn** trước, rồi mới thành recipe cho khách: Sentry hoặc Crashpad + `electron-log` với file transport bật ở production, **lưu symbol theo từng build**, WER LocalDumps hướng dẫn sẵn bằng file `.reg`. Song song: tuyển **5-10 tình nguyện viên dùng Windows/Linux** làm kênh beta (Discord/Tauri community/r/electronjs), gửi build trước mỗi release | **Đây gần như chắc chắn là hành động ROI cao nhất trong cả tài liệu — trên cả mini PC $250.** Nó cho bạn **bug THẬT, trên máy THẬT, của user THẬT** — thứ không VM nào trên Apple Silicon cho được, ở bất kỳ giá nào. Free tier của Sentry đủ dùng cho dev solo. Kênh beta có chi phí **$0** và **fidelity 100%**, và đó là cách phần lớn dev cross-platform solo **thực sự đang làm**. Đặt nó ở cuối roadmap như bản trước là sai thứ tự nghiêm trọng | ≥3 crash report thật từ máy không phải của bạn trong 30 ngày |
| **Tháng 2** — `winmetal` | Windows x64 **THẬT** làm target: agent dial-out qua Tailscale/SSH từ mini PC N100, **laptop Windows cũ 2-4 triệu** (có sẵn màn hình, bàn phím, pin = UPS tích hợp, thường kèm license OEM), Dell OptiPlex/HP EliteDesk micro refurb, hoặc máy công ty của user. Reset bằng **Hyper-V checkpoint** hoặc **Windows Sandbox** | **Đây là target x64 thật duy nhất có được mà không dính QMTH.** Chi phí kỹ thuật gần bằng 0 (chỉ là transport thứ ba). Nó biến đối thủ mạnh nhất thành backend. **Cảnh báo bắt buộc trong doc:** Hyper-V và Windows Sandbox **không có trên Windows 11 Home** — mà phần lớn mini PC giá rẻ ship Home. Phải kiểm tra edition trước khi mua | ≥10 user kết nối một máy Windows thật |
| **Tháng 3** — `gha` + Defender prescan | Backend GitHub Actions (`workflow_dispatch` bằng **fine-grained token hoặc GitHub App** của chính user — không bao giờ PAT classic scope `workflow`, xem §5.8). **Defender prescan làm cho đúng**: `Set-MpPreference -DisableRealtimeMonitoring $false` + `Update-MpSignature` + `MpCmdRun.exe -Scan` + `Get-MpThreatDetection` + `Get-AuthenticodeSignature`. Thêm `timeout-minutes` cho mọi job; dùng `SendSafeSamples` **không phải** `SendAllSamples` (nó sẽ upload installer chưa release của bạn lên Microsoft) | Tính năng có **ROI cao nhất trên mỗi giờ code** — installer bị flag = mất user thật, đo được bằng tiền. Không cần một dòng hypervisor. **Sự thật nền phải ghi rõ:** runner image của GitHub **chủ động tắt Defender** (`Configure-WindowsDefender.ps1` set `DisableRealtimeMonitoring`, `ExclusionPath = C:\, D:\`, và `ForceDefenderPassiveMode` khi registry path ATP tồn tại — độ tin cậy Cao, đọc trực tiếp source). Verdict ML đám mây (`!ml`) phụ thuộc prevalence/reputation → **CI không tái hiện được**, phải ghi vào `not_covered` | Defender prescan bắt được ≥1 false-positive thật của user |
| **Tháng 4** — `cloudwin` (Windows theo GIỜ) | Driver cho Azure B2s / AWS EC2 t3.medium **tính theo giờ**: `az vm create/deallocate/delete` + `az snapshot create`, `aws ec2 run-instances / create-image / terminate-instances`, image tuỳ chỉnh để tái dùng mỗi build | **Sửa một khẳng định sai trong phân tích thị trường:** "không có API spin up/tear down theo build, mọi thứ là per-user-per-month" là **sai**. Windows tính theo giờ tồn tại, license gộp trong giá giờ, API đầy đủ. Azure B2s Windows ~$0,10/h; **20 giờ/tháng ≈ $2 compute**, cộng disk/snapshot → **~$4-12/tháng thực tế**, rẻ hơn Windows 365 Business ($28-56/user/tháng) **3-7 lần**; điểm hoà vốn ~250-300 giờ/tháng (độ tin cậy Trung bình-Cao cho Azure/AWS, **Thấp cho GCP**). **Ràng buộc phải nói thẳng:** license gộp đó là **Windows Server**, KHÔNG phải Windows 11 client → `fidelity.label = "win-server-not-win11-client"`. Đủ cho Win32/WinUI; **không** đủ cho shell integration, MSIX, packaged app, DPI scaling đặc thù Win11 | ≥5 user dùng `cloudwin` cho ≥3 run/tháng |
| **Tháng 5** — `local-vz` + Apple `container` | Linux arm64 VM local trên Virtualization.framework (`firstrun-vzd`, Swift). `systemd-nspawn` chạy **bên trong** VM → cùng cơ chế reset. **Đánh giá `apple/container` (Apache-2.0, v1.2.2 08/08/2026) làm backend thay cho tự viết Swift**: mỗi container một micro-VM trên Virtualization.framework, và **`container run --arch amd64` chạy binary linux/amd64 qua Rosetta** — tức nó phủ đúng use case "chạy/test binary Linux x86-64 trên MacBook Apple Silicon" (độ tin cậy Cao, đọc trực tiếp repo Apple). Rosetta for Linux làm một switch, kèm cảnh báo: **không AVX/AVX2, không binary 32-bit** | Chỉ thêm khi đã có user. Giá trị là **offline + chi phí biên $0**, không phải fidelity (VPS x86-64 fidelity cao hơn). Dùng `container` thay vì tự viết Swift có thể tiết kiệm hàng tuần — **nhưng nó yêu cầu macOS 26 để có đủ tính năng** (macOS 15 chạy được nhưng container không nói chuyện được với nhau và có bug subnet), và nó là mô hình VM-per-container nên tốn RAM hơn shared-VM khi chạy nhiều container. Repo Apple vẫn ghi rõ "minor version có thể có breaking changes". **Thời gian boot: KHÔNG có con số công bố — đừng trích dẫn "dưới 1 giây", độ tin cậy Thấp** | S4 pass, HOẶC chấp nhận chỉ dùng lane `clean` với Xvfb+driver trong guest (khả năng cao hơn nhiều) |
| **Tháng 6** — Framework recipes + CI hai chiều | `firstrun init` sinh sẵn steps đúng cho Electron (electron-builder NSIS + Playwright `_electron` test skeleton), Tauri (WebView2 multi-version), Flutter (`integration_test`), .NET MAUI (**build TRONG Windows** vì thiếu tooling trên macOS, §5.4). PR bot: mỗi PR tự có ảnh app trên Ubuntu/Fedora đính kèm + báo khi layout/tray vỡ so với baseline **cùng lane** | **Đây là bước sửa bài toán tần số** — chuyển từ 4 lần/năm sang 20 lần/tuần. Chuyển sản phẩm từ "devtool tiện nghi" sang "quality gate". Việc sinh sẵn test skeleton Playwright là hành động **cho đi giá trị trước khi thu tiền** và là kênh phân phối tự nhiên nhất | Median run/user/tháng tăng ≥3x |
| **Tháng 7-9** — Target CÓ TRẠNG THÁI + auto-update testing | Backend giữ trạng thái giữa các run: cài v1.0 → chạy → apply update → verify. Đây là thứ biến rule 16 từ "cảnh báo tĩnh" thành "bằng chứng". Cơ chế: snapshot theo tên (`--snapshot after-v1.0`) trên `winmetal` (Hyper-V checkpoint) và trên VPS (btrfs snapshot của rootfs, không phải `--ephemeral`) | **Đây là lớp bug đắt nhất của desktop app và hiện không công cụ nào trên thị trường chạm tới.** Nó cũng là thứ duy nhất trong roadmap tạo ra khác biệt mà một `justfile` 40 dòng không sao chép được trong một buổi chiều. Ưu tiên cao hơn `qemu-win` | ≥20% user bật `--snapshot` |
| **Tháng 7-9 (song song, CHỈ nếu số liệu ép)** — `qemu-win` | Windows 11 ARM64 local qua QEMU+HVF+swtpm+EDK2+virtio-win ARM64. **Onboarding dùng Windows 11 Enterprise ARM64 Evaluation ISO — 90 ngày, KHÔNG cần product key** (độ tin cậy **Cao**, xác nhận trực tiếp trên Microsoft Evaluation Center: có bản Arm64, "A product key is not required for this software"). Vendor logic từ Lima v2.2 (Apache-2.0). QEMU ship **binary rời**, QMP socket, written offer for source. Hai đường license dài hạn thay thế phải nêu trong doc: **Visual Studio Professional/Enterprise subscription** (có kèm Windows cho dev/test — câu trả lời chuẩn ngành, nhưng quá đắt cho dev solo; giá cụ thể **độ tin cậy Thấp**) và **Windows Insider Preview ARM64 VHDX** (miễn phí, dùng trực tiếp cho VM, nhưng build có hạn sử dụng và là license pre-release; **độ tin cậy Trung bình**) | **Đây là phần đã giết 4/5 concept. Chỉ làm khi đã có doanh thu và đã có user đòi.** Nếu bỏ qua vĩnh viễn, sản phẩm vẫn sống. Nhắc lại: **không có warm resume** (§5.6) và **không có virtiofs** (§5.5) | ≥100 khách trả tiền VÀ ≥30% yêu cầu Windows local |
| **Tháng 10-12** — Corpus + team | Fidelity knowledge base công khai (mỗi ca "tool bảo pass mà user vẫn lỗi" → một rule mới). Pixel regression **trong-lane**. Team: shared config, baseline trong repo, SSO | Đây là moat duy nhất **tăng dần theo thời gian** | — |

**Ba thứ theo dõi nhưng KHÔNG đặt cược:**

1. **Triton + Neptune** (DirectX 11 cho QEMU, osy, 2026) — nếu ổn định, đây là sản phẩm đầu tiên có 3D cho Windows guest ngoài Parallels. Hiện chính tác giả mô tả là "very unstable".
2. **Lima** — đọc `lima-vm/lima` issue #4852 và roadmap Windows guest. **Đây là đối thủ số 1, không phải Parallels**: dự án CNCF, có nhân lực tài trợ, Apache-2.0, miễn phí vĩnh viễn. Chọn tư thế: đóng góp upstream rồi xây lớp thương mại phía trên (mô hình Docker Desktop/Rancher), HOẶC đi vào chỗ Lima cấu trúc không đi (driver automation, screenshot, DPI matrix, AV prescan, auto-update testing).
3. **macOS 27** (9/2026): `VZCustomVirtioDevice` + `VZVirtioSharedMemoryRegion` về lý thuyết cho phép tự implement virtio-gpu với Venus backend chạy trên MoltenVK/Metal — tức mang 3D acceleration vào Linux guest trên VZ, thứ chính Apple chưa làm. Rủi ro rất cao, phần thưởng rất lớn, **không phải việc của năm nay**. *(Codename "Golden Gate" là tin đồn — **độ tin cậy Thấp**, không dùng như dữ kiện. Việc Rosetta for Linux được tích hợp thẳng vào OS ở macOS 27 là tín hiệu tích cực, nhưng **đừng xây kiến trúc mà mất Rosetta là chết**.)*

---

[← 3. Năm hướng sản phẩm và khuyến nghị](03-nam-huong-va-khuyen-nghi.md) · [Mục lục](README.md) · [5. Rủi ro và mô hình kinh doanh →](05-rui-ro-va-kinh-doanh.md)
