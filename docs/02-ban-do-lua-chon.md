# 2. Bản đồ lựa chọn

> Mọi đường đi để có Windows/Linux từ một máy Mac: sản phẩm thương mại, phần cứng thật, cloud theo giờ, cloud miễn phí, và đường license.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

Phần này không chỉ liệt kê đối thủ. Nó liệt kê **mọi đường đi** một dev Mac-only có thể chọn để có được Windows/Linux — sản phẩm thương mại, phần cứng thật, cloud theo giờ, cloud miễn phí, và đường license. Đối thủ chỉ là một tập con.

Quy ước độ tin cậy dùng trong toàn phần: **Cao** = đọc trực tiếp từ tài liệu/source chính chủ; **Trung bình** = cơ chế kỹ thuật chắc, con số cụ thể chưa verify; **Thấp** = ước lượng từ kiến thức nền, **bạn phải tự kiểm tra trước khi ra quyết định chi tiền**. Toàn bộ con số giá trong phần này chưa được cross-check online trong phiên làm việc 17/08/2026 (hết ngân sách tra cứu), trừ những chỗ ghi rõ nguồn.

## 2.1. Bảng so sánh chính

| Sản phẩm / đường đi | Giá 2026 | Mạnh nhất | Yếu nhất | Tại sao KHÔNG đủ cho use case của bạn |
|---|---|---|---|---|
| **Parallels Desktop 26** | Std **$99.99/năm** hoặc bản **perpetual** (chỉ Std mới có perpetual); Pro **$119.99/năm**, Business **$149.99/năm** — **cả Pro lẫn Business chỉ bán subscription**. Con số perpetual chính xác **không xác minh được** (trang giá render bằng JS): lịch sử luôn quanh **$129.99**, bản phản biện ghi **$219.99** — *độ tin cậy Thấp, tự kiểm tra* | Giải pháp **duy nhất được Microsoft authorized**. Coherence. DX11.1. One-click cài Win11 ARM. 7 triệu user (số marketing của chính Parallels) | Subscription tăng đều, ép upgrade mỗi bản macOS. Std khoá cứng 8GB vRAM/4 vCPU. Không DX12. Không nested virt cho Windows guest | Bán **một cái máy**, không bán **một kết quả test**. Không build, không chạy test, không chụp screenshot, không biết gì về dự án của bạn — và **im lặng** về việc bạn đang test trên ARM chứ không phải x64 |
| **VMware Fusion Pro 26H1** | **$0**, kể cả **thương mại** | Miễn phí hoàn toàn, không cần key. ARM64 guest gần native | Broadcom bỏ bê: không support chính thức, lỗi display scaling trên Apple Silicon, download qua portal Broadcom rất khó chịu. Không Coherence, **không shared folder** trên Apple Silicon | **Đây là "nuclear option" đã nổ rồi.** Nghĩa là bạn KHÔNG THỂ bán "VM engine". Nhưng nó vẫn chỉ cho bạn một cửa sổ Windows rồi để bạn tự nheo mắt |
| **UTM 4.7.5** | **$0** (App Store $9.99 = donation) | Open-source Apache-2.0, 35k stars, chạy Win11 ARM near-native với HVF, hỗ trợ nhiều kiến trúc | Bắt tự tải ISO, tự cài virtio driver, tự cài SPICE tools, tự gõ lệnh mount, không auto-resolution, không 3D cho Windows guest. Emulation x86_64 chậm 5-20x | Là **bộ đồ nghề**, không phải **dây chuyền**. Hoàn toàn không có khái niệm "chạy artifact của tôi rồi trả kết quả về" |
| **OrbStack** | Free cá nhân, **$8/user/tháng** thương mại | Nhanh nhất mảng Linux dev trên Mac: boot ~2s, tiết kiệm RAM/pin. Là hình mẫu về "mặc định đúng, CLI-first" | Chỉ container + Linux VM **headless**. Không GUI, không desktop environment, không Windows | Không test được app GTK/Qt/Electron **có UI**. Khác vũ trụ, nhưng là **neo giá** và là **công thức UX** đáng học |
| **Apple `container` / Containerization framework** | **$0**, Apache-2.0. Mới nhất **v1.2.2 (08/08/2026)** — đã qua 1.0 nhưng repo vẫn ghi "minor version có thể có breaking changes" *(độ tin cậy Cao)* | Runtime container Linux **first-party của Apple**, **mỗi container một micro-VM riêng** trên Virtualization.framework. **Có `--arch amd64` chạy binary x86-64 qua Rosetta** → use case "chạy/test binary Linux x86-64 trên MacBook Apple Silicon" là khả thi | **Yêu cầu macOS 26** để dùng đủ tính năng (macOS 15 chạy được nhưng container không nói chuyện được với nhau, không đa network, có bug subnet làm mất mạng). Chưa có GUI/dashboard, chưa có Compose first-party, hệ sinh thái mỏng hơn Docker Desktop/OrbStack. VM-per-container tốn RAM hơn shared-VM. Boot time Apple nói "comparable to shared VM" nhưng **không công bố số** *(Thấp)* | Chỉ Linux, headless, không giải quyết Windows. Rosetta chỉ dịch user-space x86-64 → syscall/kernel module/AVX-512/binary 32-bit vẫn fail. Nhưng đây là **backend `local-vz` gần như miễn phí công sức**: dùng nó thay vì tự viết Swift + VZ |
| **Lima / Colima / Multipass / Podman Desktop / Rancher** | **$0** toàn bộ | Lima v2.2 (07/2026) đã có **Windows guest + TPM 2.0 qua swtpm + unattend.xml + auto virtio driver**. Dự án CNCF, có LFX Mentorship 2026 làm riêng Windows guest | Windows template phải `plain: true` → **mất file mount và port forwarding**. Headless, YAML, terminal | "Linux trên Mac cho dev" đã bị **commoditize hoàn toàn ở $0**. **Đây mới là đối thủ nguy hiểm nhất, không phải Parallels** — một dự án CNCF được tài trợ nhân lực đang bò dần vào chính chỗ này, miễn phí vĩnh viễn |
| **CrossOver 26** | **$74/năm**, Life $494 | Chạy app Windows **không cần Windows**. Wine 11 + D3DMetal 3.0. Bản native Apple Silicon đang preview (07/2026) | Bản stable vẫn là Intel chạy qua **Rosetta 2 sắp chết**. Bản ARM64 preview chưa có D3DMetal, DX12 "coming soon" | **KHÔNG phải công cụ test.** Nó không phải Windows thật — registry, DPI scaling, font rendering, native dialog, MSI installer, Windows API edge case đều khác. Test trên CrossOver rồi kết luận "chạy tốt trên Windows" là **nguy hiểm và sai** |
| **GitHub Actions** | Linux x64 **$0.006/phút**, Linux arm64 **$0.005**, Windows x64 **$0.010**, Windows arm64 **$0.010**, macOS **$0.062**. Free tier private: Free 2.000 phút/tháng, Pro/Team 3.000, Enterprise Cloud 50.000. **Public repo: standard runner miễn phí** *(độ tin cậy Cao — docs.github.com)* | Rẻ đến mức gần như miễn phí. `windows-latest` là Windows Server x64 **THẬT**. `windows-11-arm` **đã mở cho cả public lẫn private repo** (không còn public-only); biến thể `windows-11-vs2026-arm` còn ở private preview | **Headless**: build được, **không NHÌN được**, không attach debugger, không click thử, không thử installer với Defender bật (runner **chủ động tắt Defender** — `Configure-WindowsDefender.ps1` set `DisableRealtimeMonitoring`, exclusion `C:\` `D:\`, và `ForceDefenderPassiveMode=1` nếu registry path ATP tồn tại). `windows-latest` nay là **Windows Server 2025**, không phải Windows 11 client | Là **bổ sung**, không phải thay thế. Vòng lặp 20-45 phút. Và **suite test 10 phút chỉ tốn $0.10** → "tiết kiệm tiền CI" KHÔNG phải luận điểm bán hàng. Lưu ý: mô hình "nhân 2x Windows / 10x macOS" đã lỗi thời — nay tính per-minute, thực tế Windows ≈ **1,67x** Linux, macOS ≈ **10,3x** |
| **BrowserStack / Sauce Labs / LambdaTest** | BS: App Live ~$29/th, Desktop-Automate Pro $249/th, Team $449/th. Sauce Virtual Cloud $149/th (annual). LambdaTest $15-39/th | Hạ tầng chín, parallel lớn | **"Desktop" ở đây nghĩa là desktop BROWSER, không phải desktop application** | **Không một lab nào cho bạn upload file `.exe`/`.msi`/`.AppImage`/`.deb` rồi xem nó chạy thật trên Windows 11/Ubuntu.** Đây là khoảng trống rõ ràng nhất |
| **VM Windows cloud tính theo GIỜ (Azure / AWS / GCP)** | Azure **B2s Windows ~$0.09-0.10/giờ**, B2ms ~$0.14-0.15/giờ; AWS **t3.medium Windows ~$0.08/giờ**; GCP e2-medium + license ~$0.11/giờ *(GCP: Thấp)*. **20 giờ/tháng ≈ $2 compute**, cộng disk/snapshot → **tổng thực tế $4-12/tháng** | Windows x64 **THẬT**. **License Windows gộp trong giá giờ**. API spin-up / snapshot / tear-down **đầy đủ** (`az vm create/deallocate/delete`, `az snapshot create`, `aws ec2 run-instances / create-image / terminate-instances`), image tuỳ biến (AMI / Managed Image / Gallery) tái dùng mỗi build. **Hợp pháp, không dính QMTH** | Là **Windows Server 2022/2025**, **không phải Windows 11 client** — muốn đúng Win11 client (shell integration, MSIX, packaged app, driver, DPI scaling của Win11) thì vẫn vướng licensing, phải qua Windows 365 / AVD / dedicated host BYOL. Disk OS persistent tính tiền cả khi VM tắt (~$5-10/tháng cho 128GB, hoặc ~$1-3 nếu snapshot rồi xoá). Latency stream cho phiên tương tác. Không GPU (trừ SKU đắt) | **Đây KHÔNG phải "không đủ" — đây là đường x64 thật RẺ NHẤT cho nhu cầu tương tác thỉnh thoảng**, và nó đang bị gộp nhầm vào rổ per-user-per-month bên dưới. Thiếu duy nhất: Windows 11 client thật, GPU thật, và bạn phải tự quản image/automation |
| **Windows 365 / Shadow PC** | W365 Business **Basic $28 / Standard $36 / Premium $56 per user/tháng**. Shadow $37.99/th | W365 cho bạn **Windows 11 client hợp pháp 100%**, zero ops. Shadow là Windows x64 **thật có GPU** | Mô hình **per-user-per-month**: trả cả tháng dù chỉ cần 3 giờ trước release. Windows 11 ARM trên Azure Cobalt **vẫn chỉ Insider Preview** | Sai mô hình chi phí cho vòng lặp dev: ở mức **~20 giờ/tháng, VM theo giờ rẻ hơn 3-7 lần**; **điểm hoà vốn rơi vào ~250-300 giờ/tháng** — tức gần như chạy 24/7 mới khiến Cloud PC đáng giá. Latency stream bóp méo cảm nhận về UI responsiveness — đúng thứ cần test |
| **Mini PC Intel N100 16GB/512GB** | **$240-270 một lần** (VN: ~5-6 triệu) | Máy Windows **x86-64 THẬT**: driver kernel, USB token, anti-cheat, GPU thật, DX12, WSL2, Windows Sandbox, mọi bug đại diện 100% | Không snapshot/rollback tức thì, không 8 cấu hình song song, không tích hợp vào vòng lặp code trên Mac, tốn chỗ. **Phần lớn ship Windows 11 Home → không có Hyper-V, không có Windows Sandbox** (phải kiểm tra edition trước khi mua) | **Đây là đối thủ mạnh nhất, không phải Parallels.** The Register khi review x86 emulation của Parallels kết luận thẳng: ai thật sự cần x86 thì nên dùng cloud remote desktop, máy Intel cũ, hoặc mini PC — không phải emulation |
| **Laptop Windows cũ / OptiPlex – EliteDesk micro refurb** | Laptop cũ **2-4 triệu VND**; Dell OptiPlex / HP EliteDesk micro refurb **~1.5-3 triệu** *(giá VN, Trung bình)* | Cùng fidelity x86-64 thật như mini PC, nhưng: laptop có sẵn **màn hình + bàn phím + pin = UPS tích hợp** (rất đáng giá ở VN, nơi mất điện/sụt áp là chuyện thường); refurb doanh nghiệp (i5 gen 8-10) **mạnh hơn N100 rõ rệt** ở cùng tầm tiền; **thường kèm license OEM** | Pin cũ chai, ồn/nóng hơn, khó nâng cấp. License OEM **gắn vĩnh viễn với máy vật lý, không chuyển sang VM được**. Vẫn có thể là Home edition → mất Hyper-V/Sandbox | Cùng nhược điểm mini PC: không snapshot tức thì trừ khi có Pro, không song song, không nằm trong vòng lặp code trên Mac. Nhưng **rẻ hơn mini PC $250 và tiện hơn** — nên đây là lựa chọn mặc định đúng ở VN, không phải mini PC |
| **Máy Windows ARM thật (Snapdragon X, Copilot+ PC)** | **~$600-800** mới, refurb rẻ hơn | **Cách DUY NHẤT nhìn thấy nền tảng Windows ARM64 trên phần cứng thật**: Prism thật, driver ARM64 thật, DPI/GPU/thermal thật — không qua lớp hypervisor nào | Đắt nhất trong nhóm phần cứng. **Không giải quyết fidelity x64** (chính nó là ARM). Ở VN nguồn hàng hạn chế | Chỉ đáng khi bạn thực sự ship bản ARM64, hoặc user của bạn có Copilot+ PC. Với đa số app, một máy x64 thật đáng tiền hơn nhiều |
| **Oracle Cloud Always Free + các nguồn credit** | **$0 vĩnh viễn**: 2 instance AMD x86 + 4 core ARM Ampere / 24GB RAM. Cộng GitHub Student Pack, Microsoft for Startups, Azure credits | Linux **x86-64 thật ở $0 thật sự** — thay thẳng dòng "VPS €4/tháng" trong §10. ARM 24GB là món hời hiếm | Oracle nổi tiếng **reclaim instance Always Free** khi region thiếu capacity; đăng ký/verify thẻ từ VN hay bị từ chối; credits có hạn sử dụng và cần điều kiện (sinh viên / startup) | Chỉ Linux, không có Windows. Nhưng cho nhánh Linux của bài toán thì đây là đáy chi phí tuyệt đối |
| **VPS Linux x86-64 trả tiền** | Hetzner **CX22 ~€3.79/tháng + IPv4 ~€0.50** (~120-130k VND) nhưng **CX/CAX là EU-only**; Singapore chỉ có **CPX/CCX**, CPX21 ~€8-10/tháng. Vultr ~**$20**, DigitalOcean ~**$24**, Akamai/Linode ~**$24** cho 2 vCPU/4GB Singapore. Trong nước: Vietnix ~250-450k, AZDIGI ~200-400k, VNG/FPT/Viettel IDC ~400-800k VND *(toàn bộ: **độ tin cậy Thấp**)* | Linux x86-64 **thật** — fidelity cao hơn mọi VM ARM local. Chạy được `systemd-nspawn --ephemeral` cho cả ma trận distro trên một máy | **Hetzner khó với khách VN**: hay yêu cầu ảnh hộ chiếu/CCCD, từ chối thẻ phát hành tại VN, ưu tiên PayPal/SEPA — rào cản này lớn hơn cả giá. Latency **VN→Singapore ~25-50ms** nhưng **VN→Falkenstein ~230-300ms** (vọt 350ms+ khi đứt cáp) | Không có Windows, không có GPU, không có desktop session thật trừ khi bạn tự dựng. Tổ hợp hợp lý: **Hetzner EU rẻ nhất cho CI/build batch + một VPS Singapore cho phiên tương tác** |

## 2.2. Sửa lỗi nặng: "không có API spin up/tear down theo build" là SAI

Đây là lỗi nghiêm trọng nhất trong bản đồ cũ, vì nó **loại nhầm một đường đi rẻ và hợp pháp** ra khỏi danh sách cân nhắc.

| Khẳng định cũ | Sự thật |
|---|---|
| "Không có API spin up/tear down theo build" | Sai. Azure: `az vm create` / `az vm deallocate` / `az vm delete` / `az snapshot create`. AWS: `run-instances` / `create-image` / `create-snapshot` / `terminate-instances`. Cả hai đều có custom image (AMI / Managed Image / Compute Gallery) để tái dùng mỗi build |
| "Mọi thứ là per-user-per-month" | Sai. Windows 365 và Shadow là per-user-per-month; **Azure/AWS/GCP VM là per-hour, license Windows gộp trong giá** |
| "Dính QMTH" | Sai với **Windows Server**. QMTH chỉ là vấn đề khi bạn muốn **Windows 11 client** trên hardware multi-tenant |

Phép tính so sánh ở nhịp dùng thật của một dev solo:

| Nhu cầu | Windows 365 Business Standard | Azure B2s Windows theo giờ |
|---|---|---|
| 20 giờ/tháng | $36 | ~$2 compute + $2-10 disk/snapshot = **$4-12** |
| 100 giờ/tháng | $36 | ~$10 + storage = **$12-20** |
| 250-300 giờ/tháng | $36 | **~$36 — điểm hoà vốn** |
| 24/7 (~720 giờ) | $36 | ~$72+ → **W365 thắng** |

Đổi lại cho phần rẻ: bạn nhận **Windows Server**, không phải Windows 11 client, và bạn tự chịu trách nhiệm image/snapshot/automation. Nếu app của bạn chỉ cần kiểm chứng Win32/WinUI thì Windows Server 2022/2025 là đủ; nếu cần **đúng Windows 11 client** (MSIX, packaged app, shell integration, DPI scaling của Win11, driver) thì ràng buộc licensing là có thật và bạn phải quay lại W365/AVD/dedicated host BYOL.

*Độ tin cậy: Trung bình-Cao cho Azure/AWS và Windows 365 (giá lấy từ vantage.sh và trang giá Microsoft chính thức); Thấp cho GCP và cho phần license add-on tách riêng.*

## 2.3. Đường license Windows hợp pháp — bản đồ đầy đủ

Bản đồ cũ chỉ có hai ô (BYO retail, Evaluation 90 ngày). Thực tế có bốn.

| Đường | Chi phí | Được gì | Ràng buộc | Độ tin cậy |
|---|---|---|---|---|
| **Windows 11 Enterprise Evaluation ARM64** | **$0** | ISO **x64 và Arm64**, 25H2, đánh giá **90 ngày**, trang ghi rõ *"A product key is not required for this software"* (chỉ cần sign in tài khoản Microsoft). Chạy native trên Apple Silicon | Chỉ ISO, không có VHDX chính thức. Hết 90 ngày phải dựng lại VM — snapshot/rebuild được, nhưng không gia hạn vô hạn một cách hợp pháp | **Cao** |
| **Windows Insider Preview ARM64 (VHDX / ISO)** | **$0** | Tải miễn phí với tài khoản đã đăng ký Windows Insider. Có bản **VHDX dùng thẳng cho VM** trên Parallels/UTM/VMware Fusion — không phải cài từ ISO. **Không bị giới hạn 90 ngày như bản Evaluation** | Build Insider **có ngày hết hạn riêng** (tự shutdown định kỳ nếu không cập nhật lên build mới) → bạn đổi "dựng lại VM mỗi 90 ngày" lấy "cập nhật build định kỳ". License **pre-release, chỉ để đánh giá/test, không dùng production**. Đây là bản chưa ổn định → không phải môi trường đại diện cho user cuối | **Trung bình** |
| **Visual Studio Professional / Enterprise subscription** (bản standard, không phải cloud/monthly) | Con số ngành thường nhắc: **~$1.199/năm Professional** (renewal ~$799), **~$5.999/năm Enterprise** (renewal ~$2.569) — **giá KHÔNG xác minh được**, trang giá Microsoft timeout/404 | Đây là **câu trả lời chuẩn ngành** cho "dev muốn VM Windows hợp pháp lâu dài". Trang chính thức liệt kê benefit *"Software Access — **Windows**, SQL Server, and Microsoft 365 **for dev/test**"* | Chỉ dùng cho **phát triển và kiểm thử**, tuyệt đối không cho production/desktop làm việc hằng ngày. Gắn với cá nhân người đăng ký (per-user). **Với một dev solo chỉ cần VM để test, chi phí này quá đắt so với nhu cầu** — đây là lựa chọn đúng cho công ty/team | **Trung bình** (có Windows dev/test) / **Thấp** (giá) |
| **Windows 11 Pro retail (FPP)** | ~$199 quốc tế, ~4-5 triệu VND tại VN *(chưa kiểm chứng)* | License **được phép chuyển (transfer)** sang máy/VM khác, kích hoạt trong VM hợp lệ. Đây là đường duy nhất có một VM Windows **vĩnh viễn hợp pháp** cho cá nhân | **OEM thì KHÔNG**: gắn vĩnh viễn với máy vật lý đầu tiên, không chuyển sang máy khác hay sang VM. Mọi key OEM/"key giá rẻ" bán tràn lan **không hợp pháp cho VM** | **Trung bình-Cao** (quyền transfer) / **Thấp** (giá) |

**Khuyến nghị cho hoàn cảnh cụ thể của bạn:** Enterprise Evaluation ARM64 90 ngày làm mặc định (miễn phí, hợp pháp, ARM64 native), Insider ARM64 VHDX làm phương án hai khi cần VM sống lâu hơn 90 ngày. Chỉ mua VS Professional subscription nếu có ngân sách công ty. Nếu cần một VM vĩnh viễn ổn định thì mua **retail**, không mua OEM.

## 2.4. Windows Sandbox — clean room bị bỏ sót

Khi bạn **đã có** một máy Windows x64 thật (mini PC, laptop cũ, VM cloud theo giờ), Windows Sandbox là công cụ smoke test installer đơn giản hơn hẳn vòng Hyper-V checkpoint:

| | Windows Sandbox | Hyper-V checkpoint revert |
|---|---|---|
| Thiết lập | Bật một optional feature, viết file `.wsb` vài dòng | Tạo VM, cài Windows, tạo checkpoint, script revert |
| Reset | **Tự động, sạch tuyệt đối khi đóng cửa sổ** | `Restore-VMCheckpoint` + chờ boot |
| Giữ state giữa các lần | Không (đó là tính năng, không phải bug) | Có |
| Điều kiện | **Windows Pro/Enterprise**. Không có trên **Home** | Cũng cần Pro/Enterprise |

Cả hai đều **chết trên Windows 11 Home** — mà phần lớn mini PC N100 $250 ship Home. Kiểm tra edition trước khi mua là một dòng trong checklist mua hàng, và nó quyết định cả backend `winmetal`.

## 2.5. Ma trận: nhu cầu nào → đường nào

| Bạn cần gì | Đường rẻ nhất | Đường đúng nhất |
|---|---|---|
| Chạy binary Linux x86-64 để kiểm tra packaging/dependency | Apple `container --arch amd64` hoặc Docker `--platform linux/amd64` trên chính Mac ($0) | VPS/Oracle Always Free x86-64 thật (kernel x86-64 thật, không nói dối `uname -m`) |
| Ma trận nhiều distro Linux | `systemd-nspawn --ephemeral` trên một VPS $0-5 | Như trên |
| Build + test tự động mỗi push | GitHub Actions (public repo: $0) | GitHub Actions |
| **Nhìn tận mắt** Windows x64, thỉnh thoảng, tương tác | **VM Azure/AWS theo giờ: $4-12/tháng** | Laptop Windows cũ 2-4 triệu / OptiPlex refurb |
| Test driver kernel-mode, USB token, DX12, anti-cheat, app 32-bit | Không có đường rẻ | **Bắt buộc phần cứng x86-64 thật** |
| Smoke test installer trên máy sạch | Windows Sandbox (nếu Pro) | Windows Sandbox |
| Nhìn thấy nền tảng Windows ARM64 thật | `windows-11-arm` runner trên GitHub Actions | Máy Snapdragon X thật ~$600-800 |
| Windows 11 **client** hợp pháp chạy liên tục | Windows 11 Pro retail trong VM local | Windows 365 (nếu chấp nhận $28-56/tháng) |

## 2.6. Ba kết luận từ bản đồ này

**(a) Không thể cạnh tranh ở tầng hypervisor.** VMware Fusion $0 (kể cả thương mại) + UTM $0 open-source. Bất kỳ sản phẩm nào bán "hypervisor tốt hơn" sẽ thất bại.

**(b) Không thể cạnh tranh ở tầng "chạy được Windows/Linux".** Đó là commodity. Giá trị còn lại chỉ nằm ở **UX, automation và tri thức**.

**(c) Đối thủ nguy hiểm nhất không phải Parallels.** Xếp theo mức độ nguy hiểm thật:
1. **Một file `justfile` 40 dòng** mà chính developer tự viết trong một buổi chiều.
2. **Lima** (CNCF, có nhân lực tài trợ, Apache-2.0, đang bò vào Windows guest).
3. **Mini PC $250**.
4. GitHub Actions ở $0.010/phút.
5. VMware Fusion / UTM ở $0.

Parallels đứng thứ 6. Họ bán cho người ngồi vào máy, bạn bán cho máy chạy không có người — DNA khác nhau.

---

[← 1. Sự thật nền tảng](01-su-that-nen-tang.md) · [Mục lục](README.md) · [3. Năm hướng sản phẩm và khuyến nghị →](03-nam-huong-va-khuyen-nghi.md)
