# 5. Rủi ro và mô hình kinh doanh

> Ba bảng rủi ro (thị trường, kỹ thuật, pháp lý), chi phí khởi động thật, bảng giá, thuế cho dev ở Việt Nam, và dự phóng tỉnh táo.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

## 8. Rủi ro & cách giảm thiểu

Ba bảng dưới đây dùng chung thang: **Xác suất** (Thấp / Trung bình / Cao / Rất cao / Đã xảy ra / Chắc chắn) và **Tác động** đo bằng thứ mất đi, không phải bằng tính từ.

### 8.1. Rủi ro sản phẩm / thị trường

| Rủi ro | Xác suất | Tác động | Giảm thiểu |
|---|---|---|---|
| **Không ai biết đến** (indie solo, không audience, ngách trong ngách) | **Rất cao** | Chết | Đây là kịch bản khả dĩ nhất và **không liên quan gì đến chất lượng sản phẩm**. Phân phối phải ký sinh vào hệ sinh thái (PR vào doc electron-builder/tauri-action, một GitHub Action, gói npm), không phải landing page. Demo phải là **ảnh nhìn được** (tray icon biến mất side-by-side), không phải terminal in PASS |
| **Tần số dùng quá thấp** để nuôi subscription (4-8 lần/năm) | **Cao** | Churn có hệ thống | (1) `scan` chạy mỗi push (3 giây, static) → tần số cao; (2) PR bot ở tháng 5-6; (3) **đo median run/user/tháng từ tuần 4**; (4) nếu < 4 → đổi sang annual-only hoặc one-time, không phải đổi giá |
| **Free tier ăn thịt bản Pro** | Cao | Doanh thu ~0 | Paywall theo **TARGET**, không theo tính năng: `run` trên 1 target = free (kênh phân phối, vào thẳng Makefile của user); trả tiền khi chạy **ma trận nhiều target song song** + remote target + lịch sử run. Đừng khoá `run` — đó là primitive cần được phát tán miễn phí |
| **Mini PC / laptop cũ thắng** | Cao | Mất khách WTP cao nhất | **Đầu hàng có chủ đích: biến nó thành backend** (`winmetal`, tháng 2). Và phải nói thẳng con số thật: máy cũ mua ở VN **thường đã kèm license OEM Windows**, nên đường "máy x86-64 thật, hợp pháp" rẻ hơn mọi bảng tính trong tài liệu này — laptop cũ 2-4 triệu hoặc OptiPlex/EliteDesk micro refurb 1,5-3 triệu, đã có Windows, có màn hình, có pin làm UPS. Đối thủ này mạnh hơn ta tưởng, không yếu hơn |
| **Lima thêm `limactl run --artifact`** | Trung bình | Mất 50% giá trị Lớp 1 | Lima là CNCF, có nhân lực tài trợ, miễn phí vĩnh viễn. Không cược vào việc họ không làm. Giá trị phải nằm ở **corpus + rule engine + fidelity KB**, không ở orchestration plumbing |
| **Apple/Parallels thêm automation** | Thấp-TB | Mất 60% Lớp 1 | Không phòng thủ được. Đi nhanh, giá thấp, giữ quan hệ trực tiếp với 500 khách đầu |
| **Nỗi đau "cần Windows" ở thị trường VN đang tự tiêu biến** (ảnh hưởng trực tiếp §9.5) | **Trung bình-Cao** | Mất một nửa luận điểm của đường doanh thu thứ hai | Thị trường đã dịch chuyển mạnh sang **ký số từ xa** (VNPT SmartCA, Viettel-CA MySign, FPT.eSign, MISA eSign) và **kê khai qua web** (thuedientu, eTax Mobile, hoá đơn điện tử bắt buộc). Cắt hai chiều: (a) rủi ro "driver USB token kernel-mode không bao giờ chạy dưới Prism" đã bớt chí mạng với một tỷ lệ user **đang tăng**; (b) nhưng chính điều đó làm **nhu cầu chạy Windows trên Mac teo lại** theo thời gian. Đây là giả định **phải verify trước**, không phải sau — xem danh sách ở §9.5 |
| **Gánh nặng support giết dev** | Trung bình | Bỏ cuộc (như Whisky) | Ở giá thấp, 50 email/tháng là đủ để bỏ. Doc + video + FAQ từ ngày đầu; **từ chối support ca mà `scan` đã báo trước là NOT-COVERED**; không có kênh 1-1 ở tier rẻ |
| **Trung thực phản tác dụng ngắn hạn** | Trung bình | Conversion thấp | Cơ chế bảo vệ (NOT-COVERED) làm sản phẩm trông yếu với người chưa dùng. Chấp nhận: bất lợi ngắn hạn đổi lấy moat dài hạn. Nhưng **đặt NOT-COVERED trong `result.json` và report, không đặt trên landing page hero** |
| **Chi phí cố định phát sinh trước đồng doanh thu đầu tiên** | Chắc chắn | 4-11 triệu VND năm đầu | Xem bảng §9.2. Giảm thiểu thật: **hoãn Apple Developer Program** đến khi ship bản cài cho người không dùng terminal — binary cài qua Homebrew/npm/`curl` không bị gắn thuộc tính quarantine nên chưa cần notarization; chỉ khi phát hành `.dmg`/`.pkg` mới bắt buộc |

