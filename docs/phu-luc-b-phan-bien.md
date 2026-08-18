# Phụ lục B — Bản phản biện gốc

> Khoảng 40 lỗi và thiếu sót mà critic tìm ra trong bản tổng hợp đầu tiên. Giữ nguyên để đối chiếu.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

Tài liệu này ở mức tốt hiếm thấy: phần lớn ràng buộc kỹ thuật nền (VZ không boot Windows, Prism chỉ user-mode, không DX12, QMTH, Tart FSL) đều **đúng và kiểm chứng được**. Tôi đã verify lại một số con số: giá GitHub Actions ($0.006/$0.010/$0.062 từ 01/01/2026, phí self-hosted $0.002/phút hoãn vô thời hạn) — **đúng**; Windows 11 Enterprise Evaluation có bản **ARM64, 90 ngày, không key** — **đúng**; Parallels vẫn trần DX11.1 + OpenGL 3.3, chưa có DX12 tính đến 02/2026 — **đúng**; VMware Fusion 26H1 vẫn free kể cả thương mại, chưa bị khai tử — **đúng**.

Dưới đây là các điểm cần sửa/bổ sung.

---

## A. KHẲNG ĐỊNH KỸ THUẬT SAI

**A1. GNOME 49 KHÔNG xoá hoàn toàn code X11 — đây là lỗi nặng nhất, và đang bị gắn nhãn "độ tin cậy Cao, đã kiểm chứng 08/2026".**
Thực tế: GNOME 49 (9/2025) chỉ **tắt X11 session mặc định trong GDM**, và sau đó **khôi phục khả năng khởi chạy X11 session** vì không tách được code launcher khỏi legacy integration. Việc **xoá code** diễn ra ở **GNOME 50** (branch hoàn tất ~11/2025). Sửa lại §1.5, §3-Hạng 1, §3-Hạng 4 và Phụ lục A. Ngoài ra tài liệu đang lẫn "X11 session bị bỏ" với "X11 chết": **XWayland vẫn tồn tại**, app X11 vẫn chạy; cái mất là khả năng dùng `import`/`xdotool` để chụp/điều khiển **toàn màn hình compositor**, không phải mất X11 client.

**A2. Hetzner CX22 €3.79/tháng — giá đã cũ VÀ sai khu vực cho user VN.**
Sau đợt tăng giá 04/2026, CX22 ≈ **€4.35/tháng**. Quan trọng hơn: dòng **CX/CAX là EU-only** — Hetzner Singapore chỉ có dòng **CPX**. §10.4 và §10.1 đang khuyên một dev ở VN đẩy installer 200MB qua ~250-300ms RTT tới Falkenstein. Sửa thành: Hetzner Singapore CPX22, hoặc Vultr/DigitalOcean/Linode Singapore (~$5-6, ~30-60ms từ VN), hoặc VPS trong nước.

**A3. §1.5 "Không có GPU acceleration cho guest — cả Linux lẫn Windows" mâu thuẫn trực tiếp với §2.**
§2 ghi Parallels có DX11.1 + OpenGL 3.3 (và VMware Fusion cũng có 3D cho Windows ARM guest). Phải thu hẹp khẳng định thành: *"trên VZ và trên QEMU+HVF (tức là mọi stack bạn tự xây) không có 3D accel; Parallels/VMware có driver paravirtual 3D riêng, trần ở DX11.1"*. Hiện tại câu này ở dạng tuyệt đối là sai.

**A4. "Kiến trúc daemon không chạy được vì `VZVirtualMachine` không `NSSecureCoding`" — kết luận sai, và mâu thuẫn với chính §5.1.**
Tart, driver `vz` của Lima, VirtualBuddy, và chính `container` của Apple đều chạy VZ VM trong process nền dài hạn. Cái không làm được là **truyền object `VZVirtualMachine` qua XPC**; bạn hoàn toàn có thể để daemon **sở hữu** VM và expose API IPC của riêng mình. §5.1 làm đúng điều đó với `firstrun-vzd` — nên §3 phải sửa lại, nếu không tài liệu tự bác bỏ kiến trúc của mình.

