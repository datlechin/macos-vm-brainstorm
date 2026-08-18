# 1. Sự thật nền tảng

> Mười một ràng buộc kỹ thuật và pháp lý quyết định mọi phương án. Ý tưởng nào mâu thuẫn với chúng là sai, không phải khó.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

Mười một ràng buộc dưới đây quyết định mọi thứ. Nếu một ý tưởng mâu thuẫn với bất kỳ điều nào, ý tưởng đó sai — không phải "khó". Các mục có đánh dấu độ tin cậy là những chỗ chưa xác minh lại được bằng nguồn trực tuyến trong đợt rà soát 17/08/2026; đừng đọc chúng như dữ kiện cứng.

## 1.1 Apple Silicon KHÔNG chạy được Windows x86-64. Chấm hết.

CPU không có ISA đó. Không có Boot Camp trên M-series. Không hypervisor nào (Parallels/VMware/UTM/QEMU) thay đổi được điều này. Chỉ còn hai đường:

| Đường | Thực tế 2026 |
|---|---|
| Virtualize **Windows 11 ARM64** native | Gần tốc độ native. App x64 chạy qua **Prism** (translator của Microsoft, có từ 24H2). Đây là đường duy nhất dùng được. |
| Emulate **x86-64 toàn hệ thống** | Parallels x86 emulator ra 01/2025, đến bản 26 (2026) **vẫn là "early technology preview"**: boot Windows 2-7 phút, **1 vCPU**, **không USB**, **không audio**, chính Parallels ghi "responsiveness of the operating system is low". QEMU TCG còn tệ hơn (đo được 7.2x-18.7x chậm). **Không dùng được cho công việc thật.** |

**Hệ quả sản phẩm:** bất kỳ câu marketing nào ngụ ý "chạy Windows x64 như thật" đều là quảng cáo sai sự thật, và có rủi ro pháp lý thật.

Ràng buộc này chỉ áp dụng cho **Windows 11** — Windows 10 đã ra khỏi ma trận, xem 1.10.

## 1.2 Virtualization.framework (VZ) KHÔNG boot được Windows

Apple chỉ hỗ trợ chính thức 2 guest: **Linux arm64** và **macOS arm64**. `VZEFIBootLoader` (macOS 13+) về lý thuyết boot được mọi OS có EFI, nhưng:

- VZ **không cung cấp TPM 2.0 device** — Windows 11 bắt buộc có.
- Windows ARM64 **không có driver** cho bộ virtio device của Apple.
- Thực nghiệm: user UTM báo Windows 10/11 ARM chỉ ra **màn hình đen** khi dùng backend Apple Virtualization.

**Muốn Windows → bắt buộc QEMU + Hypervisor.framework (HVF) + swtpm + EDK2 + virtio-win ARM64.** Đây là điểm rẽ nhánh kiến trúc quan trọng nhất của toàn bộ bài toán, và là nơi 4/5 concept chết.

Ngược lại, cho **Linux guest** thì VZ là lựa chọn đúng — và từ 06/2026 có thêm một đường first-party: `apple/container` (Apache-2.0, v1.2.2 phát hành 08/08/2026), mỗi container một micro-VM trên VZ, có `--arch amd64` chạy binary x86-64 qua Rosetta. Yêu cầu **macOS 26** để dùng đầy đủ; trên macOS 15 bị cắt nặng (container không nói chuyện được với nhau, lỗi subnet làm mất mạng). Chưa thay thế được Docker Desktop/OrbStack về trải nghiệm và hệ sinh thái, nhưng đã đủ dùng cho use case "chạy/test binary Linux x86-64 trên MacBook". Độ tin cậy: Cao (đọc trực tiếp repo Apple); riêng con số boot time Apple **không công bố**, đừng trích.

## 1.3 Fidelity gap ARM64 vs x64 — đây là rủi ro chí mạng, không phải footnote

Test trên Windows 11 ARM64 **không tương đương** test trên Windows x64. Cụ thể, những lớp bug bị bỏ sót hoặc bị bóp méo:

| Lớp bug | Chuyện gì xảy ra trên VM ARM |
|---|---|
| Native `.node` addon (better-sqlite3, keytar, node-canvas) | Fail theo cách **khác hoàn toàn** với x64 thật; `node-gyp` mặc định không tải `node.lib` arm64 |
| DLL x64 bên thứ ba không có bản ARM64 | Prism **che giấu** lỗi, hoặc fail vì lý do khác |
| Code detect CPU/arch | Với process x64 chạy dưới emulation, `GetNativeSystemInfo` trả về `PROCESSOR_ARCHITECTURE_AMD64` — **cố tình nói dối để tương thích**. Chỉ `IsWoW64Process2`/`GetMachineTypeAttributes` mới lộ ARM64 |
| Race condition / memory ordering | ARM memory model **yếu hơn** x86 TSO → bug mới xuất hiện (báo động giả), và bug x86 thật bị Prism che |
| Hiệu năng | Benchmark RemObjects 02/2026: **AVX2 trên M2 qua emulation = 0.66x so với SSE2** — tức chậm hơn, **ngược dấu** với x64 thật (2.7x). Mọi số đo trong VM ARM là vô nghĩa |
| Driver kernel-mode | Prism **chỉ dịch user-mode**. Driver x64 không bao giờ chạy. → không test được VPN driver, ASIO audio, print driver, USB token, anti-cheat |
| DirectX 12 / Vulkan | **Không hypervisor nào trên Mac có DX12.** Parallels trần ở DX11.1 + OpenGL 3.3 (vẫn thế tính đến 02/2026) |
| Nested virt trong Windows guest | Không có → mất **Hyper-V, WSL2, Windows Sandbox, VBS/HVCI, Docker Desktop, Android Emulator**. Chỉ còn WSL1 |
| App 32-bit | Prism chỉ expose AVX/AVX2/BMI/FMA/F16C cho **x64**. App 32-bit và x64-có-helper-32-bit không thấy |

**Đây là câu quan trọng nhất trong tài liệu:** một công cụ báo "PASS trên Windows" khi thực chất chỉ mới chạy trên ARM64 sẽ giết chính nó ở lần đầu tiên user báo lỗi sau khi ship.

## 1.4 Licensing Windows — BYO-license là con đường duy nhất

- ISO Windows 11 ARM64 **đã tải công khai** từ 11/2024, hiện là 25H2, tại `microsoft.com/software-download/windows11arm64`. Rào cản "lấy đâu ra image" đã biến mất.
- **Chỉ Parallels Desktop được Microsoft "authorized"**, và chỉ cho Windows 11 **Pro/Enterprise** ARM. Không có văn bản cấm hypervisor khác — nhưng bạn ở vùng xám "unsupported": không có support từ Microsoft, khó biện minh khi doanh nghiệp audit.
- **Tuyệt đối không được đóng gói/phân phối ISO hay product key.** Windows 11 **Home ARM nằm ngoài phạm vi authorized**.
- **Không được host Windows 10/11 CLIENT OS trên hardware multi-tenant** nếu không là **QMTH partner**. Sắc thái quan trọng: điều bị cấm là hardware **dùng chung**; từ 2022 Microsoft cho phép chạy Windows 11 với license của chính khách trên **dedicated hardware** qua Authorized Outsourcer. Đây vẫn là rào cản cứng cho mô hình "bán VM Windows theo giờ", và là lý do BrowserStack/Sauce Labs không bán "Windows desktop VM".
- **Windows 11 Enterprise Evaluation ARM64, 90 ngày, KHÔNG cần product key** (Microsoft Evaluation Center, xác nhận 08/2026 — độ tin cậy Cao). Đây là đường rẻ nhất và hợp pháp nhất để một dev Mac-only có VM Windows ARM ngay hôm nay, và là cách duy nhất cho user trải nghiệm Windows guest mà không bắt họ mua license trước khi thấy giá trị. Timebomb 90 ngày, không convert được sang production, phải dựng lại VM (snapshot golden image để rebuild nhanh).

Hai đường hợp pháp khác mà tài liệu gốc bỏ sót:

| Đường | Nội dung | Độ tin cậy |
|---|---|---|
| **Visual Studio Professional/Enterprise subscription** | Kèm quyền dùng Windows client & server **cho dev/test**, không giới hạn 90 ngày. Ràng buộc: chỉ dev/test, per-user, cấm dùng làm máy làm việc hằng ngày. Đây là câu trả lời chuẩn ngành cho "dev cần VM Windows lâu dài" | Trung bình (benefit); **Thấp cho giá** — con số ~1.199 USD/năm Pro thường được nhắc nhưng chưa verify. Với dev solo VN thì quá đắt so với nhu cầu |
| **Windows Insider Preview ARM64 (VHDX/ISO)** | Tải miễn phí với tài khoản đã đăng ký Insider, có bản **VHDX** dùng thẳng cho VM. License pre-release, chỉ để đánh giá/test, build **có hạn** và tự shutdown nếu không cập nhật | Trung bình |

Về mua đứt: license **retail/FPP được phép chuyển sang máy/VM khác**; license **OEM gắn vĩnh viễn với máy vật lý đầu tiên** và **không hợp pháp cho VM**. Mọi "key giá rẻ" bán tràn lan đều là OEM hoặc tệ hơn. Giá tham khảo (~199 USD quốc tế, ~4-5 triệu VND tại VN): **độ tin cậy Thấp, chưa kiểm chứng lại**.

## 1.5 GPU acceleration cho guest: không có trên stack bạn tự xây

Đây là chỗ tài liệu bản trước phát biểu quá rộng. Phát biểu đúng:

| Stack | 3D cho guest |
|---|---|
| **VZ (Linux guest)** | Chỉ `virtio-gpu` 2D. Apple **không** implement virgl/Venus backend → Mesa fallback về **llvmpipe (software rendering)**. Tính đến macOS 26 Tahoe và công bố WWDC26 (macOS 27), vẫn chưa có 3D accel cho Linux guest |
| **QEMU + HVF (Windows ARM guest)** | Không có GPU accel. UI chạy bằng framebuffer software |
| **Parallels Desktop** | **CÓ** 3D paravirtual: trần ở **DX11.1 + OpenGL 3.3**. Không có DX12/Vulkan |
| **VMware Fusion** | **CÓ** 3D cho ARM guest, cùng hạng trần |
| **libkrun/krunkit** (Apache-2.0) | Đường **duy nhất** có Vulkan cho Linux guest: virtio-gpu **Venus** → MoltenVK → Metal. Red Hat báo cải thiện ~40x GPU compute. Không phải VZ |

Nói gọn: **trên mọi stack bạn tự xây (VZ, QEMU+HVF) thì không có 3D**; sản phẩm thương mại thì có nhưng trần ở DX11.1. Đừng viết "không có GPU accel cho guest" ở dạng tuyệt đối — nó mâu thuẫn với chính bảng đối thủ.

**Cửa sổ cơ hội 2026:** dự án **Triton + Neptune** của osy (tác giả UTM), công bố ~07-08/2026, mang **DirectX 11 thật** vào QEMU kể cả Windows 11 ARM64 guest trên Apple Silicon (host dùng D3DMetal/DXMT). Đã chạy được Crash Bandicoot, hoàn thành 3DMark FireStrike. **Cảnh báo chính chủ: "very unstable".** Đáng theo dõi, **không đáng đặt cược MVP**.

**Hệ quả:** screenshot chụp từ VM tự dựng là ảnh do llvmpipe/software renderer vẽ. Dùng để bắt bug **layout, DPI, font metric, tray icon, packaging** — **không** dùng để đánh giá màu sắc, anti-aliasing, animation, hiệu năng, hay bug driver đồ hoạ. Và Chromium/Electron dưới llvmpipe hay tự tắt renderer hoặc crash → phải doc sẵn `--disable-gpu --use-gl=swiftshader --no-sandbox`.

## 1.6 Rosetta for Linux ≠ Rosetta 2 — hai thứ đã tách đôi

Đây là điểm mà gần như mọi bài viết ngoài kia đều nhầm:

