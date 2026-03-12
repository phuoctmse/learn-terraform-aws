## Lộ trình học Terraform từ zero đến nâng cao trên AWS

Repo này ghi lại quá trình tôi **học Terraform từ con số 0**, triển khai hạ tầng trên **AWS** theo tài liệu chính thức của HashiCorp:

- Nguồn học chính: [Create infrastructure – Terraform AWS tutorial](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create)

Mục tiêu:
- **Hiểu khái niệm Infrastructure as Code (IaC)** và cách Terraform mô tả hạ tầng bằng code.
- **Tự tay tạo một EC2 instance trên AWS** bằng Terraform.
- Rèn luyện thói quen **làm việc có cấu trúc, có commit rõ ràng** cho từng bước học.

---

## Công cụ & chuẩn bị

- **Terraform CLI**: phiên bản `>= 1.2`.
- **AWS CLI**: đã cài và cấu hình.
- **Tài khoản AWS**: có quyền tạo EC2, VPC, Security Group (ưu tiên dùng free tier).
- Đặt region mặc định: `us-west-2` (theo tutorial, có thể thay bằng region khác nếu cần).

Trước khi chạy Terraform:
- Cấu hình AWS credentials (qua `aws configure` hoặc biến môi trường như `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`).

---

## Cấu trúc Terraform ban đầu

Tôi làm theo hướng dẫn trong tutorial để tạo cấu trúc tối thiểu:

- **`terraform.tf`**  
  Khai báo:
  - **Provider AWS** và version:
    - `source = "hashicorp/aws"`
    - `version = "~> 5.92"` (cho phép các bản 5.x từ 5.92 trở lên).
  - **required_version** cho Terraform CLI: `>= 1.2`.

- **`main.tf`**  
  Mô tả hạ tầng:
  - `provider "aws"` – chọn region (ví dụ: `us-west-2`).
  - `data "aws_ami" "ubuntu"` – lấy AMI Ubuntu mới nhất (Ubuntu 24.04, owner Canonical).
  - `resource "aws_instance" "app_server"` – tạo EC2 instance loại `t2.micro` (free tier), gắn tag `Name = "learn-terraform"`.

Ý nghĩa:
- **`data`**: chỉ đọc thông tin từ AWS (không tạo tài nguyên).
- **`resource`**: thực sự tạo/ quản lý tài nguyên trên AWS.
- Các block tham chiếu nhau bằng địa chỉ dạng:  
  - `data.aws_ami.ubuntu.id`  
  - `aws_instance.app_server`

---

## Quy trình làm việc với Terraform

Theo tutorial HashiCorp, tôi thực hành quy trình chuẩn sau:

1. **Viết/ cập nhật cấu hình**
   - Chỉnh sửa `terraform.tf`, `main.tf`.

2. **Định dạng & kiểm tra**
   - `terraform fmt` – format code HCL.
   - `terraform validate` – kiểm tra cú pháp và tính hợp lệ của cấu hình.

3. **Khởi tạo workspace**
   - `terraform init`  
   - Tải provider `hashicorp/aws`, tạo `.terraform/` và `.terraform.lock.hcl`.

4. **Lên kế hoạch thay đổi**
   - `terraform plan`  
   - Xem trước Terraform sẽ **tạo / sửa / xóa** gì (ví dụ: 1 resource `aws_instance.app_server` được tạo).

5. **Áp dụng thay đổi (tạo hạ tầng)**
   - `terraform apply`  
   - Xem lại plan, nhập `yes` để Terraform tạo EC2 instance trên AWS.
   - Kiểm tra trong AWS Console → EC2 để xác nhận instance đã được tạo.

6. **Quản lý state**
   - Terraform tạo file `terraform.tfstate` để lưu **trạng thái thực tế** của hạ tầng.
   - Một số lệnh hữu ích:
     - `terraform state list` – liệt kê các resource/data đang được quản lý.
     - `terraform show` – xem chi tiết state.
   - Ghi chú:
     - State có thể chứa thông tin nhạy cảm → cần được bảo vệ.
     - Sau này có thể chuyển sang **remote state** (HCP Terraform, S3, …) để làm việc nhóm.