**A5. §1.7 "virtiofs auto-mount: bắt buộc phải có guest agent" — overstated cho Linux guest.**
Bake tag virtiofs vào `/etc/fstab` hoặc một systemd `.mount` unit trong golden image là xong, mount tự động lúc boot, không cần agent, không cần host script gì. Hạn chế Apple nêu là về việc script mount vào một guest **tuỳ ý/đang chạy**.

**A6. §5.5 "clonefile ~5ms + boot" gây hiểu nhầm.**
Thời gian reset thực tế bị chi phối bởi boot (~15-20s cho Linux VM), không phải 5ms clone. Ghi con số end-to-end, đừng ghi con số của bước rẻ nhất.

**A7. Rule #2: "Windows 10 on ARM không bao giờ có x64 emulation" — không chính xác** (Microsoft có ship x64 emulation cho Win10 on ARM từ preview 12/2020). Lập luận đúng và mạnh hơn: **Windows 10 đã EOL 14/10/2025**, nên nó không còn là target — điều này nên nói thẳng ở §1 vì nó đơn giản hoá cả ma trận và tài liệu chưa nhắc lần nào.

**A8. §8.3 "nếu làm managed pool thì CHỈ Windows Server" — hẹp hơn thực tế.**
Từ 2022 Microsoft cho phép chạy Windows 11 với license của chính khách trên **dedicated hardware** qua Authorized Outsourcer; thứ bị cấm là hardware **multi-tenant/dùng chung**. Nên nêu đúng để không tự đóng một cửa còn mở.

**A9. Giá perpetual Parallels — đã giải quyết được, đừng để "nguồn mâu thuẫn".**
Đúng là: **Standard $219.99 perpetual hoặc $99.99/năm; Pro $119.99/năm và Business $149.99/năm chỉ có subscription**. Bỏ con số $129.99.

---

## B. LỰA CHỌN BỊ BỎ SÓT (quan trọng nhất)

**B1. Windows VM trên cloud tính theo GIỜ — §2 khẳng định SAI rằng "Không có API spin up/tear down theo build".**
Azure B2s Windows ≈ $0.10/giờ, AWS EC2 t3.medium Windows ≈ $0.06-0.09/giờ, GCP tương tự — **license Windows gộp trong giá**, API spin-up/tear-down đầy đủ, snapshot đầy đủ, hợp pháp 100%, không dính QMTH. 20 giờ/tháng ≈ **$2**. Đây là đường x64 thật rẻ nhất cho nhu cầu tương tác thỉnh thoảng và nó đang bị gộp nhầm vào rổ "per-user-per-month" của Windows 365/Shadow.

**B2. Automation UI thật, không phải screenshot — thiếu hoàn toàn.**
Playwright có `_electron.launch()`, WebdriverIO có electron-service: assertion thật, cross-platform, headless, $0, chạy được ngay trên Mac và trên mọi CI. Flutter có `integration_test`/`flutter drive`; MAUI/WinUI có Appium Windows Driver / FlaUI. Toàn bộ §5/§10 đang cược vào "poll screenshot + pixel-stability", một cơ chế yếu và flaky hơn nhiều so với thứ đã tồn tại miễn phí. Phải có một mục so sánh và nói rõ khi nào screenshot mới là công cụ đúng.

**B3. Apple `container` / Containerization framework (v1.0.0, 06/2026, Apache-2.0, macOS 26) — không xuất hiện một lần nào.**
Đây là runtime container Linux **first-party** của Apple, mỗi container một micro-VM trên VZ, có Rosetta cho amd64, boot dưới 1 giây. Nó liên quan trực tiếp tới §10.2 (thay thế Docker Desktop/OrbStack), tới backend `local-vz` (có thể dùng thay vì tự viết Swift), và tới §1.8 (Apache-2.0 = an toàn để dùng/đọc). Bỏ sót cái này là bỏ sót một trong hai thứ mới nhất của năm.

**B4. Windows Sandbox** — clean room tức thì, có sẵn trên Windows Pro/Enterprise, reset khi đóng. Cho smoke test installer trên máy x64 thật, nó đơn giản hơn nhiều so với vòng Hyper-V checkpoint ở §10.6.

**B5. Máy Windows ARM thật (Snapdragon X Copilot+ PC, ~$600-800, refurb rẻ hơn).**
Rule #12 bảo user phải quan tâm `windows-11-arm`, nhưng tài liệu không đưa ra một cách nào để **nhìn thấy** nền tảng đó trên phần cứng thật. Nên có một dòng.

