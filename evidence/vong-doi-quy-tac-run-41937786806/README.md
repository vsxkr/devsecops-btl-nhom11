# Vòng đời quy tắc, lượt chạy thật ngày 05/11/2026

Run `41937786806` trên nhánh `patched`, kết quả success.

Trivy quét hai ảnh nền và đối chiếu với mốc cơ sở `sbom/cve-baseline.json`:
`php:8.2-apache` có 95 lỗ hổng mức CRITICAL và HIGH, toàn bộ là mới so với mốc;
`mariadb:10.11` có 26, toàn bộ là mới. Xem `cve-watch.txt`.

`ci/cve_watch.py` cập nhật mốc cơ sở rồi mở pull request `#1` vào nhánh
`patched`, tiêu đề `sec: co CVE moi tren anh nen, can danh gia va nang phien ban`,
nội dung ở `auto-remediate-pr-body.md`.

Điểm cần ghi lại: pull request đó **đi qua đúng chuỗi cổng của mọi thay đổi khác
và bị chặn**, ở run `41938004357`. Cổng chặn là stage 3, cổng CVE, tức đúng cổng
phụ trách nội dung mà pull request đang đề xuất. Stage 5 bị bỏ qua và job tổng
kết trả mã khác 0, nên pull request không hợp nhất được. Tự động hóa phát hiện và
soạn thay đổi; nó không thay thế việc phê duyệt.

Hai điều kiện phải có để vòng lặp hoạt động, cả hai đều từng thiếu và đã sửa:

1. `auto-remediate.yml` phải nằm trên NHÁNH MẶC ĐỊNH. GitHub chỉ đăng ký workflow
   và chỉ chạy lịch từ nhánh mặc định; trước đó tệp chỉ có trên `patched` nên lịch
   hằng tuần không bao giờ kích hoạt.
2. Trigger `pull_request` của `devsecops-gate.yml` phải liệt kê nhánh `patched`.
   Thiếu nó thì pull request do vòng lặp sinh ra không kích hoạt cổng nào, đúng
   điều mà CS-07 cấm.
