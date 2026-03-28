## Lộ trình học Terraform từ zero đến nâng cao trên AWS

Repo này ghi lại quá trình tôi **học Terraform từ con số 0**, triển khai hạ tầng trên **AWS** theo tài liệu chính thức của HashiCorp và áp dụng **mindset của DevOps Lead**.

- Nguồn học chính: [Create infrastructure – Terraform AWS tutorial](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-create)

Mục tiêu:
- **Hiểu khái niệm Infrastructure as Code (IaC)** và cách Terraform mô tả hạ tầng bằng code.
- **Tự tay tạo một EC2 instance trên AWS** bằng Terraform.
- **Áp dụng best practices từ đầu**: maintainability, security, và scalability.
- Rèn luyện thói quen **làm việc có cấu trúc, có commit rõ ràng** cho từng bước học.

> **Triết lý học**: Code chạy đúng là chưa đủ. Code phải dễ maintain, dễ scale, và an toàn.

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

## Ý tưởng tổ chức commit (hành trình học có cấu trúc)

Tổ chức commit rõ ràng giúp theo dõi hành trình học và dễ dàng quay lại từng giai đoạn:

**Phase 1: Foundation (Zero to Running)**
- `chore: init terraform learning repo` – Tạo project, README mô tả mục tiêu.
- `feat: add terraform and aws provider config` – Thêm `terraform.tf` với version pinning.
- `feat: define aws ec2 instance in main.tf` – Tạo provider, data source, resource đầu tiên.
- `chore: format terraform files` – Chạy `terraform fmt`.
- `feat: run terraform init and apply to create ec2` – Ghi lại kết quả EC2 tạo thành công.
- `docs: document terraform state and destroy flow` – Giải thích state management.

**Phase 2: Refactor for Maintainability**
- `refactor: replace ternary with locals map` – Áp dụng map thay conditional logic.
- `refactor: minimize variables, move to locals` – Chỉ giữ `var.env` và `var.project`.
- `feat: add dynamic ami and az lookup` – Thay hardcode bằng data sources.
- `refactor: switch from count to for_each` – Tránh index-based resource management.
- `feat: add lifecycle rules for critical resources` – Thêm `prevent_destroy`, `create_before_destroy`.
- `feat: implement naming convention` – Áp dụng `{project}-{env}-{region}-{resource}`.
- `feat: add default tags via provider` – Tự động tag cho tất cả resources.

**Phase 3: Structure & Modules**
- `refactor: split into per-directory environments` – Tách `dev/` và `prod/`.
- `feat: create networking module` – Extract VPC, subnet logic vào module.
- `feat: create compute module` – Extract EC2, ASG logic vào module.
- `feat: setup remote backend with s3 and dynamodb` – State locking cho team.
- `feat: add backend config per environment` – Mỗi env có S3 bucket riêng.

**Phase 4: Operations & Security**
- `ci: add github actions terraform workflow` – Format check, validate, plan.
- `ci: add checkov security scanning` – Scan lỗ hổng bảo mật.
- `ci: add infracost estimation` – Ước tính chi phí trên PR.
- `ci: add manual approval for production` – Gate cho production apply.
- `docs: add pr review checklist` – Hướng dẫn review Terraform PR.
- `chore: add pre-commit hooks` – Format và validate trước khi commit.

Nhờ cách commit theo từng bước này:
- Dễ dàng **quay lại từng giai đoạn học**.
- So sánh sự khác biệt cấu hình theo thời gian.
- Sử dụng repo như một **nhật ký học Terraform** từ zero đến lead level.

---

## Kết luận

Repo này là hành trình học Terraform với **mindset của một DevOps Lead** ngay từ đầu:

- **Giai đoạn 1**: Học Terraform từ nền tảng cơ bản nhất theo tài liệu chính thức HashiCorp.
- **Giai đoạn 2-3**: Áp dụng best practices về maintainability, structure, và security.
- **Giai đoạn 4-5**: Thiết kế quy trình vận hành cho team, review code như một Lead.