**B6. Laptop Windows cũ 2-4 triệu VND thay vì mini PC N100 $250.**
Có sẵn màn hình, bàn phím, và **pin = UPS tích hợp** (rất đáng giá ở VN), thường kèm license OEM. Cũng nên nhắc Dell OptiPlex / HP EliteDesk micro refurb ~1.5-3 triệu, mạnh hơn N100.

**B7. Oracle Cloud Always Free** — 2 instance AMD x86 + 4 core ARM/24GB, miễn phí vĩnh viễn. Cùng với GitHub Student Pack / Microsoft for Startups / Azure credits, đây là đường x86-64 Linux **$0 thật sự** mà §10 chưa nhắc.

**B8. Cross-compile — tài liệu kết luận "không được" nhưng chỉ xét node-gyp và cargo-xwin.**
Thiếu: image chính chủ **`electronuserland/builder:wine`** (build installer NSIS/MSI Windows từ macOS/Linux, đây là đường electron-builder khuyến nghị), **`osslsigncode`** để ký Authenticode từ macOS, Go cross-compile `GOOS=windows` với `CGO_ENABLED=0`, và `zig cc`/MinGW-w64 cho C/C++. Kết luận "phải build trong guest" chỉ đúng cho native module + MAUI/Flutter, không đúng phổ quát.

**B9. Đường license Windows hợp pháp còn thiếu hai lựa chọn.**
(a) **Visual Studio Professional/Enterprise subscription** kèm license Windows cho mục đích dev/test — đây là câu trả lời chuẩn của ngành cho "dev muốn VM Windows", hoàn toàn vắng mặt ở §1.4. (b) **Windows Insider ARM64 VHDX / ISO** — miễn phí, hợp pháp, không giới hạn 90 ngày như bản Evaluation.

**B10. Telemetry/crash reporting là lựa chọn ROI cao nhất và không được nhắc.**
Sentry/Crashpad + electron-log (free tier đủ dùng) cho bạn **bug thật, trên máy thật, của user thật** — thứ mà không VM nào trên Apple Silicon cho được. Với một dev solo Mac-only, đây có thể là hành động đúng số 1, trên cả mini PC. Tương tự: **kênh beta với 5-10 tình nguyện viên dùng Windows/Linux** — đó là cách phần lớn dev cross-platform solo thực sự đang làm, chi phí $0, fidelity 100%.

---

## C. LỖI CỤ THỂ TRONG PHẦN "LÀM NGAY HÔM NAY" (§10) — script hiện tại chưa chạy được

**C1. §10.4 sai chắc chắn:** `--bind-ro=/artifacts` là read-only nhưng script ghi `shot.png` vào `/artifacts` → lệnh fail. Đổi sang `--bind=/artifacts` hoặc thêm một bind rw riêng cho output.

**C2. §10.4 thiếu bước tạo btrfs subvolume.** `--ephemeral` chỉ nhanh khi rootfs là **subvolume** btrfs, không phải chỉ nằm trên filesystem btrfs. Thêm `btrfs subvolume create /var/lib/machines/ubuntu2604` **trước** khi debootstrap, nếu không nó âm thầm fallback sang copy toàn bộ (30-90s thay vì 2s).

**C3. §10.4 `debootstrap ... resolute` sẽ fail** trên host có debootstrap chưa biết codename mới — thêm `ln -s /usr/share/debootstrap/scripts/gutsy /usr/share/debootstrap/scripts/resolute`. Và verify codename 26.04 trước khi in vào doc.

**C4. §10.4 thiếu DNS và FUSE trong container:** thêm `--resolv-conf=copy-host` và bind `/dev/fuse` (cho AppImage). Ngoài ra `apt-get update` mỗi lần ephemeral tốn 20-40s → pre-bake dependency vào rootfs, nếu không lời hứa "máy sạch trong 2 giây" là không có thật ở end-to-end.

**C5. §10.4 rootfs Fedora bằng `dnf --installroot` từ host Ubuntu rất khó chịu** (thiếu dnf, thiếu GPG key). Viết thẳng lệnh dùng `podman run fedora:44` thay thế, hoặc bỏ Fedora khỏi ví dụ.

