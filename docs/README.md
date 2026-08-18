# Máy Windows/Linux trên macOS — tài liệu brainstorm

Bối cảnh: một developer chỉ có máy Mac (Apple Silicon), làm app desktop cross-platform,
không có máy Windows/Linux để test. Câu hỏi ban đầu là *"xây một sản phẩm tạo máy
Windows/Linux trên macOS"*.

Tài liệu này trả lời hai việc **tách rời nhau**:

- **Sản phẩm đó có nên xây không** — phần 1 đến 5.
- **Bạn nên làm gì tuần này để test được app** — [phần 6](06-lam-gi-tuan-nay.md), hữu ích nhất trong ngắn hạn.

> **Nếu bạn chỉ có 20 phút:** đọc [phần 1](01-su-that-nen-tang.md) để biết cái gì bất khả
> thi, rồi nhảy thẳng xuống [phần 6](06-lam-gi-tuan-nay.md) để bắt tay làm.

---

## Mục lục

| | Tài liệu | Nội dung |
|---|---|---|
| 1 | [Sự thật nền tảng](01-su-that-nen-tang.md) | 11 ràng buộc kỹ thuật và pháp lý quyết định mọi phương án. Fidelity gap ARM vs x64. Ngân sách RAM / ổ cứng / pin |
| 2 | [Bản đồ lựa chọn](02-ban-do-lua-chon.md) | Parallels, Fusion, UTM, Lima, Apple `container`, GitHub Actions, VM cloud theo giờ, mini PC, VPS, và 4 đường license Windows hợp pháp |
| 3 | [Năm hướng sản phẩm và khuyến nghị](03-nam-huong-va-khuyen-nghi.md) | 5 concept + 10 phản biện độc lập, bảng điểm, và kiến trúc hai lớp được khuyến nghị thay thế |
| 4 | [Kiến trúc, MVP và roadmap](04-kien-truc-mvp-roadmap.md) | Sơ đồ kiến trúc, hợp đồng `run`, 16 rule của `scan`, 3 spike bắt buộc, MVP 4 tuần, roadmap 12 tháng |
| 5 | [Rủi ro và mô hình kinh doanh](05-rui-ro-va-kinh-doanh.md) | 3 bảng rủi ro, chi phí khởi động thật, bảng giá, thuế cho dev VN, dự phóng 4 kịch bản |
| 6 | [Làm gì ngay tuần này](06-lam-gi-tuan-nay.md) | **Bắt đầu ở đây nếu bạn cần test app ngay.** Script chạy được cho Sentry, Playwright, container, CI, VPS, cloud VM, UTM, mini PC |
| A | [Kết quả kiểm chứng lại](phu-luc-a-kiem-chung.md) | 8 điểm tài liệu gốc và bản phản biện mâu thuẫn nhau, đã tra cứu để xác định ai đúng |
| B | [Bản phản biện gốc](phu-luc-b-phan-bien.md) | ~40 lỗi và thiếu sót tìm được trong bản tổng hợp đầu tiên, giữ nguyên để đối chiếu |

Bản web đọc liền mạch, có mục lục dính và hỗ trợ dark mode:
**<https://claude.ai/code/artifact/268d4409-afcf-4d36-b08d-c24ddcf9b35c>**

---

Bạn là dev chỉ có máy Mac, làm app cross-platform, cần Windows/Linux để test. Câu hỏi ban đầu là *"xây một sản phẩm tạo máy Windows/Linux trên macOS"*. Tài liệu này trả lời hai việc tách rời nhau: **sản phẩm đó có nên xây không** (phần 2-9), và **bạn nên làm gì tuần này để test được app** (phần 10 — hữu ích nhất trong ngắn hạn).

### Sáu điều quyết định mọi thứ

1. **Apple Silicon không chạy được Windows x86-64. Vĩnh viễn.** Chỉ còn Windows 11 ARM64 + Prism (dịch app x64 ở user-mode), hoặc emulation toàn hệ thống chậm 5-20x mà chính Parallels vẫn gọi là "early preview" sau 1,5 năm.

2. **Test trên ARM64 không đại diện cho user chạy x64.** Bị bỏ sót hoàn toàn: driver kernel-mode, DX12/Vulkan, app 32-bit, WSL2/Hyper-V, native `.node` addon fail theo cách khác, và mọi số đo hiệu năng — AVX2 qua emulation trên M2 đo được **0.66x** so với SSE2, tức ngược dấu với x64 thật.

3. **Virtualization.framework của Apple không boot được Windows** (không có TPM 2.0, Windows ARM không có driver cho virtio của Apple). Muốn Windows thì bắt buộc QEMU + Hypervisor.framework. Đây là chỗ 4 trên 5 concept chết.

4. **Không hướng sản phẩm nào trong panel qua được 4.2/10.** Điểm thấp là kết quả đúng: mảng "chạy được Windows/Linux trên Mac" đã bị commoditize hoàn toàn ở $0 (VMware Fusion, UTM, Lima), và đối thủ nguy hiểm nhất không phải Parallels mà là **một `justfile` 40 dòng** người dùng tự viết.

5. **10 phản biện độc lập chỉ vào cùng hai thứ đáng giữ:** tri thức về *fidelity* (cái gì setup của bạn không bao giờ bắt được), và một hợp đồng `run <artifact>` tách rời khỏi backend. **0/10 phiếu giữ lại "xây hypervisor".**