**3 câu hỏi định hướng mỗi khi viết Terraform**:
1. 6 tháng sau người mới vào đọc code này có hiểu không?
2. Thêm một environment mới có phải sửa nhiều chỗ không?
3. Thay đổi này có thể phá gì và tốn bao nhiêu?

**Sự khác biệt giữa các level**:
- **Member**: Viết code chạy được.
- **Senior**: Viết code dễ maintain.
- **Lead**: Thiết kế hệ thống dễ scale, an toàn, và review được code của người khác.

Trong tương lai, repo này sẽ mở rộng:
- Tách thành per-directory environments.
- Tạo modules tái sử dụng.
- Setup remote backend và state locking.
- Tích hợp CI/CD pipeline với security scanning.
- Áp dụng Terragrunt cho DRY configuration.

**Nguồn kiến thức**:
- [HashiCorp Terraform Tutorials](https://developer.hashicorp.com/terraform/tutorials)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [AWS Tagging Best Practices](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html)
- Real-world insights từ Slack Engineering, Reddit r/Terraform, r/devops

---

**Infrastructure as Code không chỉ là code. Đó là cách bạn tư duy về hạ tầng.**

---

## Lộ trình tổng quan (Zero → Lead Level)

### Giai đoạn 1 – Nhập môn / Cơ bản
**Mục tiêu**: Hiểu Terraform làm gì và chạy được lệnh cơ bản.

- Cài Terraform CLI (`>= 1.2`), AWS CLI.
- Viết file đầu tiên: `terraform.tf`, `main.tf`.
- Hiểu 3 khái niệm: **provider**, **data source**, **resource**.
- Thực hành workflow: `init` → `fmt` → `validate` → `plan` → `apply` → `destroy`.
- Tạo được một EC2 instance trên AWS.

**Mindset giai đoạn này**: Làm cho chạy được. Không sợ sai.

---

### Giai đoạn 2 – Trung cấp (Tư duy Maintainability)
**Mục tiêu**: Code không chỉ chạy, mà còn dễ đọc, dễ sửa.

**Tổ chức file chuẩn**:
- `terraform.tf` – khai báo provider version, Terraform version.
- `main.tf` – định nghĩa resource chính.
- `variables.tf` – input variables (càng ít càng tốt).
- `locals.tf` – computed values, maps, logic tính toán.
- `outputs.tf` – output values.
- `data.tf` – data sources (AMI, AZ, VPC, …).

**Nguyên tắc vàng**:

1. **Map thay ternary**
   ```hcl
   # ❌ Tránh: logic rải rác
   instance_type = var.env == "prod" ? "m5.xlarge" : "t3.micro"
   
   # ✅ Tốt: data tập trung
   locals {
     instance_types = {
       dev     = "t3.micro"
       staging = "t3.small"
       prod    = "m5.xlarge"
     }
   }
   instance_type = local.instance_types[var.env]
   ```

2. **Variable càng ít càng tốt**
   - Lý tưởng: chỉ có `var.env` và `var.project`.
   - Mọi thứ khác (region, instance_type, CIDR, replica count) nằm trong `locals`, tra theo `env`.
   - Variable chỉ dành cho thứ **không suy ra được từ env**.

3. **Không hardcode AZ và AMI**
   ```hcl
   # ❌ Tránh: AZ mapping khác nhau giữa các AWS account
   availability_zone = "us-east-1a"
   
   # ✅ Tốt: dynamic lookup
   data "aws_availability_zones" "available" {
     state = "available"
   }
   availability_zone = data.aws_availability_zones.available.names[0]
   ```

4. **for_each thay count**
   - `count` dùng index → xóa phần tử giữa sẽ làm Terraform xóa nhầm resource.
   - `for_each` dùng key → xóa key nào thì chỉ resource đó bị xóa.

5. **Lifecycle rules cho resource quan trọng**
   ```hcl
   resource "aws_db_instance" "main" {
     # ...
     lifecycle {
       prevent_destroy       = true  # Không cho phép xóa
       create_before_destroy = true  # Zero-downtime replacement
       ignore_changes        = [tags["LastModified"]]
     }
   }
   ```

6. **Sensitive variables**
   ```hcl
   variable "db_password" {
     type      = string
     sensitive = true  # Không hiện trong plan output
   }
   ```

**Thực hành giai đoạn này**:
- Refactor code hiện tại theo cấu trúc trên.
- Thêm VPC, subnet, security group, RDS (tùy nhu cầu).
- Áp dụng naming convention và tagging.

---

### Giai đoạn 3 – Nâng cao (Tư duy Lead)
**Mục tiêu**: Thiết kế hạ tầng như một hệ thống, không phải một file.

**1. Naming Convention: Đặt tên như đặt tên đường**

Cú pháp: `{project}-{env}-{region_short}-{resource}-{descriptor}`

Ví dụ:
- `acme-prod-apse1-alb-public`
- `acme-dev-use1-rds-primary`
- `acme-staging-euw1-s3-logs`

```hcl
locals {
  region_short = {
    "us-east-1"      = "use1"
    "us-west-2"      = "usw2"
    "ap-southeast-1" = "apse1"
    "eu-west-1"      = "euw1"
  }
  name_prefix = "${var.project}-${var.env}-${local.region_short[var.region]}"
}

resource "aws_instance" "app" {
  # ...
  tags = {
    Name = "${local.name_prefix}-ec2-app"
  }
}
```

**2. Default Tags: Đồng bộ tags tự động**

```hcl
provider "aws" {
  region = var.region
  
  default_tags {
    tags = {
      Environment = var.env
      Project     = var.project
      ManagedBy   = "Terraform"
      Owner       = var.team_name
    }
  }
}
```

Nguồn: [AWS Tagging Best Practices](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html)

**3. Cấu trúc dự án: Per-Directory Environments**

⚠️ **TRÁNH XA WORKSPACE**: Workspace dễ gõ nhầm, apply sai env, và không hỗ trợ backend riêng cho mỗi env.

✅ **Dùng per-directory**:

```
infrastructure/
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── compute/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── environments/
    ├── dev/
    │   ├── main.tf
    │   ├── backend.tf      # S3 bucket riêng
    │   ├── terraform.tfvars
    │   └── locals.tf
    └── prod/
        ├── main.tf
        ├── backend.tf      # S3 bucket riêng
        ├── terraform.tfvars
        └── locals.tf
```

**Lợi ích**:
- Đang ở thư mục `dev` thì chỉ apply được `dev`. Không thể apply nhầm `prod`.
- Mỗi env có backend riêng, state riêng, IAM riêng.
- Code sạch, khác biệt nằm trong `tfvars` và `locals`.
- PR chỉ thay đổi trong thư mục `dev` → reviewer biết ngay scope.

**4. Module: Tái sử dụng và chuẩn hóa**

```hcl
# environments/dev/main.tf
module "networking" {
  source = "../../modules/networking"
  
  env          = var.env
  project      = var.project
  vpc_cidr     = local.vpc_cidrs[var.env]
  azs          = data.aws_availability_zones.available.names
}

module "compute" {
  source = "../../modules/compute"
  
  env             = var.env
  project         = var.project
  instance_type   = local.instance_types[var.env]
  subnet_id       = module.networking.private_subnet_ids[0]
  security_groups = [module.networking.app_sg_id]
}
```

**5. Remote Backend với State Locking**

```hcl
# environments/dev/backend.tf
terraform {
  backend "s3" {
    bucket         = "acme-terraform-state-dev"
    key            = "dev/terraform.tfstate"
    region         = "ap-southeast-1"
    encrypt        = true
    dynamodb_table = "acme-terraform-locks-dev"
  }
}
```

**Lợi ích**:
- State được lưu trên S3, không mất khi máy local hỏng.
- DynamoDB lock ngăn 2 người apply cùng lúc.
- Encrypt state vì có thể chứa thông tin nhạy cảm.

---

### Giai đoạn 4 – Lead Level (Tư duy Vận hành & Bảo mật)
**Mục tiêu**: Thiết kế quy trình cho team, đảm bảo an toàn và kiểm soát.

**1. Version Pinning & Security Monitoring**

```hcl
# terraform.tf
terraform {
  required_version = "~> 1.5"  # Ghim major version
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"  # Cho phép patch updates
    }
  }
}
```

**Theo dõi security advisory**:
- [HashiCorp Security](https://www.hashicorp.com/security)
- [AWS Provider Changelog](https://github.com/hashicorp/terraform-provider-aws/releases)
- [GitHub Advisory Database](https://github.com/advisories)
- [CVE Database](https://cve.mitre.org/)

Ví dụ: CVE-2024-6257 (go-getter command injection) – cần update Terraform để tránh bị tấn công qua Git URL.

**2. CI/CD Pipeline (Không thỏa hiệp)**

Dù team 3 người hay 30 người, pipeline phải có:

```yaml
# .github/workflows/terraform.yml (ví dụ)
name: Terraform CI/CD

on:
  pull_request:
    paths:
      - 'infrastructure/**'
  push:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # 1. Format check (không thỏa hiệp)
      - name: Terraform Format Check
        run: terraform fmt -check -recursive
      
      # 2. Security scan (bắt lỗ hổng sớm)
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: infrastructure/
          framework: terraform
      
      # 3. Plan và đăng lên PR
      - name: Terraform Plan
        run: |
          cd infrastructure/environments/dev
          terraform init
          terraform plan -out=tfplan
      
      - name: Post Plan to PR
        uses: actions/github-script@v7
        with:
          script: |
            // Đăng plan output lên PR comment
      
      # 4. Cost estimation
      - name: Infracost
        uses: infracost/actions/comment@v1
        with:
          path: infrastructure/environments/dev
  
  apply:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production  # Manual approval required
    steps:
      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
```

**Các điểm không thỏa hiệp**:
1. `terraform fmt -check` trên mỗi PR → không format đúng thì không merge.
2. `terraform plan` output đăng trên PR → cả team thấy resource nào tạo/sửa/xóa.
3. Security scan (Checkov/Trivy) → lỗ hổng bảo mật bắt sớm.
4. **Manual approve cho production** → người có quyền review plan rồi bấm Approve.
5. State luôn trên remote backend có lock → không bao giờ local state khi làm team.
6. Infracost ước tính chi phí → biết trước khi tạo 5 NAT Gateway tốn bao nhiêu.

**3. Cấu trúc Multi-Region, Multi-Service (Hierarchical)**

Khi dự án lớn, tách theo service và region:

```
infrastructure/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
└── environments/
    ├── dev/
    │   └── ap-southeast-1/
    │       ├── vpc/
    │       ├── app/
    │       └── database/
    └── prod/
        ├── ap-southeast-1/
        │   ├── vpc/
        │   ├── app/
        │   └── database/
        └── us-east-1/
            ├── vpc/
            └── app/
```

**Lợi ích**:
- Mỗi service, mỗi region, mỗi env = một state riêng.
- **Blast radius nhỏ nhất**: lỗi ở VPC dev không ảnh hưởng database prod.
- Dùng Terragrunt để DRY (Don't Repeat Yourself) config.

**4. Tách theo Service (Cách các công ty lớn làm)**

Slack chia sẻ họ tách state theo service, không gom chung ([Slack Engineering Blog](https://slack.engineering/how-we-use-terraform-at-slack/)).

Một team trên Reddit r/devops quản lý **1.500 repo Terraform riêng biệt**, mỗi service một repo, shared GitHub Actions workflow.

**Nguyên tắc**: Tách nhỏ, giảm blast radius, mỗi team tự quản lý hạ tầng của service mình.

---

### Giai đoạn 5 – Mastery (Review như một Lead)
**Mục tiêu**: Review code của người khác, đảm bảo chất lượng team.

**Lead review gì khi member mở PR?**

Cú pháp thì AI đã lo. Logic thì coworker detect được. Lead xem:

1. **Tính maintain**: 6 tháng sau người mới vào đọc có hiểu không?
2. **Tính scale**: Thêm env mới có phải sửa 15 chỗ không?
3. **Security**: Có lỗ hổng không? Sensitive data có được bảo vệ không?
4. **Cost**: Resource này tốn bao nhiêu? Có cách tối ưu không?
5. **Blast radius**: Thay đổi này ảnh hưởng bao nhiêu resource?

**Checklist review Terraform PR**:

- [ ] Code đã format (`terraform fmt`)?
- [ ] Có dùng ternary lồng nhau không? → Đề xuất map.
- [ ] Variable có quá nhiều không? → Đề xuất chuyển sang locals.
- [ ] Có hardcode AZ, AMI, region không? → Đề xuất data source.
- [ ] Dùng `count` hay `for_each`? → Ưu tiên `for_each`.
- [ ] Resource quan trọng có `prevent_destroy` không?
- [ ] Sensitive variable có đánh dấu `sensitive = true` không?
- [ ] Naming convention có đúng không? (`{project}-{env}-{region}-{resource}`).
- [ ] Tags có đầy đủ không? (Environment, Project, ManagedBy, Owner).
- [ ] Plan output có resource nào bị xóa không? → Xác nhận kỹ.
- [ ] Infracost estimate có hợp lý không?

**Công cụ Lead cần biết**:
- **Checkov**: Security và compliance scanning ([checkov.io](https://www.checkov.io/))
- **Trivy**: Vulnerability scanning ([aquasecurity/trivy](https://github.com/aquasecurity/trivy))
- **Infracost**: Cost estimation ([infracost.io](https://www.infracost.io/))
- **Terragrunt**: DRY configuration ([terragrunt.gruntwork.io](https://terragrunt.gruntwork.io/))
- **TFLint**: Linting và best practices ([github.com/terraform-linters/tflint](https://github.com/terraform-linters/tflint))

---

## Best Practices Tổng hợp

### Maintainability
- Logic càng ít càng tốt (ternary, count, conditional).
- Data càng nhiều càng tốt (map, locals, data source).
- Logic ít → ít chỗ sai. Data nhiều → dễ đọc, dễ thêm, dễ sửa.

### Structure
- **Workspace**: Tránh xa. Dễ gõ nhầm, apply sai env.
- **Per-directory**: Ưu tiên số 1. Mỗi env một thư mục riêng.
- **Hierarchical**: Mở rộng multi-region, multi-service.
- **Service-based**: Tách repo theo service (microservices).

### Security
- Ghim version provider và Terraform CLI.
- Theo dõi security advisory và CVE.
- Scan tự động trong pipeline (Checkov, Trivy).
- Sensitive variables phải đánh dấu `sensitive = true`.
- State encryption và access control.

### Operations
- `terraform fmt` check trên mỗi PR.
- `terraform plan` output đăng trên PR.
- Manual approve cho production.
- Remote backend có lock (S3 + DynamoDB).
- Cost estimation (Infracost) trên PR.

---

## Tài nguyên học thêm

**Official Documentation**:
- [Terraform AWS Tutorial](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)

**Community & Real-world Examples**:
- [r/Terraform](https://www.reddit.com/r/Terraform/)
- [r/devops](https://www.reddit.com/r/devops/)
- [Slack Engineering: How We Use Terraform](https://slack.engineering/how-we-use-terraform-at-slack/)
- [Gruntwork Blog](https://blog.gruntwork.io/)

**Security**:
- [HashiCorp Security Advisories](https://www.hashicorp.com/security)
- [AWS Provider Security](https://github.com/hashicorp/terraform-provider-aws/security)
- [Checkov Documentation](https://www.checkov.io/)

**Tools**:
- [Terragrunt](https://terragrunt.gruntwork.io/) – DRY Terraform
- [Infracost](https://www.infracost.io/) – Cost estimation
- [TFLint](https://github.com/terraform-linters/tflint) – Linter
- [Terraform Weekly Newsletter](https://www.terraformweekly.com/)

---

## Kết luận

Repo này không chỉ là nơi học Terraform, mà còn là nơi học **tư duy của một DevOps Lead**:

- **Member**: Viết code chạy được.
- **Senior**: Viết code dễ maintain.
- **Lead**: Thiết kế hệ thống dễ scale, an toàn, và review được code của người khác.

Hành trình từ zero đến lead không phải về số lượng tool biết, mà về **mindset**: code không chỉ chạy đúng, mà phải sống được lâu dài trong tay nhiều người.