**C6. §10.2 Docker thiếu `--shm-size=2g`** → Chromium/Electron crash với /dev/shm mặc định 64MB. Đây là lỗi phổ biến nhất khi chạy Electron trong container và sẽ làm user tưởng app của họ hỏng.

**C7. §10.2 thiếu session D-Bus.** Cài `dbus-x11` nhưng không chạy `dbus-run-session` / `eval $(dbus-launch)` → nhiều app GTK/Electron treo hoặc mất tray. Bọc lệnh launch trong `dbus-run-session --`.

**C8. §10.2 `echo "exit=$?"` báo exit code của `import`, không phải của app.** Bug logic, và nó là dòng duy nhất trong ví dụ được dùng làm verdict.

**C9. §10.2 thiếu cảnh báo quan trọng nhất:** `--platform linux/amd64` cho bạn **userspace x86-64 trên kernel arm64**. `uname -m` sẽ nói dối, syscall/kernel behavior là arm64, không có AVX (Rosetta), không chạy binary 32-bit. Nó bắt bug packaging/dependency, **không** bắt bug nhạy cảm với kernel/arch.

**C10. §10.6 lệnh `ssh minipc 'Start-Process ...'` sẽ FAIL.** Default shell của OpenSSH trên Windows là `cmd.exe`, không phải PowerShell. Thiếu bước:
`New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force`

**C11. §10.6 `scp dist/... minipc:/Users/you/Desktop/` dùng đường dẫn macOS.** Trên Windows phải là `minipc:C:/Users/<user>/Desktop/`.

**C12. §10.6 giả định ngầm Windows 11 Pro.** `Enable-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V` **không có trên Windows 11 Home**, mà phần lớn mini PC N100 $250 ship Home. Toàn bộ câu chuyện "reset bằng Hyper-V checkpoint" (và cả backend `winmetal` tháng 2) sụp ở đây. Thêm: kiểm tra edition trước khi mua, hoặc cộng thêm tiền nâng Pro, hoặc fallback bằng Macrium Reflect image restore / ổ thứ hai.

**C13. §10.3 thiếu `timeout-minutes`** trên mỗi job — một app GUI treo sẽ ăn hết quota (mặc định 6 giờ). Và bitmap screenshot hardcode 1920x1080 trong khi desktop của runner có thể là 1024x768.

**C14. §10.3 `SubmitSamplesConsent SendAllSamples` sẽ upload installer chưa release của bạn lên Microsoft.** Đây là hệ quả thật, chưa nói ra. Với sản phẩm thương mại/NDA thì đây là vấn đề — phải cảnh báo hoặc dùng `SendSafeSamples`.

**C15. §10.3 thiếu cơ chế nhân hệ số của free tier.** Private repo: 2.000 phút Linux-equivalent/tháng, nhưng **Windows tính 2x, macOS 10x**. Workflow như đang viết có thể đốt hết quota Free plan rất nhanh. Public repo mới là unlimited.

**C16. §10.4 muốn "nhìn tận mắt" thì không cần xrdp + XFCE.** `x11vnc -display :99 -forever -localhost` + SSH tunnel là 1 dòng, không cài DE, và xem đúng cái Xvfb mà test đang dùng (tránh lệch môi trường giữa lúc test và lúc xem).

---

## D. GÓC ĐỘ BỊ THIẾU HẲN

**D1. BẢO MẬT — container không phải security boundary.**
`systemd-nspawn`/Docker **chia sẻ kernel host**. Chạy một installer chưa ký/không rõ nguồn gốc trong đó = đặt cả VPS (và mọi artifact của khách trên đó) vào rủi ro. Phải ghi thẳng: binary không tin cậy → VM dùng một lần, không phải container. Tài liệu nói về "chia sẻ kernel" nhưng chỉ ở góc độ *fidelity*, không ở góc độ *bảo mật*.

**D2. BẢO MẬT — VPS ở §10.4 đang chạy agent với quyền root, SSH public IP.** Thiếu: `PasswordAuthentication no`, key-only, ufw chặn mọi thứ trừ 22, và **tuyệt đối không expose xrdp/VNC ra internet** (3389 mở sẽ bị quét trong vài phút) — dùng Tailscale/WireGuard. §10.6 đã dùng Tailscale, §10.4 thì không, thiếu nhất quán.

