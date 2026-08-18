# new-project

Tài liệu brainstorm: **chạy Windows/Linux trên macOS Apple Silicon để test app
cross-platform**, cho trường hợp dev chỉ có máy Mac.

Bắt đầu ở **[docs/README.md](docs/README.md)**.

Bản web đọc liền mạch (mục lục dính, dark mode):
<https://claude.ai/code/artifact/268d4409-afcf-4d36-b08d-c24ddcf9b35c>

---

## Đọc gì trước

| Nếu bạn muốn | Đọc |
|---|---|
| Test app của mình **ngay tuần này** | [docs/06 — Làm gì ngay tuần này](docs/06-lam-gi-tuan-nay.md) |
| Biết cái gì **bất khả thi** trên Apple Silicon | [docs/01 — Sự thật nền tảng](docs/01-su-that-nen-tang.md) |
| Cân nhắc **xây sản phẩm** này | [docs/03 — Năm hướng và khuyến nghị](docs/03-nam-huong-va-khuyen-nghi.md) |

## Kết luận một dòng

Apple Silicon không chạy được Windows x86-64 — vĩnh viễn. Mảng "VM trên Mac" đã
commoditize ở $0. Không nên xây sản phẩm VM; việc có ROI cao nhất cho một dev Mac-only
là telemetry + kênh beta người dùng thật, rồi Playwright chạy ngay trên Mac, rồi
container `linux/amd64`, rồi CI — VM xếp gần cuối.

> **Lưu ý:** mọi con số **giá** trong tài liệu mang nhãn *độ tin cậy Thấp* — ngân sách
> tra cứu đã cạn trước khi verify xong. Tự kiểm tra trước khi chi tiền. Các ràng buộc
> kỹ thuật nền ở mức *Cao* và có nguồn.