| | Rosetta 2 (app macOS Intel) | Rosetta for Linux (binary Intel trong Linux VM) |
|---|---|---|
| macOS 27 (9/2026) | **Bản cuối full support**; tự động gỡ khi nâng cấp | **Được tích hợp thẳng vào macOS**, không cần cài Rosetta nữa. Apple doc: *"macOS 27 directly integrates support for Intel binary translation, without needing to install Rosetta"* |
| macOS 28 (thu 2027) | Chỉ giữ **một subset** cho game cũ | Chưa có tuyên bố gỡ bỏ |

**Đọc trung thực:** hai mảng research độc lập đưa ra hai đánh giá khác nhau về khả năng sống sót của Rosetta for Linux (một bên "khả năng cao", một bên "độ tin cậy Thấp"). Sự thật kiểm chứng được là: Apple đã tách nó ra khỏi Rosetta 2 và tích hợp vào OS ở macOS 27 — tín hiệu tích cực, nhưng **đừng xây kiến trúc mà mất Rosetta là chết**.

Giới hạn kỹ thuật: đạt **~70-80% native**, nhanh hơn QEMU TCG 3-20x, nhưng **KHÔNG có AVX/AVX2/AVX512** (binary dùng AVX sẽ SIGILL), **không chạy binary 32-bit x86**, và **không boot được distro Linux x86-64** — chỉ chạy app Intel *bên trong* distro ARM. Cùng cảnh báo áp dụng cho `docker --platform linux/amd64` và `container run --arch amd64`: bạn có **userspace x86-64 trên kernel arm64**, `uname -m` sẽ nói dối, syscall và hành vi kernel là arm64.

## 1.7 Ma trận phần cứng/OS là một phần của sản phẩm, không phải chi tiết

| Ràng buộc | Chi tiết |
|---|---|
| Nested virtualization | **Chỉ M3+ và macOS 15+.** M1/M2 **vĩnh viễn** không có. Và tính đến 2026 Parallels vẫn chưa bật được cho Windows ARM guest |
| `com.apple.vm.networking` (bridged) | **BỊ Apple hạn chế**, phải xin duyệt qua "Apple representative". `com.apple.security.virtualization` và `.hypervisor` thì **không** bị hạn chế |
| virtiofs auto-mount | Hạn chế của Apple là: **app host không thể script lệnh mount vào một guest tuỳ ý đang chạy**. Với Linux guest bạn kiểm soát image thì **không cần agent**: bake tag virtiofs vào `/etc/fstab` hoặc một systemd `.mount` unit trong golden image, mount tự động lúc boot. Guest agent chỉ cần cho mount động sau khi boot |
| macOS 27 (9/2026) | Thêm **USB passthrough thật**, `VZCustomVirtioDevice`, DiskImageKit ASIF stacked images, guest provisioning. **Nhưng ra sau MVP của bạn** — install base ~0 trong nhiều tháng. Đừng target API macOS 27-only |
| 2 macOS VM đồng thời | Enforce ở kernel level. Không áp dụng cho Linux guest |

Ghi chú: daemon dài hạn sở hữu `VZVirtualMachine` là **làm được** (Tart, driver `vz` của Lima, VirtualBuddy, `apple/container` đều làm). Thứ không làm được là **truyền object `VZVirtualMachine` qua XPC** — daemon phải sở hữu VM và expose API IPC của riêng nó.

## 1.8 Bãi mìn giấy phép — 5 điều nhớ thuộc lòng

| Thứ | Giấy phép | Ý nghĩa |
|---|---|---|
| **Tart** | **FSL-1.1-ALv2**, nay là `openai/tart` | **CẤM** dùng để tạo sản phẩm cạnh tranh trong 2 năm. Không fork, không vendor, không "tham khảo code" |
| **QEMU** | GPLv2 | Không link tĩnh vào app closed-source. Phải ship **binary rời không sửa đổi**, giao tiếp qua QMP socket (mere aggregation), kèm **written offer for source** |
| **Lima v2.2** | Apache-2.0 | **An toàn để đọc và học.** Đã có Windows guest + swtpm + unattend + auto virtio driver (07/2026). Là blueprint hợp pháp gần như hoàn chỉnh |
| **virtio-win ARM64** | BSD-3 (một phần GPL) | Chỉ **attestation-signed**, KHÔNG WHQL. Nếu Microsoft siết driver signing trên ARM, luồng cài Windows guest vỡ. Phải có plan B |
| An toàn khác | Apache-2.0 / MIT / BSD | vfkit, libkrun, Code-Hex/vz, VirtualBuddy, objc2-virtualization, EDK2, **apple/container + apple/containerization** |

