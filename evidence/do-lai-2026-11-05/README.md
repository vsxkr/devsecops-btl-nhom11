# Phép đo lặp lại ngày 05/11/2026

Thư mục này chứa kết quả đo lại toàn bộ các cổng kiểm soát trên hai nhánh, chạy
cục bộ bằng đúng phiên bản công cụ mà pipeline ghim. Lý do cần đo lại cục bộ:
trên nhánh `main`, pipeline dừng ngay ở stage 1 nên không bao giờ chạm tới stage
2 và stage 4, vì vậy số liệu đối chứng của nhánh đó không thể lấy từ run.

Phiên bản công cụ: Conftest 0.71.1 với OPA 1.21.1, Semgrep 1.182.1,
SonarQube Community 26.9.0.129388 với scanner 8.1.0.6389, Trivy 0.70.0,
Chef InSpec qua ảnh ghim theo digest.

| Tệp | Nội dung |
| :--- | :--- |
| `00` đến `09` | Policy as Code trên nhánh `patched` và trên các cặp fixture |
| `10` đến `13` | Policy as Code trên nhánh `main`, trạng thái có khiếm khuyết |
| `20`, `21` | Test-Driven Security trên `patched` và trên `main` |
| `22`, `23` | Compliance as Code, InSpec trên container đang chạy của hai nhánh |
| `30` | Chỉ số SonarQube của hai nhánh, lấy qua API sau khi quét |
| `34` đến `37` | CD và chuỗi cung ứng, trích từ artifact của run `41889988744` |

Tóm tắt kết quả đối chứng:

| Lớp kiểm | `main` | `patched` |
| :--- | :--- | :--- |
| Conftest, bốn tệp cấu hình | 26 vi phạm | 0 |
| SonarQube | quality gate ERROR, 0 lỗ hổng, hạng A | quality gate OK, 0 lỗ hổng, hạng A |
| Semgrep | 64 phát hiện mức ERROR | 50 |
| Test-Driven Security | 5 phép kiểm, 0 đạt | 6 phép kiểm, 6 đạt |
| InSpec | 7 control, 3 đạt | 7 control, 7 đạt |

Bẫy đã gặp và đã xử lý trong lượt đo này: tệp `docker inspect` do host sinh ra
nếu có dấu BOM thì cả bảy control InSpec báo `undefined method 'dig' for nil` và
hiện ra như bảy control trượt, trong khi thực tế không phép kiểm nào chạy. Đó là
hỏng ở đường lấy dữ liệu chứ không phải một vi phạm tuân thủ.