7. **Hủy hạ tầng khi không dùng nữa**
   - `terraform destroy`  
   - Dùng khi chỉ học/lab để tránh tốn chi phí do EC2 chạy liên tục.

---

## Ý tưởng tổ chức commit (hành trình học)

Mặc dù repo hiện tại chưa là git repo, nhưng tôi định hướng tổ chức commit (nếu dùng git) như sau để thể hiện rõ hành trình học từ zero:

- **`chore: init terraform learning repo`**
  - Tạo thư mục project, thêm `README.md` mô tả mục tiêu học.

- **`feat: add terraform and aws provider config`**
  - Thêm `terraform.tf` với block `terraform {}` và `required_providers`.

- **`feat: define aws ec2 instance in main.tf`**
  - Thêm `provider "aws"`, `data "aws_ami" "ubuntu"`, `resource "aws_instance" "app_server"`.

- **`chore: format terraform files`**
  - Chạy `terraform fmt`, cập nhật format chuẩn.

- **`feat: run terraform init and apply to create ec2`**
  - Lưu lại ghi chú/ảnh chụp kết quả EC2 instance tạo thành công.

- **`chore: document terraform state and destroy flow`**
  - Cập nhật `README.md` với phần giải thích `terraform.tfstate`, `terraform state`, `terraform destroy`.

Nhờ cách commit theo từng bước này, tôi có thể:
- Dễ dàng **quay lại từng giai đoạn học**.
- So sánh sự khác biệt cấu hình theo thời gian.
- Sử dụng repo như một **nhật ký học Terraform** cá nhân.

---

## Kết luận

Repo này là nơi tôi:
- Học Terraform từ nền tảng cơ bản nhất.
- Thực hành theo tài liệu chính thức HashiCorp:  
  [Create infrastructure | Terraform | HashiCorp Developer](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create)
- Ghi lại quy trình, lệnh đã dùng và cách tôi hiểu về **Infrastructure as Code** trên AWS.

Trong tương lai, tôi có thể mở rộng:
- Thêm nhiều resource (VPC, Security Group, RDS, S3, …).
- Áp dụng module, remote state, workspace.
- Tích hợp CI/CD cho Terraform.

---

## Lộ trình tổng quan (Zero → Nâng cao)

- **Giai đoạn 1 – Nhập môn / Cơ bản**
  - Làm theo tutorial HashiCorp để:
    - Cài Terraform, AWS CLI.
    - Viết các file cơ bản: `terraform.tf`, `main.tf`.
    - Hiểu provider, data source, resource.
    - Sử dụng các lệnh: `init`, `fmt`, `validate`, `plan`, `apply`, `destroy`.

- **Giai đoạn 2 – Trung cấp**
  - Tổ chức lại cấu trúc file (`variables.tf`, `outputs.tf`, `providers.tf`,…).
  - Dùng `variable`, `output`, `locals`.
  - Quản lý nhiều resource hơn: VPC, subnet, security group, S3, RDS (tùy nhu cầu).

- **Giai đoạn 3 – Nâng cao**
  - Tách và sử dụng **module** (tự viết hoặc dùng từ Registry).
  - Dùng **remote backend** (HCP Terraform hoặc S3) để quản lý state.
  - Dùng **workspace** cho nhiều môi trường (dev/stage/prod).

- **Giai đoạn 4 – Chuyên sâu & Thực chiến**
  - Tích hợp Terraform vào CI/CD (GitHub Actions, GitLab CI, …).
  - Thêm linting / security check (tflint, checkov, … nếu áp dụng).
  - Xây dựng quy trình review `terraform plan` và quản lý thay đổi hạ tầng theo team.

