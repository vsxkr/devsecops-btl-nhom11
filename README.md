# Bài tập lớn DevSecOps, Nhóm 11, M26CQAT01-B

Đánh giá an toàn ứng dụng **InlaneWeb Logistics Portal** theo yêu cầu bảo vệ dữ
liệu cá nhân, bằng một pipeline năm stage có cổng chặn thật, mọi cổng viết
thành mã và chứng minh hai chiều, và mọi cổng truy được về một điều khoản chính
sách của doanh nghiệp.

Giảng viên: TS. Phạm Hoàng Duy. Trưởng nhóm nộp bài: Nguyễn Giang Nam, B26CHAT030.
Hạn nộp 07/11/2026.

## Đọc nhanh cho người chấm

| | |
|---|---|
| Nhánh `main` | trạng thái **có khiếm khuyết**, là đối tượng được đánh giá. Pipeline phải đỏ ở đây |
| Nhánh `patched` | bản vá. Pipeline phải xanh ở đây |
| Run xanh tham chiếu | [`41934351502`](https://github.com/vsxkr/devsecops-btl-nhom11/actions/runs/41934351502) trên `patched`, 7 trên 7 job. Mọi run sau đó trên `patched` cũng phải xanh, xem [tab Actions](https://github.com/vsxkr/devsecops-btl-nhom11/actions) |
| Run có đủ artifact minh chứng | [`41889988744`](https://github.com/vsxkr/devsecops-btl-nhom11/actions/runs/41889988744), 5 artifact `evidence-*` |
| Run đỏ của baseline | [`41889869182`](https://github.com/vsxkr/devsecops-btl-nhom11/actions/runs/41889869182) trên `main`, bị chặn ở stage 1 và stage 3 |
| Chứng minh hai chiều của cổng quản trị | run [`41889870707`](https://github.com/vsxkr/devsecops-btl-nhom11/actions/runs/41889870707): lần 1 đỏ, lần 2 xanh, **cùng commit** `fac7dad`, khác biệt duy nhất là bảo vệ nhánh phía GitHub |
| Báo cáo nộp thầy | không nằm trong kho này, giữ ở máy cục bộ theo yêu cầu |

Kết quả cốt lõi: SonarQube Community cho **0 lỗ hổng và xếp hạng an toàn mức A trên
cả hai nhánh**, kể cả nhánh có khiếm khuyết, trong khi Semgrep bắt 64 phát hiện mức
ERROR trên `main` và 50 trên `patched`. Một quality gate đạt của SonarQube Community
là bằng chứng về chất lượng mã, không phải bằng chứng về an toàn.

## Cấu trúc

```
app/InlaneWeb/        ung dung duoc danh gia, giu nguyen khiem khuyet lam doi tuong
policy/              Chinh sach va Policy as Code
  00-chinh-sach-attt.md       chinh sach ATTT doanh nghiep, CS-01 den CS-08
  exceptions.json             ngoai le co thoi han, theo CS-08
  docker_security.rego        + _test.rego    lop build
  compose_security.rego       + _test.rego    lop trien khai
  repo_policy.rego            + _test.rego    lop quan tri
ci/                  Security as Code
  semgrep-policy-php.yml      8 luat P1 den P5 cho PHP
  semgrep-local-rules.yml     2 luat bi mat gan cung
  sonarqube.py, policy_guard.py, semgrep_report.py, validate_configs.py
  check_branch_payload.py     kiem cau truc phan hoi API truoc khi danh gia
  check_exceptions.py         thi hanh CS-08, ngoai le het han lam stage 1 do
  cve_watch.py                theo doi CVE moi sau khi phat hanh
tests/security/      Test-Driven Security, 6 phep kiem hanh vi
compliance/          Compliance as Code, 7 control InSpec tren container dang chay
fixtures/            cap doi chung: ban co y sai va ban da sua, cho tung lop
patches/             ban va, gom phan chua telemetry
evidence/            output da luu cua tung lenh kiem chung, co ghi ma thoat
sbom/                SBOM CycloneDX cua hai anh nen
.github/workflows/
  devsecops-gate.yml          pipeline nam stage, CO CHAN, co stage CD
  auto-remediate.yml          vong doi quy tac, tu dong mo pull request va CVE
  security-baseline.yml       ghi nhan trang thai, KHONG chan
docs/demo/           huong dan demo cho tung nguoi (KHONG nop cho thay)
report/              bao cao nop thay
```

## Năm lớp kiểm soát, năm đối tượng khác nhau

| Cổng | Công cụ | Lớp kiểm | Bắt được loại gì |
|---|---|---|---|
| Policy as Code | Conftest 0.71.1, OPA 1.21.1 | cấu hình trước khi triển khai | thiếu `USER`, bí mật trong compose, `docker.sock` ghi được, cổng mở ra mọi giao diện, nhánh không được bảo vệ |
| Security as Code | SonarQube Community, Semgrep 1.182.1 | mã nguồn tĩnh | chèn mã truy vấn, bí mật gán cứng, định danh từ client |
| Test-Driven Security | PHP, socket thô, mysqli | hành vi khi chạy | quyền được xoá không được thực thi, bản xuất lộ thông tin xác thực, phân quyền ngang |
| Compliance as Code | InSpec qua `docker inspect` | container đang chạy | khoảng lệch giữa cấu hình khai báo và thực tế |
| Chuỗi cung ứng | Trivy 0.70.0 | ảnh container | CVE của ảnh nền, SBOM |
| CD, phát hành có xác minh | docker compose, InSpec | môi trường đích sau khi phát hành | sai lệch phát sinh trong chính quá trình phát hành |

## Trạng thái đo được của hệ thống

| Lớp kiểm | Công cụ | `main` (`37fe7ff`) | `patched` (`5040924`) |
|---|---|---|---|
| Policy as Code, build | Conftest | 8 vi phạm trên hai Dockerfile | 0 |
| Policy as Code, triển khai | Conftest | 18 vi phạm trên hai tệp compose | 0 |
| Policy as Code, quản trị | Conftest | 2 vi phạm | 0 |
| Security as Code | SonarQube | quality gate ERROR, 0 lỗ hổng, 10 bug, 15 code smell | quality gate OK, 0 lỗ hổng, 0 bug, 19 code smell |
| Security as Code | Semgrep | 64 phát hiện mức ERROR | 50 |
| Test-Driven Security | PHP, socket thô | 5 phép kiểm, 0 đạt | 6 phép kiểm, 6 đạt |
| Compliance as Code | InSpec | 7 control, 3 đạt | 7 control, 7 đạt |
| Chuỗi cung ứng | Trivy | 3 CVE mức CRITICAL trên hai ảnh nền | 0 sau miễn trừ có thời hạn |

`conftest verify` hiện 38 test, 38 đạt. Toàn bộ số liệu trên có tệp bằng chứng
tương ứng trong `evidence/`, mỗi tệp ghi lệnh đã chạy, kết quả, và mã thoát.

## Chứng minh hai chiều

| Lớp | Bản cố ý sai | Bản đã sửa |
|---|---|---|
| Dockerfile | 9 lỗi, exit 1 | 0 lỗi, 1 cảnh báo, exit 0 |
| Compose | 14 lỗi, exit 1 | 0 lỗi, 0 cảnh báo, exit 0 |
| Quản trị | 1 lỗi, exit 1 | 0 lỗi, exit 0 |
| Mã PHP | 8 phát hiện, exit 1 | 0 phát hiện, exit 0 |

## Ba cái bẫy mọi người phải biết

**Thiếu `--namespace`** thì Conftest in `0 tests, 0 failures` và trả mã thoát 0, tức
cổng xanh trong khi không đánh giá gì. Xem `evidence/05`.

**Thiếu `--error` cho Semgrep hoặc `--exit-code 1` cho Trivy** thì công cụ chỉ báo
cáo: 36 phát hiện vẫn exit 0, 119 CVE vẫn exit 0. Xem `evidence/17` và `evidence/23`.

**Thiếu `shell: bash` cho một bước có dấu pipe** thì mã thoát của cả chuỗi lệnh là
mã thoát của `tee` và luôn bằng 0. Ba cổng stage 1 và hai cổng stage 4 từng báo
thành công trong khi tệp kết quả của chính chúng chứa dòng FAIL. Đo được ở run
`41889869182`: job Policy as Code báo success trong khi ba tệp kết quả của chính
nó chứa tổng cộng 41 dòng FAIL và không tệp nào có dòng `exit=`. Cùng ba tệp đó
ở run `41889988744` có 0 dòng FAIL.

Pipeline `devsecops-gate.yml` có một bước kiểm riêng để bảo đảm không cổng nào
chạy ở trạng thái 0 test.

## Chạy toàn bộ

```
conftest verify -p policy/
conftest test app/InlaneWeb/web/Dockerfile -p policy/ --namespace dockerfile
conftest test app/InlaneWeb/docker-compose.prod.yml -p policy/ --namespace compose
conftest test evidence/api/branch-main-slim.json -p policy/ --namespace repo
semgrep --metrics off --error --config ci/semgrep-policy-php.yml app/InlaneWeb/web/src
trivy image --severity HIGH,CRITICAL --exit-code 1 php:8.2-apache
bash tests/security/run.sh
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v "$PWD/compliance:/profile" chef/inspec exec /profile \
  --input container=ilfreight_web --chef-license=accept-silent
```

Trên Windows, đặt `PYTHONIOENCODING=utf-8` trước mỗi lệnh Semgrep.

## Căn cứ tài liệu

Yêu cầu chính thức của thầy, hạn 07/11. Phương pháp lấy từ ba lab
trong đường dẫn môn học: `Policy_as_code_Lab ConfTest.docx`,
`SonarQube and juice shop Lab.docx`, `Juice_Shop_Telemetry_Lab_Guide.docx`, cùng
`Hands-on_guide_updated.docx` với 7 phase và 5 mốc chấm. Khung phân tích lấy từ
`3.Threat Modeling and Risk Management in DevSecOps Architecture.docx`.