**D3. BẢO MẬT — token và dữ liệu nhạy cảm.** Backend `gha` cần PAT scope `workflow` = quyền ghi repo; phải khuyến nghị fine-grained token hoặc GitHub App. Và screenshot/log rất dễ chứa API key, email khách, token trong URL → cần bước redaction trước khi lưu vào `runs/`. Cài agent bằng `curl | sh` cần pin checksum/chữ ký.

**D4. RAM — không có bảng spec tối thiểu, đây là thiếu sót lớn với một tài liệu cho dev Mac.**
Cần nói thẳng: **Mac 8GB không chạy được VM Windows ARM** (guest cần 6-8GB riêng) — với 8GB chỉ có container `linux/amd64` + CI + máy thật. 16GB là sàn và vẫn chật khi có Docker Desktop (mặc định ăn 4-8GB) + Xcode/Node. 24-32GB mới thoải mái. Hiện chỉ có một dòng "M1+ ≥16GB" nằm lẫn trong concept Cuaso.

**D5. Ổ CỨNG — không có ngân sách dung lượng.**
Windows 11 ARM VM: 30GB tối thiểu, thực tế 64GB; +VS Build Tools 10-20GB; + overlay mỗi lane; + golden image. MacBook 256GB coi như bị loại khỏi mọi phương án VM local. Con số 50TB ghi/năm chỉ xuất hiện ở phần Cuaso, trong khi nó áp dụng cho mọi phương án VM và SSD Mac hàn cứng không thay được.

**D6. PIN & NHIỆT.** Chạy VM Windows làm pin MacBook tụt nhanh 2-4x; MacBook Air không quạt sẽ throttle trong các run dài. Kết luận hành động: mọi thứ chạy lâu nên đẩy sang VPS/mini PC, không chạy trên máy đang code. Tài liệu chưa có một dòng nào về việc này.

**D7. ĐIỆN — mini PC ở VN.** N100 ~7-15W idle, ~25W tải → 5-10 kWh/tháng ≈ **15-35k VND/tháng** với giá điện bậc thang. Không đáng kể, nhưng nên nói kèm Wake-on-LAN để không phải bật 24/7.

**D8. Thiếu quy trình "tái hiện bug user báo".**
Đây là lý do phổ biến nhất khiến một dev Mac-only cần Windows, nhưng không có công thức: lấy log/version từ user, `--enable-logging`, WER LocalDumps, symbol server, crash reporter. Xem thêm B10.

**D9. Thiếu lớp bug đắt nhất trong 15 rule: auto-update.**
Squirrel/NSIS updater/delta update, file association, deep link, cài per-user vs per-machine, gỡ cài đặt còn sót, migration khi nâng version. Đây là nhóm gây thảm hoạ lớn nhất cho desktop app và không có rule nào chạm tới, cũng không có test nào trong `run`.

---

## E. ĐẶC THÙ VIỆT NAM BỊ BỎ SÓT

**E1. Vùng đặt VPS và thanh toán.** Xem A2. Bổ sung: **Hetzner yêu cầu xác minh danh tính và hay từ chối thẻ VN**. Lựa chọn thực tế cho VN: Vultr/DigitalOcean/Linode Singapore (thẻ VN dễ qua), hoặc nhà cung cấp trong nước (Vietnix, Azdigi, VNG Cloud, FPT Cloud, Viettel IDC) thanh toán VND qua chuyển khoản/Momo, ~100-300k/tháng.

**E2. Apple Developer Program $99/năm không nằm trong bất kỳ bảng chi phí nào** — nhưng nó là điều kiện bắt buộc cho Developer ID + notarization ở §5.3 và tuần 4 của MVP. Với dev VN đây là ~2,5 triệu/năm bắt buộc trước khi có đồng doanh thu đầu tiên.

**E3. Thuế và nhận tiền.** Paddle/Lemon Squeezy trả về VN qua Wise/PayPal. Phía VN: thu nhập phần mềm cá nhân/hộ kinh doanh chịu thuế khoán (khoảng 5% GTGT + 2% TNCN trên doanh thu dịch vụ phần mềm), hoặc lập công ty chịu 20% TNDN. Tài liệu mới chỉ ghi "hoá đơn điện tử". Nên có 2-3 dòng cụ thể vì đây là ma sát thật.