6. **Việc có ROI cao nhất cho bạn không phải là VM.** Là telemetry + crash reporting + một kênh beta 5-10 người dùng Windows/Linux thật: chi phí $0, fidelity 100%, và nó cho bạn thứ không VM nào trên Apple Silicon cho được — bug thật, trên máy thật, của user thật.

### Bảng điểm 5 hướng sản phẩm

Mỗi hướng được thiết kế bởi một agent riêng từ một góc nhìn riêng, rồi bị hai giám khảo độc lập tấn công (một góc kỹ thuật, một góc thị trường) với nhiệm vụ *giết* nó, không phải khen nó.

| # | Hướng | Điểm | Chết vì |
|---|---|---|---|
| 1 | **Otherside** — Linux VM zero-config trên VZ + `side run <artifact>` | 4.2 | Chọn sai hình dạng môi trường: hứa desktop session thật rồi đặt tính năng thu tiền lên đúng thứ desktop hiện đại đã khoá. Và mảng "VM Linux trên Mac" đã miễn phí hoàn toàn |
| 2 | **Ferry** — `ferry test --on windows,ubuntu` | 4.0 | Chọn sai backend cho cả hai tính năng đầu bảng, đối xứng: Linux có reset nhanh nhưng VZ không có API đọc framebuffer; Windows chụp được nhưng không có warm resume |
| 3 | **Cửa Sổ** — seamless cho người dùng thường ở VN | 3.9 | RAIL/RemoteApp không tồn tại trên macOS — chính Microsoft cũng không ship được sau 15 năm. Và USB token chữ ký số là kernel-mode: không bao giờ chạy dưới Prism |
| 4 | **Triptych** — visual QA cross-platform | 3.8 | DPI matrix không điều khiển được bằng code; fallback duy nhất là thứ tự lừa dối, sẽ báo PASS sai. Ngành visual regression không tồn tại ở mức $12/tháng |
| 5 | **Rig** — hybrid ARM local + x64 cloud | 3.6 | virtiofs từ macOS host vào Windows guest **không tồn tại** — Lima v2.2 phải đánh dấu template Windows là `plain: true` đúng vì lý do đó |

Kết luận: **không nên làm nguyên bản hướng nào.** Phần 4 ghép các mảnh còn sống thành một khuyến nghị khác hẳn — một sản phẩm hai lớp không sở hữu hypervisor và không sở hữu compute.

---

<a id="ve-do-tin-cay"></a>

## Về độ tin cậy

Ba nhãn dùng xuyên suốt tài liệu:

| Nhãn | Nghĩa |
|---|---|
| **Cao** | Đọc trực tiếp từ tài liệu hoặc source chính chủ |
| **Trung bình** | Cơ chế kỹ thuật chắc chắn, con số cụ thể chưa verify lại |
| **Thấp** | Ước lượng từ kiến thức nền — **tự kiểm tra trước khi ra quyết định chi tiền** |

**Cảnh báo quan trọng:** ngân sách tra cứu web của phiên làm việc đã cạn trước khi xác
minh xong phần giá cả. **Mọi con số giá VPS, giá license Windows và giá Parallels đều
mang nhãn Thấp** — mở trang bán hàng ra xem trước khi chi tiền. Các ràng buộc kỹ thuật
nền (VZ không boot được Windows, Prism chỉ dịch user-mode, không có DX12, virtiofs
macOS→Windows không tồn tại, QMTH) ở mức Cao và có nguồn.

---

## Tài liệu này được làm thế nào

Ba đợt agent, tổng 31 agent:

1. **Nghiên cứu** — 4 agent song song: nền tảng ảo hoá Apple Silicon, Windows on ARM
   và vấn đề licensing, bản đồ đối thủ, nỗi đau thực tế của dev cross-platform.
2. **Thiết kế và phản biện** — 5 agent thiết kế 5 concept từ 5 góc nhìn độc lập; mỗi
   concept bị 2 giám khảo tấn công (một góc kỹ thuật, một góc thị trường) với nhiệm vụ
   *giết* nó, không phải khen nó. Rồi tổng hợp, và một critic soát lỗi toàn bộ.
3. **Kiểm chứng và hợp nhất** — 4 agent tra cứu lại các điểm mâu thuẫn, 6 agent viết
   lại từng phần với correction đã nhập.

Đợt 3 tìm ra khoảng 40 sai sót trong bản tổng hợp đầu — gồm cả những chỗ từng được gắn
nhãn *"đã kiểm chứng, độ tin cậy Cao"* mà vẫn sai. Hai ví dụ đáng chú ý:

- Khẳng định *"GNOME 49 xoá code X11 nên không chụp được màn hình tự động trong Linux
  guest"* là **sai**, và kết luận sai đó đã loại nhầm cả một lớp giải pháp $0. Sự thật:
  XWayland vẫn tồn tại, và cách đúng là không dùng GNOME chút nào — `Xvfb` + `import`
  chạy headless, không dialog xin phép, không phụ thuộc phiên bản GNOME.
- Khẳng định *"cloud không có API spin-up/tear-down theo build"* là **sai**. Azure/AWS
  có VM Windows tính theo giờ, license gộp sẵn trong giá, API đầy đủ, không dính QMTH —
  khoảng $4–12/tháng ở mức 20 giờ.

Các sửa chữa đã được nhập vào phần 1–6 và ghi rõ tại chỗ. [Phụ lục A](phu-luc-a-kiem-chung.md)
và [Phụ lục B](phu-luc-b-phan-bien.md) giữ nguyên dấu vết để đối chiếu.
