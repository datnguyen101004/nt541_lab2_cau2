# Triển khai hạ tầng AWS với CloudFormation và AWS CodePipeline

## Mô tả
Dự án này sử dụng **AWS CloudFormation** để triển khai các tài nguyên AWS, bao gồm:
- **VPC**: Cấu hình mạng riêng với các subnet public và private.
- **Route Tables**: Định tuyến lưu lượng mạng.
- **NAT Gateway**: Cho phép các instance trong private subnet truy cập Internet.
- **EC2**: Triển khai máy ảo EC2 trong VPC.
- **Security Groups**: Quy tắc bảo mật để kiểm soát lưu lượng vào/ra.

Dự án tích hợp:
- **AWS CodePipeline**: Tự động hóa quy trình build và deploy từ mã nguồn trên GitHub (thay thế AWS CodeCommit).
- **AWS CodeBuild**: Xây dựng và kiểm tra mã CloudFormation, tích hợp `cfn-lint` và `TaskCat` để kiểm tra tính đúng đắn.

Mục đích: Tạo hạ tầng AWS tự động với quy trình CI/CD, đảm bảo mã CloudFormation tuân thủ các tiêu chuẩn.

## Yêu cầu tiên quyết
- Tài khoản AWS với quyền tạo VPC, EC2, NAT Gateway, CodePipeline, CodeBuild, và S3.
- Cài đặt các công cụ:
  - [AWS CLI](https://aws.amazon.com/cli/) (đã cấu hình credentials).
  - [cfn-lint](https://github.com/aws-cloudformation/cfn-lint) và [TaskCat](https://github.com/aws-quickstart/taskcat) (để kiểm tra local, tùy chọn).
- Tài khoản GitHub và quyền admin để cấu hình kết nối với AWS CodePipeline.

## Hướng dẫn triển khai

### 1. Clone repository
```bash
git clone https://github.com/datnguyen101004/nt541_lab2_cau2.git
cd nt541_lab2_cau2
```

### 2. Cấu hình AWS IAM Role
1. Tạo IAM Role trên AWS với quyền `AdministratorAccess` để CodePipeline và CodeBuild sử dụng.
2. Ghi chú ARN của Role để sử dụng trong pipeline.

### 3. Kiểm tra CloudFormation template (local, tùy chọn)
Kiểm tra tính đúng đắn của file `cloudformation_template.yaml`:
```bash
cfn-lint cloudformation_template.yaml
taskcat -c ci/taskcat.yml
```

### 4. Cấu hình AWS CodePipeline
1. Truy cập AWS Management Console, vào **CodePipeline** và tạo pipeline mới.
2. Cấu hình source stage:
   - Chọn **GitHub** làm source provider.
   - Kết nối với repository `datnguyen101004/nt541_lab2_cau2` (nhánh `main`).
3. Cấu hình build stage:
   - Sử dụng **AWS CodeBuild** để chạy `cfn-lint` và `TaskCat`.
   - Cấu hình file buildspec (ví dụ: `buildspec.yml`) để kiểm tra và triển khai template.
4. Cấu hình deploy stage:
   - Sử dụng **CloudFormation** để triển khai stack từ file `cloudformation_template.yaml`.
5. Tắt **Block Public Access** trên S3 bucket (nếu cần) để lưu trữ artifact.

### 5. Chạy và kiểm tra pipeline
- Push code lên nhánh `main` của repository để kích hoạt pipeline.
- Theo dõi tiến trình trong AWS CodePipeline console.
- Kiểm tra các tài nguyên được tạo trên AWS (VPC, EC2, NAT Gateway, v.v.) sau khi pipeline chạy thành công.

### 6. Hủy tài nguyên
Để tránh chi phí không mong muốn, xóa stack CloudFormation:
```bash
aws cloudformation delete-stack --stack-name <stack-name>
```

## Xử lý sự cố
- **Pipeline thất bại**: Xem log chi tiết trong AWS CodePipeline hoặc CodeBuild.
- **Lỗi CloudFormation**: Kiểm tra cú pháp trong `cloudformation_template.yaml` hoặc quyền IAM.
- **Lỗi S3**: Đảm bảo đã tắt **Block Public Access** trên bucket được sử dụng.