**Thêm:** Mac App Store **bất khả thi** (cần hypervisor entitlement; UTM đã phải ra bản SE không acceleration). Con đường là **Developer ID + notarization + phân phối trực tiếp** — tức Apple Developer Program **$99/năm (~2,5 triệu VND)** là chi phí bắt buộc trước đồng doanh thu đầu tiên.

## 1.9 "X11 session bị bỏ" ≠ "X11 chết" — và bạn VẪN chụp được màn hình tự động trên Linux

Bản trước kết luận sai. Sửa lại cho đúng, vì kết luận cũ ("không thể chụp màn hình tự động trên Linux") đã loại nhầm cả một lớp giải pháp $0.

Trình tự sự kiện đúng:

| Mốc | Chuyện thật sự xảy ra |
|---|---|
| GNOME 49 (9/2025) | **KHÔNG xoá code X11.** gnome-shell/mutter vẫn build được với backend X11; chỉ có việc GDM không còn hiển thị "GNOME on Xorg" mặc định. Sau phản hồi cộng đồng, khả năng khởi chạy session X11 vẫn được giữ qua tuỳ chọn build/cấu hình distro |
| GNOME 50 (chu kỳ 3/2026) | Nơi việc **xoá code X11 session** khỏi mutter/gnome-shell được lập kế hoạch. Độ tin cậy: **Trung bình** — không xác minh lại được bằng nguồn trực tuyến |
| Ubuntu 26.04 LTS (4/2026) | Đi theo hướng Wayland-only cho phiên GNOME. Độ tin cậy: **Trung bình** |

Điểm quan trọng nhất, và là chỗ bản trước lẫn lộn: **XWayland vẫn tồn tại.** `xdotool`, `import` (ImageMagick), `xwd`, `ffmpeg -f x11grab` vẫn chạy bình thường với **ứng dụng X11**. Cái mất đi là khả năng dùng chúng để chụp/điều khiển **toàn màn hình của compositor Wayland** — không phải mất X11 client.

**Kết luận thực hành cho automation trong VM/container Linux: đừng dùng GNOME chút nào.**

```bash
# Đường chuẩn: headless, không dialog xin phép, không phụ thuộc phiên bản GNOME
xvfb-run -s "-screen 0 1920x1080x24" -- \
  dbus-run-session -- /opt/myapp/myapp &
sleep 5
DISPLAY=:99 import -window root /artifacts/shot.png
```

Các lựa chọn khác không cần dialog:

| Cách | Ghi chú |
|---|---|
| `Xvfb` / `xvfb-run` + `import`/`xwd`/`x11grab` | Đơn giản nhất, chạy trên VPS không GPU, không phụ thuộc DE |
| compositor wlroots headless (`sway --headless`, `labwc`, `wayfire`) + `grim` / `wf-recorder` | Khi cần đường Wayland thật |
| `weston --backend=headless` | Reference compositor, ổn định cho CI |
| `ydotool` (qua `/dev/uinput`) | Giả lập input trên Wayland khi `xdotool` không với tới được |

Chỉ khi bạn **bắt buộc** chạy bên trong một GNOME/Wayland session thật thì mới phải qua `xdg-desktop-portal` (Screenshot/ScreenCast) — nơi có dialog xin phép; giảm nhẹ bằng `restore_token` để chỉ hỏi lần đầu, hoặc D-Bus `org.gnome.Shell.Screenshot` khi được phép. Đó là lane `desktop` (opt-in, fragile), không phải lane mặc định.

Độ tin cậy: **Cao** cho cơ chế kỹ thuật (Xvfb/XWayland/portal); **Trung bình** cho mốc phiên bản GNOME 50 và GNOME trong Ubuntu 26.04.

