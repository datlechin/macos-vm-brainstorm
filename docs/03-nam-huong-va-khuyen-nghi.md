# 3. Năm hướng sản phẩm và khuyến nghị

> Năm concept, mỗi cái bị hai giám khảo độc lập tấn công. Không cái nào qua 4.2/10 — và đây là thứ nên xây thay thế.

*[Mục lục](README.md) · [Ràng buộc & độ tin cậy](README.md#ve-do-tin-cay)*

---

Ghi chú trước khi đọc: một số phản bác trong panel gốc dựa trên hai khẳng định đã bị kiểm chứng lại và **sai**. Chúng được sửa tại chỗ dưới đây, kèm đánh giá lại mức độ nghiêm trọng. Điểm số giữ nguyên như phiếu gốc để không viết lại lịch sử; chỗ nào việc sửa đáng lẽ làm điểm nhích lên, tôi ghi thẳng ra.

Hai sửa chữa nền, áp dụng cho nhiều concept:

| Khẳng định trong panel gốc | Sự thật sau kiểm chứng | Hệ quả |
|---|---|---|
| "GNOME 49 xoá hoàn toàn code X11 → không chụp được màn hình trong Linux guest" | GNOME 49 (9/2025) chỉ bỏ mục "GNOME on Xorg" mặc định trong GDM, code X11 session vẫn build được (và khả năng khởi chạy đã được khôi phục sau phản hồi cộng đồng). Việc xoá code lên lịch cho GNOME 50 (chu kỳ 3/2026); Ubuntu 26.04 LTS đi hướng Wayland-only. **XWayland vẫn tồn tại**: `xdotool`, `import`, `xwd`, `ffmpeg -f x11grab` vẫn chạy với app X11 — chỉ không "nhìn" được cửa sổ Wayland native và không chụp được toàn màn hình compositor Wayland. Độ tin cậy: Cao cho cơ chế, Trung bình cho mốc phiên bản. | Phản bác "screenshot trong Linux guest vỡ" **hẹp lại rất nhiều**: nó chỉ đúng khi bạn khăng khăng chạy bên trong một session GNOME/Wayland thật. Trong CI/VM headless, `Xvfb` + `import`/`x11grab`, hoặc `sway --headless`/`weston --backend=headless` + `grim`, giải quyết sạch, không dialog, không phụ thuộc GNOME. Vẫn là vấn đề THẬT cho lane "desktop thật". |
| "Kiến trúc daemon không chạy được vì `VZVirtualMachine` không `NSSecureCoding`" | **Sai.** Tart, driver `vz` của Lima, VirtualBuddy và chính `container` của Apple đều chạy VZ VM trong process nền dài hạn. Thứ không làm được là **truyền object `VZVirtualMachine` qua XPC**; hoàn toàn có thể để daemon **sở hữu** VM và expose API IPC riêng. | Bỏ khỏi danh sách phản bác. Nếu giữ, tài liệu tự bác bỏ `firstrun-vzd` ở §5.1. |

---

## Hạng 1 — Otherside (4.2/10)

> **"Linux desktop VM zero-config trên VZ + `side run <artifact>` trả về screenshot/log."**

**Mô tả.** Bỏ hoàn toàn Windows ở v1. Chỉ Linux guest trên Virtualization.framework — 100% khả thi, không vướng license Microsoft. Ship image dựng sẵn (Ubuntu/Fedora + desktop + guest agent + auto-login), boot ~15s, virtiofs tự mount, clipboard qua SPICE, reset về máy sạch trong ~3 giây bằng APFS `clonefile(2)`. Tính năng thu tiền là `side run ./dist/app.deb` → cài, chạy, chờ cửa sổ, chụp screenshot, gom log, trả về terminal trên Mac.

**Cho ai.** Indie/solo/team 1-5 người ship Electron/Tauri có bản `.deb`/`.rpm`/`.AppImage`.

**Điểm TB: 4.2** (feasibility 3-6, differentiation 5, market_demand 3, moat 2-3, speed_to_value 4-8)

**Phản bác mạnh nhất (đã hiệu chỉnh sau kiểm chứng).**

*Phần đã bị bác bỏ, không còn tính:* khẳng định "GNOME 49 xoá code X11, Ubuntu 26.04 = GNOME 50 Wayland-only, nên cột Linux vô nghĩa" là sai về trình tự sự kiện, và khẳng định "daemon không chạy được vì `NSSecureCoding`" là sai hoàn toàn. Nếu chỉ còn hai đòn này, Otherside sẽ ở hạng cao hơn.

*Phần còn sống, và vẫn đủ nặng:* concept chọn sai **hình dạng môi trường**, không phải sai bài toán. Nó hứa một "Linux desktop VM" với GNOME/KDE thật, rồi đặt tính năng thu tiền duy nhất lên đúng thứ mà desktop session hiện đại đã khoá: `org.gnome.Shell.Screenshot` bị GNOME Shell chặn cho caller ngoài Shell từ bản 41; portal `org.freedesktop.portal.Screenshot` **bắt buộc hiện dialog xin phép** trên GNOME — phá vỡ hoàn toàn một lệnh CLI chạy không người trực (giảm nhẹ được bằng `restore_token` để chỉ hỏi lần đầu, nhưng "chỉ hỏi lần đầu" vẫn là một bước tương tác thủ công trong một công cụ tự xưng là headless); `grim` chỉ chạy trên compositor wlroots — Mutter và KWin đều không hỗ trợ. Đây là vấn đề thật, chỉ là **giải được bằng cách bỏ desktop session thật đi**: `Xvfb :99` + openbox + `import`, hoặc `sway --headless`/`cage` + `grim`, cho bạn capture tự động 100%, $0, không dialog, không phụ thuộc GNOME phiên bản nào. Cái giá phải trả là bạn không còn test được tray icon/StatusNotifierItem, portal, CSD, window chrome — tức là đúng nhóm bug DE-specific mà "desktop VM" được bán để bắt.

*Phần thị trường, không đổi:* toàn bộ giá trị của Otherside là "VM Linux zero-config trên Mac" — mảng đã bị commoditize hoàn toàn ở $0 bởi Lima/Colima/OrbStack/Apple `container`, trong đó Lima là dự án CNCF có nhân lực tài trợ. Và Apple `container` (Apache-2.0, v1.2.2 ngày 08/08/2026, mỗi container một micro-VM trên VZ, có `--arch amd64` qua Rosetta) làm phần lớn công việc dựng máy Linux sạch mà bạn định tự viết bằng Swift — miễn phí, first-party, chỉ đòi macOS 26.

Ước tính 8-10 tuần → thực tế **4-6 tháng** (hạ từ "5-7 tháng" của bản gốc, vì phần daemon và phần capture headless đơn giản hơn panel tưởng).

**Kết luận: KHÔNG NÊN làm nguyên bản. NÊN lấy:**
- Hợp đồng `side run <artifact>` như một **protocol độc lập với backend** — đây là tài sản duy nhất trong concept không bị vô hiệu hoá bởi bất cứ giới hạn nào của Apple Silicon.
- **Fidelity Notice** dán kèm mọi output.
- Reset máy sạch tức thì — nhưng khi triển khai, dùng `systemd-nspawn --ephemeral` trên một máy Linux thật thay vì `clonefile(2)` + boot VM (xem §4.4); con số thật của đường VM là thời gian boot ~15-20s, không phải 5ms của bước clone.
- Kỷ luật "cắt tàn nhẫn xuống 1 platform".
- **Bài học ngược từ phản bác:** tách rõ hai lane ngay từ thiết kế — `clean` (headless Xvfb/wlroots, tự động 100%, tin cậy) và `desktop` (session DE thật, opt-in, cần người bấm, dùng để bắt bug DE-specific). Đừng bán lane `desktop` như thể nó chạy được không người trực.

---

## Hạng 2 — Ferry (4.0/10)

> **"`ferry test --on windows,ubuntu` — máy sạch mỗi lần, verdict PASS/FAIL/NOT-COVERED, có nhãn fidelity."**

**Mô tả.** CLI-first test matrix runner. VZ cho Linux, QEMU+HVF cho Windows. Ephemeral **state** (clone copy-on-write mỗi run) chứ không ephemeral process (warm pool). Mọi kết quả gắn nhãn `native-arm64` / `emulated-x64-prism` / `real-x64-remote`. `ferry doctor --project` quét repo và báo chính xác cái gì VM ARM64 KHÔNG phát hiện được. AV & SmartScreen prescan.

**Cho ai.** Dev Electron/Tauri đã bị installer flag; **vòng ưu tiên cao nhất: dev .NET MAUI/WinUI3 và Flutter desktop** — hai stack này bị chặn ở mức **tooling** (MAUI cần Windows SDK + MSBuild trên Windows cho target `net*-windows`; `flutter build windows` đòi Visual Studio với C++ desktop workload và chỉ chạy trên host Windows), nên VM/máy Windows là điều kiện bắt buộc chứ không phải tiện nghi.

**Điểm TB: 4.0**

**Phản bác mạnh nhất.** *Kiến trúc chọn sai backend cho cả hai tính năng đầu bảng — và sai đối xứng.*

Trên Linux (VZ): VZ **không có API đọc framebuffer**; đường hiển thị duy nhất là `VZVirtualMachineView`, tức một NSView phải nằm trong app có GUI — mâu thuẫn trực tiếp với "KHÔNG CÓ GUI" trong `mvp_scope`. Đây là phản bác thật và không bị kiểm chứng làm suy yếu. (Cách vòng qua: chụp **bên trong** guest bằng agent + Xvfb/`grim` rồi đẩy file ra qua virtiofs/vsock — nhưng lúc đó bạn không cần VZ view, và cũng không cần phần lớn giá trị mà "VM desktop" hứa hẹn.)

Trên Windows (QEMU+HVF): có `screendump` qua QMP nhưng **không có warm resume**. `savevm`/`loadvm` với `-accel hvf` trên aarch64 không được hỗ trợ. Và phương án thay thế "clone qcow2 overlay rồi `cont`" là **sai về nguyên lý**: không thể tháo overlay đang mount khỏi một guest đã boot rồi gắn clone mới vào — guest memory đang giữ page cache/NTFS metadata coherent với đĩa cũ, filesystem hỏng ngay. Nghĩa là mỗi run Windows = boot lạnh, và lời hứa "máy sạch mỗi lần, nhanh" chỉ đúng ở cột Linux.

Cộng thêm **nghịch đảo WTP**: kiến trúc `ephemeral state` + `clone rồi vứt` + `không GUI` là thứ độc hại nhất với đúng segment có willingness-to-pay cao nhất. Dev MAUI/Flutter cần một máy dev **BỀN**: VS Build Tools ~10-20GB, build cache 20-40GB, NuGet/pub cache, symbol. Vứt máy sau mỗi run nghĩa là mỗi lần chạy phải dựng lại toàn bộ — bạn bán "sạch" cho nhóm đang cần "ấm".

Ước tính 4-4.5 tháng → thực tế **8-12 tháng**.

**Kết luận: KHÔNG NÊN làm nguyên bản. NÊN lấy:**
- **Fidelity labeling ba trạng thái PASS / FAIL / NOT-COVERED.** Đây là phát minh khái niệm giá trị nhất của Ferry: từ chối biến "chưa test được" thành "pass".
- **`doctor --project` static analyzer** — chạy trên chính Mac, không cần VM, không cần license, ship trong 2-4 tuần.
- **Defender/SmartScreen prescan.**
- Nguyên tắc "CLI **từ chối** phát ngôn *app của bạn chạy trên Windows* khi mới chỉ test trên ARM64".

---

## Hạng 3 — Cuaso / Cửa Sổ (3.9/10)

> **"App Windows chạy như app Mac cho người dùng thường VN — và nói thật cái gì không chạy được."**

**Mô tả.** Không nhắm dev. Nhắm kế toán/nhân viên văn phòng VN kẹt 1-3 phần mềm Windows. Tính năng số 1 là **Compatibility Oracle**: gõ tên phần mềm (HTKK, MISA, Fast, BHXH...) → trả về 1 trong 4 trạng thái đã test thật + lý do dễ hiểu + giải pháp thay thế. Cài đặt một lần bấm. Seamless window qua RDP RemoteApp/RAIL. Bảng Sự Thật công khai cái gì KHÔNG chạy.

**Cho ai.** Người dùng Mac không phải dev ở VN/ĐNA, M1+ với ≥16GB RAM.

**Điểm TB: 3.9**

**Phản bác mạnh nhất (hai đòn, cả hai đều nặng).**

*Kỹ thuật:* toàn bộ định vị nằm trên một thành phần **chưa ai trên thế giới có sẵn trên macOS**. FreeRDP client macOS và `sdl-freerdp` **không hỗ trợ RAIL/RemoteApp** — RAIL chỉ có reference implementation ở client X11. Và chính Microsoft, với source code RDP và ngân sách vô hạn, **không ship được tính năng này trên macOS**: bảng "Compare Remote Desktop clients" của Microsoft Learn ghi "Integrated apps" = có trên Windows, không có trên macOS ở cả ba pivot (RDS, AVD, Windows 365), sau ~15 năm có client macOS. Parallels làm được Coherence bằng display driver kernel-mode + 20 năm tích luỹ — con đường mà concept đã (đúng) loại bỏ. Ước tính 12 tháng → thực tế **24-30 tháng**.

Thêm: "máy in Mac dùng được trong Windows mà KHÔNG cần driver Windows" là sai — client macOS chỉ hỗ trợ PostScript generic, "Native printer drivers aren't supported". Thêm: suspend `savevm` 8GB mỗi 5 phút = ~160GB ghi/ngày ≈ **~50TB/năm** trên SSD hàn cứng không thay được của MacBook.

*Thị trường:* **gọng kìm chi phí.** Windows 11 Pro **OEM** khoá cứng vào thiết bị vật lý đầu tiên, **không được chuyển sang VM** — đây là điểm pháp lý chắc chắn, không phải phỏng đoán. Bản hợp pháp cho VM là **FPP/Retail**, được phép transfer. Con số VND cụ thể (~2,31 triệu OEM / ~4,79 triệu FPP) **độ tin cậy Thấp, chưa kiểm chứng được trong phiên này** — nhưng kết luận không phụ thuộc vào con số chính xác: tổng đường "Cuaso hợp pháp" (phí app + license FPP) rơi vào **cùng bậc giá với một mini PC N100 hoặc một laptop Windows cũ**, mà những thứ đó là Windows x86-64 **thật**, chạy được USB token, SQL Server, app 32-bit. Ở VN, mini PC/laptop cũ **thường đã kèm license OEM**, làm chênh lệch còn tệ hơn cho Cuaso. Cuaso chỉ rẻ hơn khi user dùng key lậu — nhưng concept đã tự tuyên bố không bán key. **Nhóm sẵn sàng trả tiền không cần bạn; nhóm cần bạn không trả tiền.**

Và moat #2 (kênh đại lý) sẽ không bao giờ kích hoạt: hoa hồng 30% × 690k = 207k/máy, trong khi nhân viên ShopDunk ăn hoa hồng trên MacBook 25-40 triệu — bạn đang nhờ họ làm một việc vừa tốn công vừa **hại doanh số chính của họ** (phải nói với khách "USB token của chị sẽ không chạy") để lấy 207k.

*Và biggest_risk tự thân:* kế toán chạy HTKK được, kết xuất tờ khai được — nhưng bước **cuối cùng** là ký và nộp bằng USB token, mà driver token của các CA Việt Nam là kernel-mode x86/x64, **không bao giờ** chạy dưới Prism. Sản phẩm "chạy được phần mềm" nhưng không "hoàn thành được công việc". Sắc thái cần verify: thị trường VN đang dịch chuyển mạnh sang **ký số từ xa** (VNPT SmartCA, Viettel-CA MySign, FPT.eSign) và kê khai web — điều này vừa làm biggest_risk bớt chí mạng, vừa làm **chính nỗi đau "cần Windows" tự tiêu biến**. Cả hai chiều đều không cứu được concept.

**Kết luận: KHÔNG NÊN làm nguyên bản.**

**NÊN lấy — và đây là thứ đáng giá nhất trong cả 5 concept về mặt kinh doanh thuần:** **Compatibility Oracle tách ra làm một sản phẩm độc lập, không cần một dòng code hypervisor nào.** Một CSDL tra cứu được kiểm chứng thật cho câu hỏi "phần mềm X có chạy được trên MacBook Apple Silicon không, và bằng cách nào". Chi phí hạ tầng ~$0, một người làm được, bắt đúng intent tìm kiếm có ý định mua ("HTKK cho MacBook", "MISA trên Mac"), và **kiếm tiền từ 100% người ghé thăm** bằng định tuyến affiliate (Parallels / mini PC / bản web-cloud của chính phần mềm đó / ký số từ xa / Windows 365) — kể cả 40% người mà câu trả lời là "không chạy được", đúng nhóm mà concept gốc phải từ chối bán và mất trắng.

Kèm nguyên tắc **TỪ CHỐI BÁN** khi biết trước sản phẩm không giải quyết được việc của khách — thứ đối thủ không thể copy bằng tiền. (Kiểm tra sớm: iTaxViewer chạy trên JRE; nếu nó chạy thẳng trên JVM macOS thì câu trả lời của Oracle cho một trong hai truy vấn traffic cao nhất sẽ khác hẳn.)

---

## Hạng 4 — Triptych (3.8/10)

> **"Một màn hình app, ba hệ điều hành, một khung hình — và chỉ ra chính xác chỗ nào giao diện vỡ."**

**Mô tả.** Visual QA cross-platform chuyên sâu. Probe JS nhúng trong app dump layout graph chuẩn hoá. Lane matrix (os × dpi × theme × locale). Pseudolocalization tích hợp. 6 rule cross-OS (no-clipped-text, no-out-of-bounds, no-overlap, box-drift, font-fallback, contrast) thay vì so pixel. **Không tạo VM** — gắn vào hypervisor/máy user đã có, kể cả mini PC N100.

**Cho ai.** Team Electron/Tauri **đã có sẵn** một môi trường Windows nhưng không có quy trình kiểm tra giao diện.

**Điểm TB: 3.8**

**Phản bác mạnh nhất.**

*Kỹ thuật, đòn chính (không bị kiểm chứng làm suy yếu):* DPI matrix — tính năng đặt cược để biện minh giá trị — là **thành phần không điều khiển được bằng code**. API chính (`DISPLAYCONFIG_DEVICE_INFO_SET_DPI_SCALE`) dùng type value **âm** nằm ngoài enum công khai, được reverse-engineer từ Settings app, và **không nhận giá trị tuyệt đối** 100/125/150 — nó nhận chỉ số **tương đối** bị chặn bởi `minScaleRel`/`maxScaleRel` mà Windows tự tính từ EDID; trong VM thì EDID là tổng hợp → **không đảm bảo** tập scale step chứa 125/150/175. Fallback registry đòi **sign-out/reboot** mỗi ô ma trận. Fallback cuối (`--force-device-scale-factor`) là **fallback tự lừa dối**: nó chỉ scale DPR trong renderer, KHÔNG đổi system metrics, KHÔNG đổi non-client area, KHÔNG đổi font hệ thống, KHÔNG kích hoạt `WM_DPICHANGED` — tức là **chính xác toàn bộ lớp bug đang hứa bắt**. Lane "windows/125" chạy bằng fallback sẽ in "22 checks passed" trong khi máy Windows thật đang vỡ. **Một công cụ QA báo PASS sai còn tai hại hơn không có công cụ.**

Thêm: rule `font-fallback` không implement được bằng agent JS — `getComputedStyle().fontFamily` trả về giá trị **đã khai báo**, CSSOM không expose used font.

*Kỹ thuật, đòn về cột Linux — đã hiệu chỉnh:* panel gốc viết "cột Linux là cột rỗng vì GNOME 49/50 đã xoá X11". **Sai.** XWayland vẫn tồn tại, và một lane `Xvfb + X11` chạy hoàn hảo, headless, $0, bền qua mọi phiên bản GNOME. Nhưng **phiên bản mạnh của phản bác vẫn đứng nguyên, và nó mới là phiên bản quan trọng**: một lane X11-only **không bao giờ bắt được** đúng lớp bug được trích ra để chứng minh nỗi đau Linux (Tauri #14234: tray icon mất trên Wayland, bình thường trên X11). Cột Linux không rỗng — nó **mù đúng chỗ đau**. Muốn bắt lớp bug đó phải chạy session Wayland thật, và ở đó capture tự động lại vướng portal + dialog xin phép. Đây vẫn là mâu thuẫn thật, chỉ khác chỗ: nó là bài toán **chọn môi trường**, không phải bài toán "công cụ chụp màn hình đã tuyệt chủng".

*Thị trường:* **cả một ngành "visual regression testing" đã chứng minh không tồn tại ở mức $12/tháng.** Percy free 5.000 screenshot rồi nhảy lên ~$199/tháng; Chromatic Starter $149-179/tháng; Applitools bắt đầu $899-969/tháng. Không sản phẩm nào bán được cho solo dev ở $8-15 — vì sản phẩm dùng theo **nhịp release**, không phải hằng ngày. Triptych lại neo giá theo OrbStack $8 / Docker $11, là **hai công cụ infrastructure chạy 8 tiếng/ngày**. Cộng phép lọc đối tượng tự giết mình: 6 điều kiện đồng thời, trong đó điều kiện "**đã có sẵn** môi trường Windows" **loại bỏ đúng nhóm có pain cao nhất**. Ước lượng thật ~300-900 khách, thấp hơn mục tiêu 1.500-3.000 một bậc độ lớn. Và JTBD này **không có search demand** — không ai gõ Google "cross-platform desktop UI diff tool".

**Kết luận: KHÔNG NÊN làm nguyên bản. NÊN lấy ba thứ, đều rất giá trị:**
1. **Kỷ luật "lane"**: KHÔNG BAO GIỜ so pixel chéo OS (bài toán ill-posed — font, hinting, gamma, subpixel khác nhau, kết quả luôn 100% đỏ). Chỉ assert **bất biến phát biểu được bằng lời** khi so chéo OS, và chỉ so pixel **trong cùng một lane** với baseline của chính nó. Đây là insight đúng nhất trong cả panel.
2. **Pseudolocalization + no-clipped-text**: chạy 100% trên chính máy Mac, không cần VM, không cần Windows, không cần license — một công cụ **2-3 tuần** bắt được lớp bug tràn chữ/cắt chữ THẬT.
3. **Bring-your-own-target với reverse-connect agent**: một lệnh `curl`/`iwr` biến BẤT KỲ môi trường Windows/Linux user đã có thành test target được quản lý; agent tự kết nối **ngược** về host qua WebSocket nên **không cần port forwarding**, chạy xuyên NAT của mọi hypervisor. Đây là mảnh **bền nhất** trong cả panel.

Bổ sung một cảnh báo mà cả Triptych lẫn panel đều bỏ sót: trước khi xây rule engine dựa trên screenshot + pixel-stability, phải so với thứ đã tồn tại miễn phí — Playwright `_electron.launch()`, WebdriverIO electron-service, `flutter drive`/`integration_test`, Appium Windows Driver/FlaUI. Assertion thật trên cây UI thật luôn ít flaky hơn suy diễn từ ảnh. Screenshot là công cụ đúng cho **cái không có API để hỏi**: window chrome, tray, dialog hệ thống, installer UI, font fallback thực tế.

---

## Hạng 5 — Rig (3.6/10)

> **"Vòng lặp ARM local 3 giây, verify trên x64 THẬT 3 phút, cùng một lệnh — và luôn nói thật bạn vừa test được cái gì."**

**Mô tả.** Hybrid local + cloud. Windows 11 ARM64 VM local (QEMU+HVF) lo vòng lặp nhanh; burst lên Windows x64 thật (chạy bằng tài khoản GitHub Actions của chính user) cho verify cuối. **Fidelity Ledger** dán nhãn mọi kết quả. Delta artifact sync (FastCDC + zstd + convergent encryption). `rig attach` mở phiên tương tác vào máy x64 khi fail.

**Cho ai.** Dev solo/team 2-15 người ship desktop app có bản Windows.

**Điểm TB: 3.6**

**Phản bác mạnh nhất.**

*Kỹ thuật:* **vòng lặp 3 giây — trái tim sản phẩm — là bất khả thi**, và vỡ ở cả hai vế.

**Vế build.** Panel gốc viết "không cross-compile được từ macOS ra Windows" — **quá rộng và sai**. Sự thật chia ba nhóm:

| Nhóm | Thực tế |
|---|---|
| Cross-compile sạch, không cần Windows | Go `GOOS=windows GOARCH=amd64/arm64` với `CGO_ENABLED=0` chạy hoàn hảo trên Apple Silicon. Rust tương tự với `*-pc-windows-gnu`. Ký Authenticode cho `.exe`/`.dll`/`.msi` từ macOS bằng `osslsigncode` (kể cả timestamp RFC3161) — **làm được**, trừ khi chứng chỉ OV/EV nằm trong HSM/token theo yêu cầu CA/B Forum từ 2023, lúc đó phải qua cloud signing (Azure Trusted Signing, DigiCert KeyLocker). |
| Cross-compile có điều kiện | `electron-builder` build NSIS installer cho Windows từ macOS/Linux được, đường chính thống là image `electronuserland/builder:wine` chạy trên Linux/CI (Wine trên macOS ARM rất khó dựng). Không build được APPX/MSIX và MSI đầy đủ. Tauri sang `*-pc-windows-msvc` qua `cargo-xwin` hoạt động nhưng **không phải luồng chính thức**, và `cargo-xwin` tải header/lib MSVC từ Microsoft — EULA chỉ cho phép trên máy có giấy phép Visual Studio, dùng thương mại là vùng xám. (Trạng thái 2026 của hai mục này: **độ tin cậy Thấp**, không verify được.) |
| Bắt buộc cần Windows | Native module Electron (better-sqlite3, keytar, node-canvas — **chính là beachhead của Rig**) đòi node-gyp + MSVC, macOS không có `link.exe`. .NET MAUI target `net*-windows` cần Windows SDK + MSBuild. `flutter build windows` đòi Visual Studio C++ workload trên host Windows. MSIX/APPX, MSI qua WiX, driver, và **testing thật** — tất cả cần Windows. |

Nghĩa là phản bác **mất tính phổ quát nhưng giữ nguyên sức nặng ở đúng chỗ Rig đứng**: nhóm khách hàng của Rig là nhóm ở hàng thứ ba. Với họ, build phải xảy ra **bên trong** guest → phải nhét VS Build Tools ~10-20GB vào golden image → vòng lặp thành 30 giây đến 5 phút, không phải 3 giây.

**Vế sync.** **virtiofs từ macOS host vào Windows ARM64 guest qua QEMU KHÔNG TỒN TẠI** — `virtiofsd` là Linux-only, QEMU không build virtfs/9p trên macOS, virtiofs của Apple chỉ có trong VZ (mà VZ không boot được Windows), và driver virtio-fs ARM64 còn thiếu trong virtio-win. **Bằng chứng xác nhận mạnh nhất: chính Lima v2.2 — codebase mà Rig định vendor logic Windows từ đó — đánh dấu template Windows là `plain: true`, tức TẮT file mount và port forwarding, chính xác vì thứ này không làm được.** Rig định vay mượn một blueprint mà blueprint đó có sẵn dòng chữ "phần này không chạy".

Thêm hai lỗ hổng fidelity tự sát, **cả hai đều đã được xác minh trực tiếp**:

(a) `windows-latest` hiện trỏ tới **Windows Server 2025**, không phải Windows 11 client → cả hai target trong MVP đều không phải môi trường thật của user cuối. (Ghi chú cập nhật: `windows-11-arm` **đã mở cho cả public lẫn private repo**, không còn giới hạn public-only — nên nếu tài liệu gốc nói ngược lại thì phải sửa. Số core: 4 cho public, 2 cho private.)

(b) GitHub Actions runner image **chủ động tắt Defender**. Đọc trực tiếp `images/windows/scripts/build/Configure-WindowsDefender.ps1` trong `actions/runner-images`: header ghi thẳng "Desc: Disables Windows Defender", script set `DisableRealtimeMonitoring`, `DisableBehaviorMonitoring`, `DisableScriptScanning`, `DisableIOAVProtection`, `EnableNetworkProtection = "Disabled"`, `ExclusionPath = C:\, D:\`, và cuối script:

```powershell
$atpRegPath = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection'
if (Test-Path $atpRegPath) {
    Set-ItemProperty -Path $atpRegPath -Name 'ForceDefenderPassiveMode' -Value '1' -Type 'DWORD'
}
```

(Sắc thái: `ForceDefenderPassiveMode` chỉ được set **nếu** registry path ATP tồn tại.) Hệ quả: verdict "sạch" từ CI là **vô nghĩa** nếu không re-enable + update signature + bật cloud protection — và phần lớn false-positive của electron-builder là verdict ML đám mây (`!ml`) phụ thuộc prevalence/reputation, **không tái hiện được trên CI** dù có bật lại Defender. Hệ quả phụ: đừng lấy thời gian build trên runner Windows làm chuẩn suy ra thời gian build trên máy Windows thật của user.

*Thị trường:* **tần số của job không khớp với mô hình giá, và free tier đã phủ 100% nhu cầu trả tiền.** Hai tính năng duy nhất đáng mở ví (Defender prescan, verify x64) đều ở nhịp **release** — team nhỏ release 1-4 lần/tháng → chạm phần trả tiền 4-10 lần/tháng. Concept lại cho free tier **50 burst verify/tháng + FULL local loop** — rộng gấp 5-10 lần nhu cầu thực. **Sẽ không ai nâng cấp, không phải vì keo kiệt mà vì không bao giờ chạm trần.** Còn tính năng có tần số hằng ngày (`rig watch`) đòi giữ một VM Windows 8GB+ chạy thường trực trên MacBook 16GB — mà chính Fidelity Ledger liên tục nói với họ là "không đại diện". **Sản phẩm tự bắn vào chân mình: nó thuyết phục user rằng lớp 4-tháng-công-sức-kỹ-thuật của nó là lớp đáng ngờ nhất.**

**Kết luận: KHÔNG NÊN làm nguyên bản. NÊN lấy:**
- **Fidelity Ledger + `rig doctor`** — đã được 4/10 giám khảo độc lập chọn giữ.
- **BYO-metal**: agent dial-out từ mini PC / laptop Windows cũ của chính user. Rẻ nhất để làm (chỉ là một transport thứ ba của cùng một agent), và là thứ **DUY NHẤT** cho ra verdict đúng.
- **Defender/SmartScreen prescan trên Windows x64 thật có Defender bật thật** — nói rõ trong doc rằng CI không thay thế được bước này, và nêu lý do (`ForceDefenderPassiveMode`, `!ml` phụ thuộc prevalence).
- **Bổ sung mà Rig bỏ sót và rất rẻ:** VM Windows trên cloud **tính theo giờ** (Azure B2s Windows ~$0.10/h, AWS t3.medium Windows ~$0.06-0.09/h; license Windows **Server** gộp trong giá; `az vm create/deallocate/delete`, `aws ec2 run-instances/create-image/terminate-instances` đầy đủ). 20 giờ/tháng ≈ **$2 compute**, tổng thực tế **$4-12/tháng** kể cả disk/snapshot — rẻ hơn Windows 365 Business ($28-56/user/tháng) khoảng 3-7 lần; điểm hoà vốn ~250-300 giờ/tháng. Cảnh báo phải nói kèm: đó là **Windows Server**, không phải **Windows 11 client**; muốn đúng Windows 11 client hợp pháp thì vẫn vướng licensing (Windows 365 / AVD / dedicated host BYOL / QMTH). Độ tin cậy: Trung bình-Cao cho Azure/AWS, Thấp cho GCP.

---

## 3.6. Bảng tổng hợp phiếu "GIỮ LẠI"

Đây là dữ liệu quan trọng nhất trong toàn bộ panel — 10 phản biện độc lập, 10 phiếu:

| Thứ được giữ | Số phiếu | Đặc điểm chung |
|---|---|---|
| **Fidelity knowledge / doctor / ledger / Oracle** | **6/10** | Không cần VM, không cần hypervisor, không cần license, ship 2-4 tuần, moat **tích luỹ theo thời gian** |
| **Hợp đồng `run <artifact>` tách rời backend** | **3/10** | Không bị vô hiệu hoá bởi giới hạn Apple Silicon, không bị copy trong một release |
| **BYO-target / reverse-connect agent** | **2/10** | Biến đối thủ mạnh nhất (mini PC) thành backend |
| **Nguyên tắc trung thực (Fidelity Notice / từ chối bán)** | **5/10** | Marketing asset tái sử dụng được ở mọi concept |
| Hypervisor engineering | **0/10** | Không ai giữ |

---

## 4. KHUYẾN NGHỊ CHÍNH

## 4.1. Quyết định

**Xây một sản phẩm hai lớp, tạm gọi là `firstrun`** *(tên phải tra USPTO/EUIPO/npm/domain trước khi công bố — tên là thứ rẻ nhất để đổi ở tuần 1 và đắt nhất ở tháng 12. Tên thay thế: Coldstart, Portage, Proofrun.)*

> **Lớp 0 — `firstrun scan` (4 tuần, MIỄN PHÍ, không cần VM):** static analyzer + fidelity knowledge base. Trả lời câu hỏi chưa công cụ nào trả lời — *"với setup Mac-only của bạn, chính xác lớp bug nào bạn KHÔNG BAO GIỜ nhìn thấy trước khi user nhìn thấy?"*
>
> **Lớp 1 — `firstrun run` (tuần 5-16, có phí):** hợp đồng chuẩn hoá `run <artifact> --on <target>`, với target là **BẤT KỲ** máy nào: một VPS x86-64 của bạn, mini PC N100 của bạn, laptop Windows cũ, máy công ty, GitHub Actions runner, một Windows VM cloud tính theo giờ, hoặc (sau cùng) một Linux VM local trên VZ.
>
> **KHÔNG BAO GIỜ:** sở hữu hypervisor cho Windows. Sở hữu compute. Redistribute Windows.

Một hiệu chỉnh về con số VPS so với bản nháp trước — **tất cả các số dưới đây độ tin cậy Thấp, chưa verify được bằng nguồn trực tuyến trong phiên này, bạn phải tự kiểm tra trước khi in vào doc**:

| Lựa chọn | Giá ước tính | Latency từ VN | Ghi chú |
|---|---|---|---|
| Hetzner CX22 (EU: Falkenstein/Nuremberg/Helsinki) | ~€3.79 + ~€0.50 IPv4 ≈ **€4.3/tháng** | **~230-300ms**, vọt 350ms+ khi đứt cáp | Rẻ nhất. Dòng CX/CAX **chỉ có ở châu Âu**. Hetzner nổi tiếng khó với khách VN: hay đòi xác minh danh tính, hay từ chối thẻ phát hành tại VN. Tốt cho CI/build batch, tệ cho phiên tương tác |
| Hetzner Singapore (chỉ dòng CPX/CCX) | CPX21 ~**€8-10/tháng** | ~25-50ms | Có phụ phí vị trí |
| Vultr / DigitalOcean / Akamai-Linode Singapore, 2 vCPU 4GB | **$20-24/tháng** | ~25-50ms (HCM 25-40, HN 40-60) | Thẻ VN dễ qua nhất |
| VPS trong nước (Vietnix, AZDIGI) | **200-450k VND/tháng** | <10-20ms | Hoá đơn VAT, hỗ trợ tiếng Việt |
| Oracle Cloud Always Free | **$0** | tuỳ region | 2 instance AMD x86 + ARM; đáng thử trước khi trả tiền |

Tổ hợp hợp lý cho hoàn cảnh của bạn: một box EU rẻ nhất cho CI/build không tương tác, cộng một box Singapore cho phiên tương tác/xem màn hình. Nếu chỉ chọn một, chọn Singapore.

## 4.2. Tại sao đây là lựa chọn đúng cho ĐÚNG hoàn cảnh của bạn

**(a) Bạn là dev solo có Mac. Lớp 0 không cần một dòng code virtualization nào.** Không Swift, không Virtualization.framework, không QEMU, không swtpm, không entitlement Apple, không GPL, không license Windows, không QMTH. Một binary Go/TypeScript. Bạn ship được nó **trong lúc vẫn đi làm**, trong 4 tuần, với rủi ro kỹ thuật gần bằng 0. Bốn trong năm concept đều chết vì chính phần bạn đang bỏ qua.

**(b) Nó giải quyết vấn đề của CHÍNH BẠN trước, ngay tuần 6.** Bạn không có máy Windows/Linux. Lớp 1 chĩa vào một VPS x86-64 cho bạn Linux **x86-64 thật** (fidelity cao hơn mọi VM ARM local), và chĩa vào một mini PC, một laptop Windows cũ, GitHub Actions, hoặc một Azure/EC2 VM theo giờ cho bạn Windows x64 **thật**. Dogfooding là con đường thực tế duy nhất để tích luỹ được corpus tương thích — thứ moat duy nhất trong cả panel tăng dần theo thời gian.

**(c) Nó đảo ngược thứ tự build của mọi concept đã bị bác — và đó chính xác là lý do nó sống.** Cả 5 concept đều bắt đầu bằng "dựng VM", rồi mới đến "chạy artifact". Sai. Backend đầu tiên phải là **SSH tới một máy Linux bất kỳ** — vì nó mất 2 tuần thay vì 6 tháng, cho fidelity x86-64 thật thay vì arm64, và **nó ép bạn thiết kế `run` như một protocol ngay từ đầu** thay vì như một tính năng của app VM. Local VM trên VZ là backend **thứ tư**, thêm vào khi đã có người dùng, thuần tuý vì nó offline và chi phí biên bằng 0. (Và khi đến lúc đó: cân nhắc dùng Apple `container` — Apache-2.0, v1.2.2, micro-VM/container trên VZ, có `--arch amd64` qua Rosetta — thay vì tự viết daemon Swift. Đổi lại là ràng buộc macOS 26.)

**(d) Nó vô hiệu hoá đối thủ mạnh nhất thay vì đối đầu.** Mini PC N100 $250 (hoặc laptop Windows cũ 2-4 triệu VND, có sẵn màn hình, bàn phím và **pin làm UPS** — rất đáng giá ở VN) là máy Windows x86-64 thật. Bạn không thắng được nó. **Nên bạn biến nó thành backend.** Câu "sao tôi không mua mini PC?" trở thành "mua đi, `firstrun` điều khiển giúp — deploy, chạy, thu log, revert bằng Hyper-V checkpoint hoặc Windows Sandbox". Tương tự: UTM/Fusion/Parallels đã có VM của user → chúng trở thành target, không phải đối thủ. Cảnh báo mua hàng phải in đậm trong doc: **phần lớn mini PC giá rẻ ship Windows 11 Home, mà Home KHÔNG có Hyper-V và KHÔNG có Windows Sandbox** — kiểm tra edition trước khi mua, hoặc cộng tiền nâng Pro, hoặc dựng fallback bằng image restore/ổ thứ hai.

**(e) Nó giải bài toán tần số — thứ đã giết 4/5 concept.** VM test chạy 2-8 lần/năm. Nhưng `firstrun scan` chạy được **mỗi lần push** (nó chỉ là static analysis, mất 3 giây), và `firstrun run` gắn được vào PR bot. Đó là khác biệt cấu trúc giữa "devtool tiện nghi" và "quality gate" — và **quality gate thì có ngân sách**.

**(f) Nó có kênh phân phối, trong khi 4/5 concept thì không.** `npx firstrun scan` chạy trong 3 giây, không cần cài gì, không cần VM, không cần Windows, không cần đổi hành vi. Output của nó là **một bảng đáng chụp màn hình đăng lên HN/Twitter**. Và nó ký sinh được vào hệ sinh thái: PR vào doc của electron-builder, plugin cho `tauri-action`, một GitHub Action. Đây là điều duy nhất trong toàn bộ panel có cơ chế lan truyền.

## 4.3. Các "steal_this" được ghép vào

| Lấy từ | Thứ được ghép | Vào đâu |
|---|---|---|
| **Rig / Ferry** | Fidelity Ledger + verdict 3 trạng thái **PASS / FAIL / NOT-COVERED** | Lõi của Lớp 0 và mọi output của Lớp 1 |
| **Cuaso** | Nguyên tắc **TỪ CHỐI BÁN** + Compatibility Oracle như tri thức tích luỹ | Lớp 0 mở rộng thành CSDL công khai |
| **Otherside** | Hợp đồng `run <artifact>` như **protocol**, không phải feature của app VM | Lõi của Lớp 1 |
| **Otherside** | Tách lane `clean` (headless Xvfb/wlroots, tự động 100%) và `desktop` (session DE thật, opt-in) | Thiết kế lane của Lớp 1, tuần 2 |
| **Triptych** | **Kỷ luật lane** (không so pixel chéo OS) + **pseudolocalization** + reverse-connect agent | Rule engine + transport của Lớp 1 |
| **Ferry / Rig** | **Defender/SmartScreen prescan** trên Windows x64 thật có Defender **bật thật** | Tính năng ROI cao nhất, tháng 3 |
| **Otherside** | Reset về máy sạch tức thì (nhưng bằng `systemd-nspawn --ephemeral`, không phải clonefile) | Lớp 1, tuần 3 |

## 4.4. Một quyết định kiến trúc mà không concept nào tìm ra

Cả 5 concept đều giả định "máy sạch mỗi lần test" đòi hỏi **VM + snapshot + copy-on-write disk image**. Sai.

Với bài toán *"cài `.deb`/`.rpm`/`.AppImage` này lên một máy sạch và chứng minh nó khởi động được"*, thứ bạn cần là:

```bash
systemd-nspawn --ephemeral -D /var/lib/machines/ubuntu2604 ...
```

**`--ephemeral` cho bạn một máy sạch trong ~2 giây, tự huỷ khi thoát, không hypervisor, không snapshot API, không APFS clonefile, không QEMU, không GPL.** Chạy song song được nhiều container. Và với nhiều rootfs được `debootstrap`/`dnf --installroot` sẵn, bạn có **cả một ma trận distro trên MỘT máy**:

```bash
firstrun run app.deb --on ubuntu2404,ubuntu2604,debian13,fedora44
```

bốn máy sạch, song song, trên một VPS duy nhất. Đây tốt hơn mọi thứ trong 5 concept cho trường hợp Linux.

Ba điều kiện để lời hứa "2 giây" là thật, nếu thiếu thì nó âm thầm trở thành 30-90 giây:

| Điều kiện | Chi tiết |
|---|---|
| rootfs phải là **btrfs subvolume**, không chỉ nằm trên filesystem btrfs | `btrfs subvolume create /var/lib/machines/ubuntu2604` **trước** khi debootstrap. Thiếu bước này, `--ephemeral` fallback sang copy toàn bộ |
| pre-bake dependency vào rootfs | `apt-get update` mỗi lần ephemeral tốn 20-40s — đủ để giết lời hứa ở mức end-to-end |
| DNS + FUSE | `--resolv-conf=copy-host`, và bind `/dev/fuse` nếu chạy AppImage (hoặc dùng `--appimage-extract-and-run`) |

**Giới hạn về fidelity (đưa thẳng vào NOT-COVERED):** nspawn container **chia sẻ kernel** với host → không test được kernel module, không test được kernel version khác, không test được driver. Nhưng cho *"app của tôi cài được và mở lên được không"* thì nó chính xác là công cụ đúng. Cần kernel khác → dùng VM thật, và lúc đó dùng lại đúng agent protocol.

**Giới hạn về BẢO MẬT — nghiêm trọng hơn giới hạn fidelity, và không được nhầm hai thứ:**

**Container KHÔNG phải security boundary.** `systemd-nspawn` và Docker **dùng chung kernel với host**; một lỗ hổng escalation trong kernel, một cấu hình capability lỏng, một bind mount sai chiều là đủ để thoát ra. Nghĩa là:

- Chạy một **installer chưa ký hoặc không rõ nguồn gốc** trong container = **đặt cả VPS vào rủi ro** — kèm theo mọi artifact, log, screenshot, SSH key và token của mọi dự án khác đang nằm trên máy đó.
- **Binary không tin cậy → VM dùng một lần, không phải container.** Đây là quy tắc cứng, không phải khuyến nghị. VM có ranh giới hypervisor; container thì không. Nếu bạn chạy `.exe`/`.deb` của người khác gửi tới (kiểm thử cho khách, phân tích một bản build lạ, tái hiện bug từ artifact user upload), dựng VM riêng, chạy xong xoá, không tái sử dụng.
- Với artifact **của chính bạn, tự build từ source của chính bạn**, container là hợp lý — rủi ro ở đây là bug, không phải malware.

Kèm theo, cấu hình tối thiểu cho cái VPS đó, vì nó có IP public và đang chạy agent:

| Hạng mục | Yêu cầu |
|---|---|
| SSH | `PasswordAuthentication no`, key-only, đổi/giới hạn port, fail2ban |
| Firewall | ufw deny mọi thứ trừ SSH |
| Xem màn hình từ xa | **Tuyệt đối không expose VNC/xrdp/3389 ra internet** — 3389 mở sẽ bị quét trong vài phút. Dùng `x11vnc -display :99 -forever -localhost` + SSH tunnel, hoặc Tailscale/WireGuard |
| Quyền agent | Không chạy agent bằng root nếu tránh được; tách user riêng cho mỗi loại target |
| Token | Backend `gha` cần PAT scope `workflow` = quyền ghi repo → dùng fine-grained token hoặc GitHub App. Cài agent bằng `curl \| sh` phải pin checksum/chữ ký |
| Dữ liệu thu được | Screenshot và log rất dễ chứa API key, email khách, token trong URL → cần bước redaction trước khi ghi vào `runs/` |

---

[← 2. Bản đồ lựa chọn](02-ban-do-lua-chon.md) · [Mục lục](README.md) · [4. Kiến trúc, MVP và roadmap →](04-kien-truc-mvp-roadmap.md)