### 8.2. Rủi ro kỹ thuật

| Rủi ro | Xác suất | Tác động | Giảm thiểu |
|---|---|---|---|
| **Chụp màn hình trong guest Linux vỡ** | Trung bình (đã bị phóng đại trong bản trước) | Mất tính năng bán hàng | **Phải phát biểu lại cho đúng, vì bản trước sai:** GNOME 49 (9/2025) **không xoá code X11** — nó chỉ bỏ "GNOME on Xorg" khỏi GDM mặc định, và sau phản hồi cộng đồng khả năng khởi chạy session X11 vẫn được giữ qua tuỳ chọn build/distro; việc **xoá code** nằm ở GNOME 50 (chu kỳ 3/2026). Quan trọng hơn: **XWayland vẫn tồn tại**, nên `xdotool`/`import`/`xwd`/`x11grab` vẫn chạy với app X11 — cái mất là chụp **toàn màn hình compositor Wayland**, không phải mất X11. Kiến trúc đúng là **không dùng GNOME chút nào**: lane `clean` chạy `Xvfb` (hoặc `xvfb-run`) + app + `import`/`xwd` trong môi trường ta kiểm soát — headless, không dialog xin phép, không phụ thuộc phiên bản GNOME. Lane Wayland: compositor wlroots headless (`sway --headless`, `labwc`, `wayfire`) + `grim`, hoặc `weston --backend=headless`. Chỉ khi buộc phải chạy **bên trong một session GNOME thật** mới phải qua `xdg-desktop-portal` — nơi có dialog — và lúc đó dùng `restore_token` để chỉ hỏi lần đầu (độ tin cậy cơ chế: Cao; mốc phiên bản GNOME 50 / Ubuntu 26.04: Trung bình, chưa xác minh online được) |
| **Container không phải security boundary** — chạy artifact chưa ký của khách trong `systemd-nspawn`/Docker | **Cao nếu không xử lý** | Mất cả VPS **và mọi artifact của mọi khách đang nằm trên đó** | Đây là góc hoàn toàn thiếu ở bản trước: tài liệu chỉ nói "chia sẻ kernel" ở góc độ *fidelity*, không ở góc độ *bảo mật*. `nspawn` và Docker **dùng chung kernel host** — một installer chưa ký, một binary từ repo lạ, một AppImage từ issue tracker đều là mã tuỳ ý chạy cạnh kernel đó. Quy tắc cứng: **binary không tin cậy → VM dùng một lần, không phải container.** Cụ thể: (1) container chỉ dùng cho artifact do **chính chủ máy** build; (2) artifact của bên thứ ba → VM ephemeral riêng, xoá sau mỗi run; (3) trong mọi trường hợp: user không phải root trong container, `--private-network` khi test không cần mạng, không bind thư mục chứa artifact của khách khác, một VPS **không** phục vụ nhiều khách |
| **VPS chạy agent quyền root, SSH mở ra IP public** | **Cao** | Mất máy trong vài giờ | Baseline bắt buộc, không phải tuỳ chọn: `PasswordAuthentication no`, key-only, `ufw` chặn mọi thứ trừ 22, agent chạy bằng user thường (chỉ `sudo` đúng lệnh cài gói qua sudoers whitelist), fail2ban. Và **tuyệt đối không expose xrdp/VNC ra internet** — cổng 3389 mở sẽ bị quét trong vài phút — dùng Tailscale/WireGuard cho mọi thứ không phải SSH. Bản trước dùng Tailscale ở phần mini PC nhưng bỏ quên ở phần VPS: phải nhất quán |
| **Tỷ lệ false positive > 10% → dev ngừng tin** | Cao | Chết như flaky test | **Không bao giờ so pixel cheo OS.** Chỉ assert bất biến phát biểu được bằng lời. Pixel diff chỉ trong cùng lane với baseline của chính nó. Dành 4-6 tuần **chỉ để tinh chỉnh ngưỡng** trên 8-10 app thật |
| **Phát hiện "app đã lên"** không có API hợp lệ trên GNOME Wayland | Đã xảy ra | Logic điều phối trung tâm vỡ | Poll screenshot + pixel-stability, DE-agnostic. Escape hatch `--wait` / `--wait-for-log`. Trong lane `clean` (Xvfb) thì `xdotool search --sync` cũng dùng được vì ta kiểm soát X server |
| **llvmpipe làm screenshot mất giá trị** (GTK4 compile shader, Chromium tự tắt renderer) | Cao | False positive | Doc sẵn `--disable-gpu --use-gl=swiftshader --no-sandbox`. Ghi vào NOT-COVERED: *"ảnh này render bằng software — dùng để bắt bug layout/DPI/tray/packaging, KHÔNG dùng để đánh giá hiệu năng, animation, hay bug driver đồ hoạ"* |
| **`windows-latest` không phải Windows 11 client, và Defender bị tắt sẵn trong image** | **Chắc chắn** | Verdict "sạch" từ CI là vô nghĩa nếu không sửa | Đã đọc trực tiếp source `actions/runner-images`: `Configure-WindowsDefender.ps1` set `DisableRealtimeMonitoring`, `DisableBehaviorMonitoring`, `DisableScriptScanning`, `ExclusionPath = C:\, D:\`, và set `ForceDefenderPassiveMode=1` **nếu** registry path ATP tồn tại. Đồng thời `windows-latest` hiện trỏ **Windows Server 2025**, không phải Windows 11 client. → Defender prescan phải tự bật lại + `Update-MpSignature` + bật cloud protection, và **không được lấy thời gian build trên CI làm chuẩn** cho máy user thật. Ghi chú thêm: `windows-11-arm` **đã mở cho cả public lẫn private repo** (không còn public-only như tài liệu cũ ghi) |
| **`savevm`/`loadvm` không hoạt động với `-accel hvf` aarch64** | **Cao** | "Warm resume 3-8s" bất khả thi | **Không hứa warm resume.** Reset Windows = cold boot 40-60s hoặc Hyper-V checkpoint trên máy thật. Kiểm tra trước tháng 7 |
| **virtiofs macOS→Windows guest không tồn tại** | **Chắc chắn** | Mất file sharing cho `qemu-win` | Agent làm HTTP file server trên `hostfwd`, hoặc mount `\\10.0.2.2\share`. Bằng chứng: Lima v2.2 phải đánh dấu template Windows là `plain: true` |
| **Không có target Windows x64 thật với chi phí hợp lý** | Trung bình | Mất verdict đáng tin nhất | Bản trước khẳng định sai rằng "không có API spin up/tear down theo build". **Có**: `az vm create/deallocate/delete` + `az snapshot create`, `aws ec2 run-instances / create-image / terminate-instances`. Azure B2s Windows ~$0.10/giờ, AWS t3.medium Windows ~$0.08/giờ, **license gộp trong giá giờ** → 20 giờ/tháng ≈ **$2 compute** (+disk hoặc snapshot, tổng thực tế ~$4-12/tháng; độ tin cậy: Trung bình-Cao cho Azure/AWS, Thấp cho GCP). Ràng buộc phải nói rõ: cái được cấp là **Windows Server**, không phải Windows 11 client |
| **macOS point update phá vỡ HVF/VZ** | Trung bình | Sản phẩm ngừng chạy | Đã có tiền lệ. `firstrun doctor` kiểm tra môi trường; update nhanh qua Sparkle; **kiến trúc SSH-first nghĩa là VM local vỡ thì sản phẩm vẫn dùng được** |
| **Token và dữ liệu nhạy cảm rò qua artifact** | Trung bình | Mất lòng tin, có thể mất repo của khách | Backend `gha` cần PAT scope `workflow` = quyền ghi repo → khuyến nghị fine-grained token hoặc GitHub App. Screenshot/log rất dễ chứa API key, email khách, token trong URL → **bước redaction bắt buộc trước khi ghi vào `runs/`**. Cài agent bằng `curl | sh` phải pin checksum + chữ ký |
| **virtio-win ARM64 mất chữ ký / Microsoft siết driver signing** | Thấp-TB | Luồng cài Windows guest vỡ | Chỉ ảnh hưởng tháng 7+. Plan B: ISO Evaluation với Secure Boot tắt trong EDK2 config |
| **Ước tính thời gian trượt** | **Rất cao** | Cạn tiền/động lực | 60% thời gian nằm ở chỗ không ai dự tính. MVP 4 tuần **chỉ có Lớp 0 + SSH backend** — không Swift, không QEMU, không entitlement |

Baseline bảo mật cho VPS, viết ra để không phải nhớ:

```bash
# /etc/ssh/sshd_config.d/99-firstrun.conf
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin prohibit-password