## 1.10 Windows 10 đã hết đời — ma trận target rút gọn còn Windows 11

**Windows 10 EOL 14/10/2025.** Không còn security update cho bản consumer, không còn là target hợp lý cho một app desktop ship năm 2026. Nói thẳng điều này vì nó xoá bớt một chiều của cả ma trận test và của mọi bảng chi phí.

Nhân tiện, đính chính một luận điểm sai hay được lặp lại: "Windows 10 on ARM không bao giờ có x64 emulation" là **sai** — Microsoft đã ship x64 emulation cho Windows 10 on ARM từ bản preview 12/2020. Lập luận đúng và mạnh hơn không nằm ở khả năng kỹ thuật mà ở vòng đời: **Windows 10 không còn là target**, nên câu hỏi Prism-hay-không trên Win10 không cần trả lời nữa.

Còn lại phải quan tâm: Windows 11 x64 (đại đa số user), Windows 11 ARM64 (Copilot+ PC, đang tăng), Windows Server 2022/2025 (chính là `windows-latest` trên GitHub Actions — **không phải** Windows 11 client, đừng nhầm hai thứ khi đọc verdict CI).

## 1.11 Ngân sách phần cứng: RAM, ổ cứng, pin, nhiệt

Đây là ràng buộc quyết định **bạn có được phép chọn phương án VM local hay không**, và nó phải nằm ở phần sự thật nền tảng chứ không nằm rải rác trong từng concept.

### RAM

| Cấu hình Mac | Chạy được gì |
|---|---|
| **8GB** | **KHÔNG chạy được VM Windows ARM.** Guest cần 6-8GB riêng cho nó, host còn lại không đủ cho Xcode/Node/trình duyệt. Với 8GB, ba đường còn lại là: container `linux/amd64`, CI (GitHub Actions), và máy Windows thật |
| **16GB** | Là **sàn**, và vẫn chật: Docker Desktop mặc định ăn 4-8GB, cộng Xcode/Node/Electron dev server. Chạy được một VM Windows ARM 8GB nếu đóng gần hết mọi thứ khác — tức không code song song được |
| **24-32GB** | Mới thoải mái: VM Windows 8-12GB + toolchain + trình duyệt cùng lúc |

### Ổ cứng

| Khoản | Dung lượng |
|---|---|
| Windows 11 ARM VM | 30GB tối thiểu (Microsoft), **thực tế 64GB** sau update + page file |
| VS Build Tools trong guest | +10-20GB |
| Overlay/snapshot mỗi lane | +vài GB đến vài chục GB tuỳ số lane |
| Golden image Linux + rootfs distro | +5-20GB |

**MacBook 256GB coi như bị loại khỏi mọi phương án VM local.** 512GB là sàn thực tế nếu muốn giữ một VM Windows; 1TB nếu muốn nhiều lane.

Và một khoản không hiện trên hoá đơn: **SSD MacBook hàn cứng, không thay được.** Vòng lặp `suspend`/`savevm` 8GB mỗi 5 phút = ~160GB ghi/ngày = **~50TB/năm**. Con số này áp dụng cho **mọi** phương án VM local có suspend/snapshot thường xuyên, không riêng một concept nào.

### Pin và nhiệt

- VM Windows chạy nền làm **pin tụt nhanh 2-4x** so với vòng lặp code thuần trên Mac.
- **MacBook Air không có quạt** → throttle trong các run dài (build trong guest, test suite 20 phút, soak test).

**Kết luận hành động — áp dụng cho toàn bộ tài liệu:** mọi thứ chạy lâu (build trong guest, test suite dài, VM thường trực, soak test) nên đẩy sang **VPS hoặc máy thật**, không chạy trên chính chiếc máy bạn đang code. Điều này củng cố kiến trúc ở §4: backend đầu tiên là một máy ở xa, VM local là backend cuối cùng chứ không phải đầu tiên.

---

[← Mục lục](README.md) · [Mục lục](README.md) · [2. Bản đồ lựa chọn →](02-ban-do-lua-chon.md)