**E4. Bài toán "gọng kìm chi phí" của Cuaso đang tính thiếu một vế.** Mini PC/laptop cũ mua ở VN **thường đã kèm license OEM Windows** — nghĩa là đường "mini PC hợp pháp" rẻ hơn con số tài liệu đưa ra, làm kết luận "Cuaso chỉ rẻ hơn khi dùng key lậu" còn mạnh hơn nữa. Nên nói rõ.

**E5. Luận điểm "USB token giết Cuaso" cần được cập nhật — và nó cắt theo hai chiều.**
Thị trường VN đã dịch chuyển mạnh sang **ký số từ xa** (VNPT SmartCA, Viettel-CA MySign, FPT.eSign, MISA eSign) và sang kê khai web (thuedientu, eTax Mobile, hoá đơn điện tử bắt buộc). Nghĩa là: (a) biggest_risk "token kernel-mode" đã bớt chí mạng với một tỷ lệ user đang tăng; nhưng (b) cũng nghĩa là **nỗi đau "cần Windows" đang tự tiêu biến** — điều này làm yếu luận điểm thị trường của Cuaso theo một hướng tài liệu chưa nêu. Phải đưa vào danh sách giả định cần verify.

**E6. HTKK và iTaxViewer nên được kiểm tra riêng.** iTaxViewer chạy trên JRE; nếu nó chạy được trực tiếp trên macOS/Linux JVM thì câu trả lời của Compatibility Oracle cho hai truy vấn có traffic cao nhất sẽ khác hẳn. Thêm vào thí nghiệm 10 phần mềm ở §9.5.

---

## F. GHI CHÚ NHỎ

- "macOS 27 (Golden Gate)" — codename là tin đồn, nên gắn nhãn độ tin cậy Thấp thay vì dùng như dữ kiện.
- "Parallels 7 triệu user" là số marketing của chính Parallels, nên ghi nguồn.
- §5.3 "Go 1.24" — tới 08/2026 đã là Go 1.26/1.27; sửa hoặc bỏ số version.
- §1.3 "`GetNativeSystemInfo` trả về thông tin CPU ảo" — chính xác hơn: với process x64 chạy dưới emulation, nó trả về `PROCESSOR_ARCHITECTURE_AMD64` (cố tình nói dối để tương thích); chỉ `IsWoW64Process2`/`GetMachineTypeAttributes` lộ ARM64.
- §10.2 thiếu `fonts-noto-cjk` nếu app có UI CJK; `fonts-noto-core` đủ cho tiếng Việt nhưng không đủ cho Trung/Nhật/Hàn.
- Phụ lục A nên hạ độ tin cậy dòng GNOME 49 từ "Cao — đã kiểm chứng" xuống "Sai — đã bác bỏ" (xem A1), và thêm dòng cho giá Hetzner (xem A2). Việc một dòng gắn nhãn "Cao, đã kiểm chứng 08/2026" lại sai là rủi ro uy tín lớn nhất của chính tài liệu này, vì toàn bộ giá trị của nó nằm ở kỷ luật trung thực.

Sources: [GitHub Actions pricing changes](https://github.com/resources/insights/2026-pricing-changes-for-github-actions), [Windows 11 Enterprise Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise), [X11 Session Removal FAQ – GNOME](https://blogs.gnome.org/alatiera/2025/06/23/x11-session-removal-faq/), [GNOME 49 Restores Ability to Launch X11 Desktop Sessions](https://www.omgubuntu.co.uk/2025/09/gnome-49-reenables-x11-session-support-in-gdm), [VMware Workstation and Fusion 26H1](https://blogs.vmware.com/cloud-foundation/2026/05/14/announcing-vmware-workstation-and-fusion-26h1/), [Parallels Desktop pricing](https://www.parallels.com/products/desktop/buy/), [KB Parallels: Limitations of running Windows 11 on Apple silicon](https://kb.parallels.com/en/129497), [Hetzner Cloud Singapore location](https://www.hetzner.com/pressroom/new-location-singapore/), [Hetzner 2026 price increases](https://northflank.com/blog/hetzner-cloud-server-price-increases), [apple/containerization](https://github.com/apple/containerization)

---

[← Phụ lục A — Kết quả kiểm chứng lại](phu-luc-a-kiem-chung.md) · [Mục lục](README.md) · [Mục lục ↑](README.md)