# chặn tất cả trừ SSH
ufw default deny incoming
ufw allow 22/tcp
ufw --force enable

# mọi thứ KHÔNG phải SSH đi qua Tailscale, không mở port ra internet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up --ssh

# muốn NHÌN màn hình test: bind localhost rồi tunnel, KHÔNG mở 5900/3389
x11vnc -display :99 -forever -localhost -nopw &
# từ Mac:
ssh -L 5900:localhost:5900 vps
```

Và quy tắc phân loại artifact, đặt thẳng trong code:

```
artifact do chính chủ máy build      → systemd-nspawn --ephemeral   (nhanh, đủ an toàn)
artifact của bên thứ ba / khách hàng → VM dùng một lần, xoá sau run (chậm hơn, đúng boundary)
```

### 8.3. Rủi ro pháp lý

| Rủi ro | Xác suất | Tác động | Giảm thiểu |
|---|---|---|---|
| **Windows licensing / redistribute** | Thấp (nếu làm đúng) | Nghiêm trọng | **KHÔNG BAO GIỜ** đóng gói ISO hay product key. Chỉ **mở trình duyệt** tới trang Microsoft, verify SHA256. Onboarding dùng **Windows 11 Enterprise ARM64 Evaluation — 90 ngày, không cần product key** (đã xác minh, độ tin cậy Cao). Hết hạn thì user tự nhập key Pro/Enterprise của họ. Cảnh báo rõ: Windows 11 **Home ARM nằm ngoài phạm vi authorized**, và **key OEM không được chuyển sang VM** — chỉ retail/FPP mới chuyển được |
| **QMTH / host Windows client cho nhiều khách** | Thấp | Chết sản phẩm | Kiến trúc **không bao giờ sở hữu máy Windows nào**. Nhưng phải nêu đúng thay vì tự đóng một cửa còn mở: từ 2022 Microsoft cho phép chạy **Windows 11 với license của chính khách trên dedicated hardware** qua Authorized Outsourcer — thứ bị cấm là hardware **multi-tenant/dùng chung**. Nghĩa là managed pool **không nhất thiết chỉ Windows Server**; đường Windows 11 client vẫn hợp pháp nếu mỗi khách có phần cứng riêng và mang license riêng. Lý do ta vẫn không làm là **kinh tế đơn vị** (một máy dedicated/khách phá vỡ biên gộp 95% ở giá $99/năm), **không phải pháp lý**. Nếu sau này thật sự làm pool: Windows Server (license gộp trong giá cloud theo giờ) là đường đơn giản nhất; Windows 11 client là đường đắt hơn nhưng không bị cấm |
| **Không có nhãn "Microsoft-authorized"** | Chắc chắn | Vùng xám khi audit | Disclosure rõ trong onboarding, một prompt xác nhận, không giấu trong ToS: *"Đây không phải giải pháp được Microsoft authorized — chỉ Parallels Desktop có. Bạn sẽ không được Microsoft support."* Sự trung thực này là **điểm khác biệt marketing thật** vì mọi bài bán Parallels ở VN đều mập mờ chỗ này |
| **GPLv2 của QEMU** | Trung bình (từ tháng 7) | Buộc mở source | Ship **binary rời không sửa đổi**, spawn process, giao tiếp qua **QMP socket** (mere aggregation). Repo công khai chứa build script + patch + source tarball, trang `/compliance`, **written offer for source**. **Tải QEMU ở bước `firstrun doctor`, không nhét trong `.pkg`.** Tương tự swtpm (LGPL), EDK2 (BSD-2-Clause-Patent). **Không link tĩnh** |
| **Tart FSL-1.1-ALv2** | Thấp | Kiện | **TUYỆT ĐỐI không fork, không vendor, không "tham khảo code"** `openai/tart`. An toàn: Lima, vfkit, libkrun, VirtualBuddy, Code-Hex/vz, virtio-win, và **`apple/container` (Apache-2.0)** |
| **Ký code Windows sau 2023** | Chắc chắn nếu ship bản Windows | Không phát hành được | Từ khi CA/B Forum bắt buộc khoá nằm trong HSM/token, **file PFX rẻ tiền không còn dùng được** cho OV/EV. `osslsigncode` ký Authenticode được từ macOS (kể cả timestamp RFC3161) nhưng chỉ khi khoá ở dạng file — với chứng chỉ HSM phải qua cloud signing (Azure Trusted Signing, DigiCert KeyLocker). Đưa chi phí này vào bảng §9.2 ngay từ đầu |
| **GitHub Actions ToS** | Trung bình | Ban tài khoản **của user** | ToS: Actions chỉ dùng cho việc liên quan đến chính repo đó. Test đúng repo = hợp lệ. Nhưng phiên tương tác (RDP/VNC tunnel) rất dễ bị đọc thành "remote desktop service". **Bắt buộc:** cap thời lượng phiên ≤20 phút, gắn cứng vào repo đang test, không cho chạy job không liên quan |
| **Redistribute image Linux đã sửa** | Thấp | Trademark | `debootstrap` từ archive chính chủ ngay trên máy target, không host lại image gốc. Chỉ host layer overlay của mình |
| **Screenshot/binary của user chứa dữ liệu nhạy cảm** | Trung bình | GDPR / mất lòng tin | **MVP: 100% local, không upload gì.** Backend `gha` dùng tài khoản của chính user → binary đi qua GitHub, không qua bạn. Nếu sau này có relay: E2E encryption client-side + TTL + không log plaintext + DPA + tuỳ chọn self-host. **Nói thẳng: mã hoá này bảo vệ khỏi CHÚNG TÔI, không bảo vệ khỏi GitHub** |
| **Quảng cáo sai sự thật** | Trung bình | Uy tín + pháp lý | Không được nói "chạy Windows trên Mac", "test như máy thật", hay bất kỳ câu nào ngụ ý x86-64 native. Mọi số liệu hiệu năng đo trong VM ARM phải kèm disclaimer |
| **Nghĩa vụ thuế xuyên biên giới** | Chắc chắn | Phiền phức, phạt chậm nộp | **Merchant of Record (Paddle/Lemon Squeezy)** gánh VAT/sales tax toàn cầu; Stripe trực tiếp tạo nghĩa vụ thuế ở EU/UK mà indie VN không xử nổi. Phía VN xem §9.6 |
| **Trademark tên sản phẩm** | Thấp | Đổi tên tốn kém | Tra USPTO/EUIPO + npm + domain + App Store **ở tuần 1** |

---

## 9. Mô hình kinh doanh

### 9.1. Nguyên tắc định giá (rút từ 10 phản biện)

1. **Không neo giá theo OrbStack $8 / Docker $11.** Đó là infrastructure chạy 8 tiếng/ngày. Sản phẩm này chạy theo nhịp release.
2. **Không dùng subscription tháng cho công cụ tần số thấp.** $12/tháng cho công cụ mở 4 lần/tháng = churn có hệ thống (user huỷ rồi cài lại).
3. **Không đặt paywall vào thứ mà một `justfile` 40 dòng thay được.** Paywall phải đặt vào **ma trận nhiều target** và **tri thức tích luỹ**.
4. **Không bán compute.** Biên gộp phải ~95% vì tất cả chạy trên máy/tài khoản của user.
5. **Không đặt giá trên ~$10/tháng nếu chưa trả lời được "sao tôi không mua mini PC $250?"** — và câu trả lời phải tính cả việc **máy cũ ở VN thường đã kèm license OEM**, tức đối thủ này còn rẻ hơn con số $250.

### 9.2. Chi phí khởi động thật sự

Không bảng nào trong tài liệu này từng liệt kê đủ. Đây là toàn bộ tiền phải chi **trước khi có đồng doanh thu đầu tiên**. Tỷ giá dùng ~26.000 VND/USD (08/2026, ước lượng — tự kiểm tra lại).

| Khoản | Chi phí | Bắt buộc? | Ghi chú |
|---|---|---|---|
| **Apple Developer Program** | **$99/năm ≈ 2,5 triệu VND** | Bắt buộc **khi ship bản cài** | Điều kiện duy nhất để có **Developer ID + notarization**. Mac App Store bất khả thi (cần hypervisor entitlement) → phân phối trực tiếp là con đường duy nhất, mà phân phối trực tiếp không notarize thì Gatekeeper chặn. **Hoãn được:** binary cài qua Homebrew/npm/`curl` không bị gắn quarantine → tuần 1-3 chưa cần. Đến tuần 4 phát hành `.dmg`/`.pkg` thì bắt buộc |
| **Domain `.com`** | $10-15/năm ≈ 260-400k | Bắt buộc | |
| **Landing + doc hosting** | **$0** | — | Cloudflare Pages / GitHub Pages |
| **Email tên miền** | $0-72/năm | Không | Cloudflare Email Routing $0; Google Workspace ~$6/tháng nếu cần gửi |
| **VPS x86-64 Linux (CI/build, batch)** | Hetzner CX22 EU ~**€4,3/tháng ≈ 120-130k VND** | Gần như bắt buộc | Rẻ nhất, nhưng **CX/CAX chỉ có ở châu Âu** — latency VN→Falkenstein ~230-300ms, có thể vọt 350ms+ khi đứt cáp. Tốt cho batch/CI, tệ cho phiên tương tác. Thêm: **Hetzner nổi tiếng khó với khách VN** (đòi xác minh danh tính, hay từ chối thẻ phát hành tại VN) — rào cản này thực tế lớn hơn giá. Độ tin cậy giá: **Thấp** |
| **VPS Singapore (phiên tương tác)** | Vultr ~**$20/tháng**, DigitalOcean ~**$24**, Akamai/Linode ~**$24** (2 vCPU/4GB) | Không, chỉ khi cần latency | VN→Singapore ~25-50ms (TP.HCM 25-40, Hà Nội 40-60). Thẻ VN qua dễ hơn Hetzner. Độ tin cậy giá: **Thấp** |
| **VPS trong nước** | Vietnix ~250-450k, AZDIGI ~200-400k, VNG/FPT/Viettel IDC ~400-800k VND/tháng | Không | Đắt hơn nhưng có **hoá đơn VAT**, hỗ trợ tiếng Việt, latency <10-20ms. Độ tin cậy: **Thấp** |
| **Windows VM theo giờ (x64 thật)** | Azure B2s ~$0.10/h, AWS t3.medium ~$0.08/h → **~$2/tháng ở 20 giờ**, tổng thực tế **$4-12/tháng** kèm disk/snapshot | Không, nhưng ROI cao | License **Windows Server** gộp trong giá giờ. Muốn **Windows 11 client** thì vướng licensing (xem §8.3) |
| **Mini PC N100 / laptop Windows cũ** | Mini PC ~5-6 triệu; **laptop cũ 2-4 triệu**; OptiPlex/EliteDesk micro refurb 1,5-3 triệu | Không, nhưng là fidelity 100% | **Ở VN thường đã kèm license OEM Windows** → không phải cộng thêm tiền license. Kiểm tra **edition trước khi mua**: Hyper-V (dùng cho reset bằng checkpoint) **không có trên Windows 11 Home**, mà phần lớn mini PC giá rẻ ship Home. Điện: ~15-35k VND/tháng |
| **License Windows nếu chạy VM local** | **$0** với Enterprise ARM64 Evaluation (90 ngày, không cần key) | Không | Win 11 Pro **retail/FPP** ~199 USD quốc tế, ~4-5 triệu tại VN nếu cần VM vĩnh viễn (độ tin cậy: **Thấp**). **Không mua OEM/key rẻ** — không hợp pháp cho VM. Visual Studio Professional subscription có kèm Windows cho dev/test nhưng ~1.199 USD/năm (độ tin cậy: **Thấp**) — quá đắt cho dev solo, đúng cho công ty |
| **Chứng chỉ ký code Windows** | ~$100-400/năm tuỳ CA; Azure Trusted Signing ~$10/tháng | Bắt buộc **nếu ship bản Windows** | Sau 2023 khoá phải nằm trong HSM/token → không còn PFX rẻ. Nếu chỉ ship CLI cho dev thì hoãn được |
| **Merchant of Record (Paddle / Lemon Squeezy)** | **~5% + ~$0,50/giao dịch** | Bắt buộc khi bắt đầu bán | Ở giá $99/năm: mất ~$5,45/đơn, tức **~5,5% doanh thu**. Đổi lại họ gánh toàn bộ VAT/sales tax toàn cầu — với indie VN đây là món hời, không phải phí. Độ tin cậy: **Trung bình** |
| **Hạ tầng cho Compatibility Oracle (§9.5)** | $0-5/tháng | Không | Cloudflare Workers + D1/SQLite |
| **Thuế phía VN** | ~7% doanh thu (hộ kinh doanh) hoặc 20% lợi nhuận (công ty) | Chắc chắn | Xem §9.6 |

**Tổng năm đầu, kịch bản tối thiểu (chỉ Lớp 0 + Lớp 1 qua SSH):** Apple Dev 2,5 triệu + domain 0,3 triệu + VPS EU 1,5 triệu ≈ **4,3 triệu VND (~$165)**.
**Kịch bản đầy đủ (thêm máy Windows x86-64 thật + ký code):** ≈ **10-14 triệu VND**.

Con số này nhỏ so với mọi startup, nhưng nó là **tiền mặt chi trước, không hoàn lại**, trong khi §9.7 nói rõ kịch bản khả dĩ nhất là 40-80 khách. Biết trước để không bất ngờ.

### 9.3. Bảng giá đề xuất

| Tier | Giá | Bao gồm | Vì sao |
|---|---|---|---|
| **Scan** | **$0 vĩnh viễn** | `firstrun scan` đầy đủ, mọi rule, mọi framework, không giới hạn | Đây là **kênh phân phối**, không phải freemium. Nó phải tốt đến mức người ta chụp màn hình đem đi khoe |
| **Free** | **$0 vĩnh viễn**, cá nhân + OSS + public repo | `run` trên **1 target tại một thời điểm**, không giới hạn số run, lịch sử 7 ngày | `run` là primitive cần vào Makefile/justfile của user để tạo entrenchment. Khoá nó là tự sát |
| **Pro** | **$99/năm, annual-only** (≈$8,25/tháng nhưng **không bán theo tháng**) | Ma trận nhiều target **song song**, remote target, lịch sử run + so sánh screenshot giữa các lần, `scan` KB cập nhật liên tục, JUnit/CI integration, PR bot | **Annual-only là quyết định có chủ đích:** nó loại bỏ vòng huỷ-rồi-cài-lại của công cụ tần số thấp |
| **Team** | **$299/năm cho 5 seat** (+$50/seat thêm) | Pro + shared config/baseline, quyền dùng thương mại, invoice/PO/VAT | Bán theo **repo/team**, không theo người — job là của dự án, không phải của cá nhân |
| **Giá vùng VN/SEA/Ấn Độ** | **Pro ~890k VND/năm** (giảm ~60%) | như Pro | Qua regional pricing của Paddle. $99 = 2,5 triệu là rào cản thật ở VN |

**Neo giá thật không phải Parallels.** Neo thật là: **$0** của Lima/UTM/VMware Fusion ở một đầu, và **2-6 triệu VND cho một máy x86-64 thật đã kèm OEM Windows** ở đầu kia. Đừng xây luận điểm định giá lên con số của Parallels — xem §9.4, con số đó chưa xác minh được.

**Không làm trong 2 năm đầu:**
- Bán compute / hosted Windows. Kể cả khi đường Windows 11 client trên dedicated hardware là hợp pháp (§8.3), một máy riêng cho mỗi khách phá vỡ biên gộp 95%.
- License vĩnh viễn — sản phẩm cần bảo trì ma trận liên tục, perpetual là tự sát vận hành.
- VC funding.

### 9.4. So sánh với đối thủ ở cùng bảng

| | Giá/năm | Cho bạn cái gì |
|---|---|---|
| **UTM / VMware Fusion / Lima** | **$0** | Một cái máy. Bạn tự làm mọi thứ còn lại |
| **firstrun Pro** | **$99** | Một câu trả lời + bằng chứng + danh sách những gì CHƯA được trả lời |
| **Parallels Desktop Standard** | ~$99,99/năm subscription, **hoặc** bản perpetual (một lần, không kèm nâng cấp phiên bản mới) | Máy đẹp nhất, **được Microsoft authorized**, DX11.1 |
| **Parallels Desktop Pro** | ~$119,99/năm — **subscription-only** | Như trên, thêm 32GB vRAM / nhiều vCPU, tích hợp CLI |
| **Windows VM theo giờ (Azure/AWS)** | **~$24-144/năm** ở 20 giờ/tháng | Windows **Server x64 THẬT**, API spin-up/snapshot/tear-down đầy đủ, license gộp trong giá giờ |
| **Windows 365 Business** | $28-56/user/**tháng** = $336-672/năm | Windows **11 client** hợp lệ, zero ops. Điểm hoà vốn so với VM theo giờ rơi vào ~250-300 giờ/tháng — tức gần như chạy 24/7 mới đáng |
| **Chromatic / Percy** | $1.788-2.388 | Visual regression cho **web** |
| **Sauce Labs Virtual Cloud** | $1.788 | Browser + mobile, 1 parallel test |
| **BrowserStack Desktop-Automate Pro** | $2.988 | Desktop **BROWSER**, không phải desktop app |
| **Mini PC N100 / laptop cũ VN** | **$250 / 2-4 triệu VND một lần** | Máy Windows x86-64 **THẬT**, fidelity 100%, **thường đã kèm OEM Windows**. Không snapshot/matrix/integration |

**Về giá Parallels — phải nói thẳng vì bản trước để "nguồn mâu thuẫn $129.99 vs $219.99":** kiểm chứng 08/2026 **không đọc được giá chính xác** — trang bán hàng chính thức render giá bằng JavaScript, các URL `/products/desktop/pricing/` đều 404. Thứ **xác nhận được** (độ tin cậy Trung bình-Cao) là **cấu trúc edition**: chỉ **Standard** có cả subscription lẫn mua đứt (perpetual); **Pro và Business chỉ bán theo subscription** — nên mọi câu nói "Pro có bản perpetual" là sai. Về con số: mức perpetual lịch sử của Standard luôn quanh **$129,99**; **$219,99 không khớp bất kỳ SKU nào đã biết** (nhiều khả năng là giá gộp nhiều năm). **Độ tin cậy giá: Thấp — không dùng con số này làm cơ sở cho bất kỳ quyết định định giá nào.** Nếu cần chốt, tự mở `parallels.com/products/desktop/buy/` bằng trình duyệt thật.

### 9.5. Đường doanh thu thứ hai (độc lập, đáng cân nhắc riêng)

**Compatibility Oracle cho thị trường VN** — lấy nguyên từ concept Cuaso, tách hẳn khỏi sản phẩm dev tool:

Một CSDL tra cứu công khai miễn phí: *"Phần mềm X có chạy được trên MacBook Apple Silicon không, và bằng cách nào"* — 4 trạng thái + lý do dễ hiểu + giải pháp thay thế. Bắt đúng intent tìm kiếm có ý định mua ("HTKK cho MacBook", "MISA trên Mac", "máy Mac chạy được phần mềm kế toán không").

- **Chi phí hạ tầng:** vài USD/tháng (SQLite/D1 + Cloudflare Workers).
- **Không cần một dòng code hypervisor nào.**
- **Doanh thu = affiliate định tuyến:** Parallels / mini PC & laptop cũ (đại lý) / bản web-cloud của chính phần mềm đó (MISA AMIS) / **ký số từ xa (VNPT SmartCA, Viettel-CA MySign, FPT.eSign, MISA eSign)** / Windows 365. **Kiếm tiền từ 100% người ghé thăm, kể cả 40% mà câu trả lời là "không chạy được"** — đúng nhóm mà một sản phẩm phần mềm phải từ chối bán và mất trắng.
- **Rào chắn nhẹ:** kết quả CHẠY/KHÔNG CHẠY miễn phí (để SEO), chi tiết có giá trị (phiên bản chính xác, workaround, log lỗi) sau email → chống scrape + xây list email.
- **Và quan trọng nhất về mặt chiến lược:** log tìm kiếm trong 3 tháng cho biết chính xác phần mềm nào được hỏi nhiều nhất, bao nhiêu người, họ ở đâu. Đó là cách kiểm chứng nhu cầu thật với chi phí gần bằng không.

**Bốn giả định phải verify TRƯỚC khi bỏ một tháng vào đây:**

| Giả định | Nếu sai thì sao | Cách verify rẻ nhất |
|---|---|---|
| Người VN vẫn cần chạy phần mềm Windows trên Mac | Cả đường doanh thu này teo dần | Ký số từ xa (SmartCA/MySign/FPT.eSign/MISA eSign) + kê khai web (thuedientu, eTax Mobile, hoá đơn điện tử bắt buộc) đang **thay thế đúng lý do "cần Windows"**. Đo bằng search volume theo tháng của "HTKK cho MacBook" trong 24 tháng gần nhất — nếu đường xu hướng đi xuống thì bỏ |
| Traffic tìm kiếm đủ lớn | Không có ai để định tuyến | Keyword research trước, viết code sau. 1 ngày |
| Affiliate thật sự trả tiền | Doanh thu = 0 dù traffic cao | Hỏi thẳng 3 chương trình (Parallels, một CA ký số từ xa, một đại lý mini PC) về commission trước khi viết dòng nào |
| Câu trả lời cho 2 truy vấn nhiều traffic nhất là "không chạy được" | Nếu thật ra **chạy được**, nội dung và affiliate phải đổi hoàn toàn | Xem thí nghiệm dưới đây, đặc biệt dòng iTaxViewer |

**Thí nghiệm rẻ nhất có tỷ lệ thay đổi quyết định cao nhất:** mượn một máy M-series + UTM + Windows 11 ARM 25H2, cài thử đúng 10 phần mềm quyết định — MISA SME, Fast Accounting, phần mềm BHXH, dự toán G8, **iTaxViewer**, HTKK 5.x, 1-2 app ngân hàng doanh nghiệp, 1 app chữ ký số USB token, 1 app quản lý bán hàng.

Hai kết quả có thể chấm dứt sớm cả hướng đi:

1. **Nếu MISA/Fast không cài được vì SQL Server không có bản ARM64 — khả năng này cao — thì toàn bộ wedge kế toán chết ngay tại đó, và bạn tiết kiệm được 12 tháng.**
2. **iTaxViewer phải test riêng, không gộp chung.** Nó chạy trên JRE. **Nếu nó chạy trực tiếp được trên JVM của macOS/Linux thì câu trả lời của Oracle cho một trong hai truy vấn có traffic cao nhất đổi từ "cần Windows" sang "không cần Windows, tải JRE về là xong"** — nghĩa là bài viết đó không định tuyến sang Parallels/mini PC được nữa, mà thành một trang hướng dẫn miễn phí. Tốt cho uy tín, nhưng phải biết trước để tính lại doanh thu kỳ vọng, không phải phát hiện sau khi đã xây xong.

### 9.6. Thuế và nhận tiền — phần dành riêng cho dev ở VN

Đây là ma sát thật, và bản trước chỉ ghi đúng ba chữ "hoá đơn điện tử". Bốn điểm cụ thể (độ tin cậy: **Trung bình** — chính sách thuế VN thay đổi nhanh, hỏi kế toán trước khi làm theo):

1. **Nhận tiền về:** Paddle và Lemon Squeezy đều trả về VN qua **Wise** hoặc **PayPal** (Payoneer cũng được với một số nhà). Wise phí thấp hơn PayPal đáng kể và cho giữ số dư USD. Ngân hàng VN có thể hỏi chứng từ nguồn tiền với khoản lớn — giữ lại invoice/payout report của MoR.
2. **Cá nhân hoặc hộ kinh doanh:** doanh thu dịch vụ phần mềm chịu **thuế khoán theo tỷ lệ trên doanh thu — khoảng 5% GTGT + 2% TNCN, tổng ~7%**, nộp trên **doanh thu** chứ không phải lợi nhuận. Có ngưỡng doanh thu năm được miễn (ngưỡng này vừa được nâng lên trong chu kỳ 2026 — **phải tự kiểm tra con số hiện hành**, đừng tin số cũ).
3. **Lập công ty (TNHH):** chịu **20% TNDN trên lợi nhuận**, khấu trừ được chi phí (VPS, Apple Dev, thiết bị). Hoạt động **sản xuất sản phẩm phần mềm** thuộc diện ưu đãi thuế đáng kể ở VN (miễn/giảm nhiều năm, thuế suất ưu đãi) — nhưng phải đăng ký đúng ngành và đủ hồ sơ chứng minh. Chi phí kế toán thuê ngoài ~1-3 triệu/tháng, cộng nghĩa vụ BHXH nếu có nhân sự.
4. **Ngưỡng để chuyển:** ở mức doanh thu của kịch bản "Xấu" ($4.000-8.000/năm ≈ 100-210 triệu VND) thì **hộ kinh doanh gần như luôn đúng** — 7% trên doanh thu vẫn rẻ hơn chi phí vận hành một công ty. Chỉ tính đến công ty khi (a) doanh thu vượt ~500 triệu/năm, (b) cần xuất hoá đơn VAT cho khách doanh nghiệp trong nước, hoặc (c) cần ký hợp đồng có tư cách pháp nhân. Đừng lập công ty vì nó "chuyên nghiệp hơn".

### 9.7. Dự phóng tỉnh táo (không PR)

| Kịch bản | Khách Pro năm đầu | Doanh thu gộp | Sau ~5,5% phí MoR | Sau thuế khoán ~7% |
|---|---|---|---|---|
| **Xấu (khả dĩ nhất)** | 40-80 | $4.000-8.000 | $3.780-7.560 | ~$3.520-7.030 |
| Trung bình | 200-400 | $20.000-40.000 | $18.900-37.800 | ~$17.580-35.150 |
| Tốt | 800-1.200 | $80.000-120.000 | $75.600-113.400 | ~$70.300-105.470 |
| Trần thực tế (3 năm) | 1.500-3.000 | $150.000-300.000 | $141.750-283.500 | ~$131.830-263.660 |

Trừ tiếp chi phí cố định ~$165-540/năm ở §9.2. Ở kịch bản Xấu, thu nhập ròng năm đầu rơi vào khoảng **$3.400-6.900** — đủ để tiếp tục, không đủ để nghỉ việc.

**TAM trả tiền ước lượng: vài chục nghìn dev toàn cầu.** Đây là sản phẩm **bootstrapped, side-income hoặc nuôi 1-2 người**. **Nếu mục tiêu là $10k MRR trong năm đầu thì đây là ý tưởng sai.** Chi phí biên phải gần $0 ngay từ đầu để tồn tại được ở quy mô đó — đó chính là lý do kiến trúc "không sở hữu compute" không phải là sự khiêm tốn mà là điều kiện sống.

---

[← 4. Kiến trúc, MVP và roadmap](04-kien-truc-mvp-roadmap.md) · [Mục lục](README.md) · [6. Làm gì ngay tuần này →](06-lam-gi-tuan-nay.md)
