---
title: "Blog 1"

weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Sao lưu tài nguyên Amazon Elastic Kubernetes Service (EKS) bằng NetApp Trident Protect

Kubernetes là một nền tảng điều phối container mã nguồn mở tự động hóa việc triển khai, mở rộng và quản lý ứng dụng được container hóa. Cho phép người dùng chạy ứng dụng và tác vụ trên [Kubernetes](https://kubernetes.io/), bảo vệ tài nguyên và dữ liệu do vô tình xóa hoặc lỗi phần cứng là rất quan trọng để duy trì tính liên tục của hoạt động kinh doanh và đáp ứng các yêu cầu về tuân thủ. Trong khi Kubernetes cung cấp tính sẵn sàng cao thông qua bảng điều khiển và dự phòng worker node, nó vốn không bảo vệ khỏi lỗi do con người, như là vô tình xóa namespaces, deployments, hoặc lưu trữ thực tế (persistent volumes), nó cũng không chống lại các lỗi khu vực (regional) hoặc hỏng dữ liệu.

Sự phức tạp kiến trúc microservices hiện đại và quy mô ngày càng tăng của các triển khai Kubernetes làm quan trọng hóa việc duy trì các bản sao lưu định kỳ và đã được kiểm tra rằng có thể khôi phục một cách nhất quán trong các môi trường khác nhau and Amazon Web Services [(AWS) Regions](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/). Bao gồm sao lưu thành phần thiết yếu như là toàn bộ namespaces, persistent volumes chứa dữ liệu ứng dụng, tài nguyên tùy chỉnh, và đối tượng cấu hình. Nếu không có cơ chế sao lưu thích hợp, các tổ chức có thể đối mặt nguy cơ gián đoạn kéo dài, mất mát dữ liệu và khả năng vi phạm các thỏa thuận mức độ dịch vụ (SLAs). Những điều này có thể dẫn đến tác động tài chính đáng kể và mất niềm tin từ khách hàng. Bài viết này sẽ giới thiệu một công cụ bảo vệ dữ liệu mới và cung cấp giới thiệu từng bước để thực hiện thử nghiệm tính khả thi (proof-of-concept) trong môi trường Kubernetes.

## NetApp Trident Protect

Nó cung cấp khả năng tự động thông qua Kubernetes-native API và CLI *tridentctl-protect* mạnh mẽ, cho phép chương trình truy cập tích hợp liền mạch với luồng công việc hiện có. Người cùng AWS có thể sử dụng Trident Protect để xử lý các hoạt động phục hồi sau thảm họa giữa các cụm, di chuyển các dịch vụ có trạng thái giữa các dịch vụ lưu trữ, hoặc chuyển các tài nguyên đang chạy trên cụm Kubernetes tự quản lý sang dịch vụ [Amazon Elastic Kubernetes Service](https://aws.amazon.com/aws.amazon.com/pm/eks) (Amazon EKS). Kiến trúc sao lưu EKS và Trident được hiển thị trong Ảnh 1.

![Amazon EKS và Trident](/images/3-blogstranslated/3.1-blog1/Amazon-EKS-and-Trident-backup-architecture.png.png)
> *Hình 1. Kiến trúc sao lưu Amazon EKS và Trident.*

[Amazon FSx for NetApp ONTAP](https://aws.amazon.com/fsx/netapp-ontap/) là dịch vụ lưu trữ chia sẻ được quản lý toàn phần, được xây dựng trên hệ thống tập tin ONTAP phổ biến của NetApp. Nó cung cấp cho người dùng quyền truy cập vào các tính năng và dịch vụ lưu trữ cấp doanh nghiệp của ONTAP, chẳng hạn như thin provisioning (cấp phát mỏng), data deduplication (khử trùng lặp dữ liệu), và chức năng Snapshot (bản sao dữ liệu) những chức năng thúc đẩy tính linh hoạt của NetApp trong việc cấp phát lưu trữ và bảo vệ dữ liệu.

Trong bài viết này, chúng tôi sẽ tập trung vào việc sử dụng Trident Protect với ứng dụng mẫu cửa hàng bán lẻ trên AWS của chúng tôi để xử lý các tác vụ bảo vệ và di chuyển dữ liệu đang chạy trên một cụm AKS. Chúng tôi còn đi sâu vào một số tùy chọn sao lưu, khôi phục và di chuyển dữ liệu giúp bạn quyết định triển khai cái gì là tốt nhất cho môi trường tổ chức của mình. Kiến trúc sau đây tạo các bản sao lưu trong AWS Region cục bộ, nơi có thể sử dụng để khôi phục các namespaces khác trong cùng một cụm và di chuyển dữ liệu từ các dịch vụ lưu trữ khác nhau của AWS.

## Kiến trúc phần mềm

[Ứng dụng mẫu cửa hàng bán lẻ đơn giản](https://github.com/aws-containers/retail-store-sample-app) trên AWS được xây dựng từ các microservices và được hỗ trợ bởi một số dịch vụ có trạng thái. Tổng cộng chúng sử dụng bốn [Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PVC) cho mỗi stack như trong Hình 2.

- **Assets**: Dùng để phục vụ các tài nguyên tĩnh như là hình ảnh liên quan đến danh mục sản phẩm-cần volume RWX như NFS.
- **Orders**: Tiếp nhận và xử lý các đơn đặt hàng của người dùng, được hỗ trợ bởi MySQL DB và RabbitMQ-cần hai volume lưu trữ khối RWO.
- **Catalog**: Danh sách và chi tiết sản phẩm được hỗ trợ bởi MySQL DB-cần volume lưu trữ khối RWO.

![Amazon EKS và Trident](/images/3-blogstranslated/3.1-blog1/Sample-retail-store-application.png.png)
> *Hình 2. Ứng dụng mẫu cửa hàng bán lẻ đơn giản.*

## Các điều kiện tiên quyết

Để cấp phát tài nguyên thủ công được sử dụng trong hướng dẫn này, danh sách sau đây cung cấp các thành phần và phiên bản sử dụng. Ngoài ra, sử dụng cấp phát tài nguyên TerraForm để tự động toàn bộ việc triển khai được giới thiệu trong Bước 1 của hướng dẫn.
- Cụm Amazon EKS được triển khai phiên bản 1.32
  - [Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-id-agent-setup.html) add-on phiên bản v1.3.4
  - [NetApp Trident CSI add-on](https://docs.netapp.com/us-en/trident/trident-use/trident-aws-addon.html) phiên bản 25.06
  - [Amazon EBS CSI add-on](https://docs.aws.amazon.com/eks/latest/userguide/workloads-add-ons-available-eks.html#add-ons-aws-ebs-csi-driver) phiên bản v1.40.0-eksbuild.1
  - [Snapshot controller](https://docs.aws.amazon.com/eks/latest/userguide/csi-snapshot-controller.html) add-on phiên bản v8.0.0
    - Tạo VolumeSnapshotClass (Lớp Ảnh chụp nhanh Volume) cho [Amazon Elastic Block Storage (Amazon EBS)](https://aws.amazon.com/ebs/) và FSx for ONTAP (các manifest mẫu đã được cấp phát như một phần của triển khai Terraform).
  - [Load balancer controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) version 2.11.0
- Triển khai hệ thống file [FSx for ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/getting-started.html)

## Hướng dẫn

1. **Tạo kiến trúc cần thiết với Terraform**

Clone kho lưu trữ mẫu từ GitHub và tạo tất cả các tài nguyên liên quan bằng cách sử dụng mã Terraform trong kho lưu trữ đó:

```bash
git clone https://github.com/aws-samples/sample-eks-backup-with-trident-protect.git
cd sample-eks-backup-with-trident-protect/terraform
terraform init
terraform apply -auto-approve
```
*LƯU Ý: Script Terraform sẽ nhắc bạn nhập một địa chỉ IP công cộng (public IP address). Đây là địa chỉ của máy chủ (host) sẽ truy cập vào giao diện người dùng (UI) của ứng dụng mẫu.*

```bash
var.ui_service_public_ip 
The public IP addess of the host that will access the sample application >UI from the web browser  
Enter a value: A.B.C.D/32 
```
Quá trình triển khai này có thể mất 20-25 phút để hoàn tất. Khi kết thúc, đầu ra của lệnh sẽ trông giống như sau:

```bash
fsx-management-ip = toset([
  "10.0.1.13",
])
fsx-ontap-id = "fs-a1b2c3d4e5f6g7h8i"
fsx-svm-name = "ekssvm"
region = "us-east-1"
secret_arn = "arn:aws:secretsmanager:us-east-1:0123456789ab:secret:fsxn-password-secret-8DKLpwTi-8DLlwE"
zz_update_kubeconfig_command = "aws eks update-kubeconfig --name eks-protect-8DKLpwTi --alias eks-primary --region us-east-1"
```

Tiếp theo, hãy sao chép và chạy lệnh [Giao diện Dòng lệnh AWS (AWS CLI)](https://aws.amazon.com/cli/) từ đầu ra `update_kubeconfig_command` và kiểm tra xem bạn có thể truy cập được cụm (cluster) hay không bằng cách chạy lệnh `kubectl get nodes`

 ```bash
NAME                         STATUS   ROLES    AGE     VERSION
ip-10-0-1-120.ec2.internal   Ready    <none>   2m15s   v1.32.1-eks-5d632ec
ip-10-0-1-125.ec2.internal   Ready    <none>   95m     v1.32.1-eks-5d632ec
ip-10-0-2-42.ec2.internal    Ready    <none>   95m     v1.32.1-eks-5d632ec
ip-10-0-2-95.ec2.internal    Ready    <none>   2m9s    v1.32.1-eks-5d632ec
```
2. **Triển khai Trident Protect từ cụm EKS**

Chạy các lệnh sau đây để cài đặt Trident Protect:

QUAN TRỌNG: Hãy đảm bảo rằng bạn đã thay đổi `clusterName` thành tên cụm EKS của bạn trong môi trường.

```bash
helm repo add netapp-trident-protect https://netapp.github.io/trident-protect-helm-chart
helm install trident-protect netapp-trident-protect/trident-protect --set clusterName=eks-protect-ymd8hvHr --version 100.2506.0 --create-namespace --namespace trident-protect
```
Mong đợi output:

```bash
"netapp-trident-protect" has been added to your repositories
NAME: trident-protect
LAST DEPLOYED: Tue Jul 22 19:44:30 2025
NAMESPACE: trident-protect
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

3. **Tạo S3 bucket**

Nếu bạn đã có sẵn một S3 bucket , bạn có thể sử dụng nó. Nếu không, hãy làm theo các bước dưới đây để tạo một S3 bucket mới. Thay thế `<bucket_name>` và `<aws_region>` bằng các giá trị của bạn:
```bash
aws s3 mb s3://<bucket_name> --region <aws_region>
```

4. **Tạo EKS Secret để lưu trữ thông tin đăng nhập người dùng – Tùy chọn**

EKS Pod Identity là phương thức được khuyến nghị để xác thực IAM, và đã được cấu hình như một phần của quá trình triển khai Terraform ban đầu. Nếu bạn không muốn sử dụng EKS Pod Identity cho việc xác thực IAM, thì hãy tạo một Secret để lưu trữ accessKey và secretKey của người dùng Trident Protect AWS. Đảm bảo rằng thông tin đăng nhập người dùng bạn cung cấp có quyền truy cập cần thiết vào S3 bucket—tham khảo chính sách Amazon S3 mẫu [(Amazon S3 policy statement)](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-appvault-custom-resources.html#s3-compatible-storage-iam-permissions).

Sử dụng ví dụ sau để tạo Secret:

```bash
kubectl create secret generic <secret-name> \
--from-literal=accessKeyID=<accessKey> \
--from-literal=secretAccessKey=<seceretKey> \
-n trident-protect
```

5. **Tạo Trident Protect AppVault**

Hãy tạo Trident Protect `AppVault`. `AppVault` này sẽ trỏ đến S3 bucket—nơi cả snapshots lẫn nội dung sao lưu (bao gồm dữ liệu và metadata) được lưu trữ. `AppVault` được tạo trong namespace chuyên biệt `trident-protect` (đã tạo ở Bước 2), và có thể được bảo mật bằng [role-based access control (RBAC)](https://docs.netapp.com/us-en/trident/trident-protect/manage-authorization-access-control.html) để hạn chế quyền truy cập vào các đối tượng đặc quyền chỉ dành cho quản trị viên.

Tất cả các tác vụ còn lại sẽ được tạo trong namespace (không gian tên) ứng dụng gốc hoặc namespace đích trong trường hợp là các ví dụ khôi phục. Để tạo `AppVault`, hãy sử dụng tệp cấu hình mẫu `protect-vault.yaml`.

```bash
cd <repo>/manifests
```

Cập nhật các tham số sau đây:

- `providerConfig.s3.bucketName`: Tên S3 bucket (thùng chứa S3).
- `providerConfig.s3.endpoint`: Endpoint S3 nếu bucket không nằm trong Vùng (Region) `us-east-1`.
- `useIAM`: Sử dụng EKS Pod Identity để xác thực IAM.
- `providerCredentials.accessKeyID.name`: Tên EKS Secret từ bước trước.
- `providerCredentials.secretAccessKey.name`: Tên EKS Secret từ bước trước.

QUAN TRỌNG: Khi sử dụng `useIAM: true` với EKS Pod Identity, không được thiết lập (set) các tham số `providerCredentials.accessKeyID.name` và `providerCredentials.secretAccessKey.name`.

```bash
apiVersion: protect.trident.netapp.io/v1
kind: AppVault
metadata:
  name: amazon-s3-trident-protect-src-bucket
  namespace: trident-protect
spec:
  properties:
    dataMoverPasswordSecretRef: my-optional-data-mover-secret
  providerType: AWS
  providerConfig:
    s3:
      bucketName: trident-protect-blog
      endpoint: s3.amazonaws.com
      useIAM: true
  # providerCredentials:
  #   accessKeyID:
  #     valueFromSecret:
  #       key: accessKeyID
  #       name: s3-secret
  #   secretAccessKey:
  #     valueFromSecret:
  #       key: secretAccessKey
  #       name: s3-secret
```

Chạy lệnh dưới đây để tạo `AppValut`:

```bash
kubectl create -f protect-vault.yaml
```

Để kiểm tra Appvault được tạo thành công chưa, chạy lệnh:

```bash
kubectl get appvault -n trident-protect
```

Kết quả mong đợi:

```bash
NAME                STATE       ERROR   MESSAGE   AGE
eks-protect-vault   Available                     4s
```

6. **Tạo Trident Protect Application**

Để thực hiện các hoạt động bảo vệ dữ liệu trên các ứng dụng Amazon EKS của bạn, bạn cần tạo một tài nguyên Trident Protect Application. Một Application có thể được định nghĩa theo các cách sau:
- Là một namespace, theo đó mọi thứ thuộc về namespace đó phải được bảo vệ.
- Là một tập hợp con của một namespace dựa trên labels (nhãn), nếu bạn chỉ muốn bảo vệ một phần của namespace (ví dụ: chỉ các PVC).
- Nó có thể mở rộng (span) qua nhiều namespace.
- Một Application cũng có thể tính đến các tài nguyên trên toàn cụm (cluster-wide resources).

Để định nghĩa các namespaces mà tài nguyên ứng dụng tồn tại, hãy sử dụng `spec.includedNamespaces` và chỉ định labels của namespace hoặc tên namespace. Bạn có thể sử dụng tệp cấu hình mẫu [trident-application.yaml](https://owolabip/aws-eks-trident-protect-blog/-/blob/main/manifests/trident-application.yaml) này để định nghĩa một Application cho sample-app.

```bash
apiVersion: protect.trident.netapp.io/v1
kind: Application
metadata:
  name: sample-app
  namespace: tenant0
spec:
  includedNamespaces:
    - namespace:  tenant0
```
Chạy lệnh sau đây để tạo Application:

```bash
cd <repo>/manifests
kubectl create -f trident-application.yaml
```
Để kiểm tra ứng dụng đã được tạo thành công chạy lệnh dưới đây:
```bash
kubectl get application -n tenant0
```
Kết quả mong đợi:
```bash
NAME         PROTECTION STATE   AGE
sample-app   None               14s
```

## Hướng dẫn Sao lưu, Khôi phục và Di chuyển (Dữ liệu/Ứng dụng)
**Tạo sao lưu ứng dụng**

Trident Protect có sẵn một số tùy chọn bảo vệ dữ liệu sau:
1. [On-demand snapshot: Ảnh chụp nhanh theo yêu cầu](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-protect-apps.html#create-an-on-demand-snapshot)
2. [On-demand backup: Sao lưu theo yêu cầu](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-protect-apps.html#create-an-on-demand-backup)
3. [Data protection schedule: Lịch trình bảo vệ dữ liệu](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-protect-apps.html#create-a-data-protection-schedule)
4. [Application replication: Nhân bản ứng dụng](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-use-snapmirror-replication.html)
5. [Application migration: Di chuyển ứng dụng](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-migrate-apps.html)

Trong bài viết này, chúng tôi tập trung vào sao lưu theo yêu cầu (on-demand backup) và di chuyển ứng dụng (application migration). Tuy nhiên, bạn có thể đọc thêm về cách sử dụng các tùy chọn bảo vệ dữ liệu khác trong các tài liệu tham khảo đã đề cập trước đó.

Để tạo một bản sao lưu theo yêu cầu cho ứng dụng mẫu, bạn cần tạo một tài nguyên backup cho Application mà bạn vừa định nghĩa và trỏ nó đến `AppVault` đã tạo trên Amazon S3. Bạn có thể sử dụng tệp cấu hình mẫu `trident-backup.yaml` sau đây:

```bash
apiVersion: protect.trident.netapp.io/v1
kind: Backup
metadata:
  namespace: tenant0
  name: sample-app-backup-1
spec:
  applicationRef: sample-app
  appVaultRef: eks-protect-vault
```

Chạy lệnh dưới đây để tạo sao lưu:

```bash
cd <repo>/manifests
kubectl create -f trident-backup.yaml
```

Để kiểm tra xem Bản sao lưu đã được tạo thành công hay chưa, hãy chạy lệnh sau đây:

```bash
kubectl get backup -n tenant0
```

Kết quả mong đợi:

```bash
NAME                  APP          RECLAIM POLICY   STATE       ERROR   AGE
sample-app-backup-1   sample-app   Retain           Completed           9m33s
```

QUAN TRỌNG: Trạng thái của bản sao lưu có thể hiển thị là Running (Đang chạy) trong khi quá trình sao lưu đang diễn ra. Hãy chờ cho đến khi trạng thái chuyển sang **Completed** (Đã hoàn thành). Nếu trạng thái là **Failed** (Thất bại), hãy sử dụng lệnh 
`kubectl describe backup -n tenant0 sample-app-backup-1` để xem thêm chi tiết về lỗi đó.

Nếu bạn kiểm tra `Application` Protection State, trạng thái đó hiện được đặt là `Partial` (Một phần) vì bạn chỉ mới thiết lập một bản sao lưu theo yêu cầu (on-demand backup) cho ứng dụng của mình, nhưng chưa có lịch trình sao lưu (backup schedule) nào. Bạn có thể sử dụng `lệnh kubectl describe application -n tenant0` để kiểm tra trạng thái này.

**Trident Protect có sẵn một số tùy chọn khôi phục sau:**

1. Khôi phục từ Backup

- [Khôi phục từ bản sao lưu sang một namespace khác](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-restore-apps.html#restore-from-a-backup-to-a-different-namespace).
- [Khôi phục từ bản sao lưu về namespace gốc](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-restore-apps.html#restore-from-a-backup-to-the-original-namespace).
- [Khôi phục từ bản sao lưu sang một cụm](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-restore-apps.html#restore-from-a-backup-to-a-different-cluster).
2. Khôi phục từ Snapshot

- [Khôi phục từ snapshot sang một namespace khác](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-restore-apps.html#restore-from-a-snapshot-to-a-different-namespace).
- [Khôi phục từ snapshot  về namespace gốc](https://docs.netapp.com/us-en/trident/trident-protect/trident-protect-restore-apps.html#restore-from-a-snapshot-to-the-original-namespace).

Trong bài viết này, chúng tôi tập trung vào việc khôi phục bản sao lưu của ứng dụng mẫu (sample-app) sang một namespace khác trên cùng một cụm EKS. Bạn có thể đọc thêm về cách sử dụng các tùy chọn khôi phục khác trong các tài liệu tham khảo đã đề cập trước đó. Dưới đây là phần ôn tập về trường spec của tài nguyên BackupRestore (Sao lưu-Khôi phục):

- `spec.appArchivePath`: Đường dẫn lưu trữ ứng dụng (Archive Path) trong `AppVault`—nơi chứa nội dung bản sao lưu.

Bạn có thể truy xuất Archive Path bằng cách chạy lệnh sau đây trên tài nguyên backup của bạn:

```bash
kubectl get backup sample-app-backup-1 -n tenant0 -o jsonpath='{.status.appArchivePath}'
```

- `spec.appVaultRef`: Tên của AppVault—nơi chứa nội dung bản sao lưu.
- `spec.namespaceMapping`: Ánh xạ namespace nguồn của hoạt động khôi phục tới namespace đích.

Bạn có thể sử dụng tệp cấu hình mẫu `trident-protect-backup-restore.yaml` sau đây:

```bash
apiVersion: protect.trident.netapp.io/v1
kind: BackupRestore
metadata:
  name: sample-app-restore-1
  namespace: tenant1
spec:
  appArchivePath: <APP ARCHIVE PATH>
  appVaultRef: eks-protect-vault
  namespaceMapping: 
    - source: tenant0
      destination: tenant1
  resourceFilter:
    resourceSelectionCriteria: "Exclude"
    resourceMatchers:
      - kind: TargetGroupBinding
```

Nếu bạn cần chọn chỉ các tài nguyên cụ thể trong ứng dụng để khôi phục, hãy sử dụng cơ chế lọc (filtering) sau để bao gồm (Include) hoặc loại trừ (Exclude) các tài nguyên được đánh dấu bằng các labels (nhãn) cụ thể.
- `resourceFilter.resourceSelectionCriteria`: Sử dụng Include (Bao gồm) hoặc Exclude (Loại trừ) để bao gồm hoặc loại trừ một tài nguyên được định nghĩa trong resourceMatchers.
- `resourceFilter.resourceMatchers`: 
  - `resourceMatchers[].group`: Group (Nhóm) của tài nguyên cần lọc
  - `resourceMatchers[].kind`: Kind (Loại) của tài nguyên cần lọc
  - `resourceMatchers[].version`: Version (Phiên bản) của tài nguyên cần lọc
  - `resourceMatchers[].names`: Tên trong trường metadata.name của tài nguyên cần lọc
  - `resourceMatchers[].namespaces`: Namespaces trong trường metadata.name của tài nguyên cần lọc
  - `resourceMatchers[].labelSelectors`: Chuỗi bộ chọn label trong trường metadata.name của tài nguyên

Ví dụ, bạn có thể sử dụng đoạn mã mẫu này để bao gồm (include) các tài nguyên trong tệp cấu hình `BackupRestore` của bạn:

```bash
spec:
  resourceFilter:
    resourceSelectionCriteria: "Include"
    resourceMatchers:
      - group: my-resource-group-1
        kind: my-resource-kind-1
        version: my-resource-version-1
        names: ["my-resource-names"]
        namespaces: ["my-resource-namespaces"]
        labelSelectors: ["trident.netapp.io/os=linux"]
```

LƯU Ý: Quá trình khôi phục ứng dụng sử dụng dịch vụ loại `LoadBalancer` (Cân bằng tải) trong Amazon EKS. Do đó, bạn nên loại trừ các tài nguyên `TargetGroupBinding` để [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/) có thể tạo một `TargetGroupBinding` mới cho namespace mới mà không xung đột với ứng dụng hiện có trên cụm.

Chạy lệnh sau đây để tạo **BackupRestore**:

```bash
cd <repo>/manifestskubectl
create -f trident-protect-backup-restore.yaml
```

Để kiểm tra xem tài nguyên **BackupRestore** đã được tạo thành công hay chưa, hãy chạy lệnh sau đây:
```bash
kubectl get backuprestore -n tenant1
```

Kết quả mong đợi:
```bash
NAME                   STATE       ERROR   AGE
sample-app-restore-1   Completed           115s
```

QUAN TRỌNG: Trạng thái của `BackupRestore` có thể hiển thị là Running (Đang chạy) trong khi quá trình khôi phục đang diễn ra. Hãy chờ cho đến khi trạng thái chuyển sang `Completed` (Đã hoàn thành). Nếu trạng thái là `Failed` (Thất bại), hãy sử dụng lệnh `kubectl describe backuprestore -n tenant1 sample-app-restore-1` để xem thêm chi tiết về lỗi đó.

Sau khi quá trình khôi phục hoàn tất, bạn có thể xác minh ứng dụng đã được khôi phục đang hoạt động bằng cách truy cập endpoint của dịch vụ UI (giao diện người dùng) trong trình duyệt của bạn. Bạn có thể lấy endpoint của dịch vụ UI bằng cách chạy lệnh sau đây.

```bash
kubectl get svc ui -n tenant1 --output jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Nếu mọi thứ đều thành công, bạn sẽ thấy giao diện người dùng (UI) trông giống như Hình 3:

![alt text](![Amazon EKS và Trident](/images/3-blogstranslated/3.1-blog1/Restored-retail-store-application.png.png)
> *Hình 3. Ứng dụng cửa hàng bán lẻ đã được khôi phục.*

**Di chuyển giữa các dịch vụ lưu trữ**

Trong bước này, bạn sẽ di chuyển một phần các dịch vụ có trạng thái của ứng dụng mẫu từ dịch vụ lưu trữ này sang dịch vụ lưu trữ khác. Bạn nên thực hiện điều này bằng cách sử dụng tính năng `storageClassMapping` của tài nguyên Trident Protect `BackupRestore`. Bạn sẽ di chuyển cơ sở dữ liệu MySQL của dịch vụ catalog từ một Volume EBS sang một Volume FSx for ONTAP sử dụng LUN được trình bày qua iSCSI.

Để làm điều đó, bạn sẽ thực thi hai tài nguyên `BackupRestore`. Một tài nguyên sẽ khôi phục tất cả các tài nguyên và dữ liệu của ứng dụng mẫu ngoại trừ `catalog-mysql` statefulset; và tài nguyên còn lại sẽ khôi phục và di chuyển riêng `catalog-mysql` statefulset từ Amazon EBS sang lưu trữ khối FSx for ONTAP.

Hãy xem xét tệp cấu hình mẫu `trident-protect-migrate.yaml`. Sử dụng `resourceFilter` để loại trừ và bao gồm các tài nguyên khỏi quá trình khôi phục, và sử dụng `storageClassMapping` để di chuyển các tài nguyên có trạng thái sang các hệ thống lưu trữ bên dưới khác.

```bash
resourceFilter:
    resourceSelectionCriteria: "Exclude"
    resourceMatchers:
      - kind: StatefulSet
        names: ["catalog-mysql"]
      - kind: TargetGroupBinding
```

QUAN TRỌNG: Hãy đảm bảo bạn cập nhật trường `spec.appArchivePath` trên cả hai tài nguyên [BackupRestore]. Bạn có thể truy xuất Đường dẫn Lưu trữ bằng cách chạy lệnh sau đây trên bản sao lưu của bạn:

```bash
kubectl get backup sample-app-backup-1 -n tenant0 -o jsonpath='{.status.appArchivePath}'
```

Bạn có thể kiểm tra ứng dụng nguồn và thấy rằng volume (khối lưu trữ) ban đầu đang sử dụng Amazon EBS bằng cách dùng chính lệnh đó:
```bash
kubectl get pvc data-catalog-mysql-0 -n tenant0
```
Kết quả mong đợi:
```bash
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-catalog-mysql-0   Bound    pvc-0d795501-aca2-4e10-98b5-111ecc3aef2c   30Gi       RWO            ebs-csi        <unset>                 71m
```
Chạy lệnh sau đây để tạo các tài nguyên `BackupRestore` cần thiết cho quá trình di chuyển:

```bash
cd <repo>/manifests
kubectl create -f trident-protect-migrate.yaml
```

Để kiểm tra xem các tài nguyên BackupRestore đã được tạo thành công hay chưa, hãy chạy lệnh sau đây:
```bash
kubectl get backuprestore -n tenant2
```
QUAN TRỌNG: Trạng thái của tài nguyên `BackupRestore` có thể hiển thị là `running` (Đang chạy) trong khi quá trình khôi phục đang diễn ra. Hãy chờ cho đến khi trạng thái chuyển sang `Completed` (Đã hoàn thành). Nếu trạng thái là `Failed` (Thất bại), hãy sử dụng lệnh `kubectl describe backuprestore -n tenant2 sample-app-migrate-#` để xem thêm chi tiết về lỗi đó.
```bash
Kết quả mong đợi:
NAME                   STATE       ERROR   AGE
sample-app-migrate-1   Completed           26m
sample-app-migrate-2   Completed           38s
```

Bạn có thể kiểm tra xem PVC (Persistent Volume Claim) của `catalog-mysql` đã được di chuyển từ Amazon EBS sang FSx for ONTAP hay chưa bằng cách chạy lệnh sau đây:
```bash
kubectl get pvc data-catalog-mysql-0 -n tenant2
```
Kết quả mong đợi:
```bash
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      VOLUMEATTRIBUTESCLASS   AGE
data-catalog-mysql-0   Bound    pvc-8e45fa11-13ad-490d-b70c-d649b6fef4e9   30Gi       RWO            trident-csi-san   <unset>                 118s
```
Sau khi quá trình di chuyển (migration) hoàn tất, bạn có thể xác minh ứng dụng đã di chuyển đang hoạt động bình thường bằng cách truy cập endpoint của dịch vụ UI (giao diện người dùng) trong trình duyệt của bạn. Bạn có thể lấy endpoint của dịch vụ UI bằng cách chạy lệnh sau đây:
```bash
kubectl get svc ui -n tenant2 --output jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```
Nếu mọi thứ đều thành công, bạn sẽ thấy giao diện người dùng (UI) trông giống như Hình 4:

![alt text](![Amazon EKS và Trident](/images/3-blogstranslated/3.1-blog1/Restored-retail-store-application-after-PVC-migration.png.png)
> *Hình 4. Ứng dụng cửa hàng bán lẻ đã được khôi phục sau khi di chuyển PVC.*

Tuyệt vời! Bạn đã hoàn thành và nắm vững các kiến thức cơ bản về sao lưu (backup), khôi phục (recovery) và di chuyển (migration) ứng dụng trên Amazon EKS sử dụng NetApp Trident Protect. Hướng dẫn này có thể được sử dụng như một thực tiễn tốt nhất để triển khai việc bảo vệ, di chuyển và phục hồi sau thảm họa (disaster recovery) trong môi trường của bạn.

## Dọn dẹp
Để tránh các khoản phí không cần thiết, hãy đảm bảo rằng bạn đã xóa tất cả các tài nguyên đã được tạo bằng Terraform bằng cách chạy kịch bản sau đây từ terminal (cửa sổ lệnh) của bạn:
```bash
sh ../scripts/cleanup.sh
```
Nếu bạn đã tạo một S3 bucket mới cho bài tập này, thì hãy dọn dẹp bằng cách đi tới giao diện điều khiển Amazon S3 (Amazon S3 console), xóa hết nội dung (emptying) của bucket, và xóa bucket đó.
## Kết luận
Tóm lại, NetApp Trident Protect cung cấp một giải pháp mạnh mẽ để hợp lý hóa (streamlining) việc bảo vệ dữ liệu cho các môi trường Kubernetes. Phương pháp này đáp ứng nhu cầu thiết yếu về các chiến lược sao lưu toàn diện trong các kiến trúc cloud-native (ứng dụng đám mây). NetApp Trident Protect cho phép người dùng tạo và quản lý một cách hiệu quả các bản sao lưu của toàn bộ namespaces , persistent volumes (khối lưu trữ lâu dài) và các tài nguyên Amazon EKS thiết yếu khác, qua đó đảm bảo tính liên tục của hoạt động kinh doanh và tuân thủ các yêu cầu về bảo vệ dữ liệu.

Hướng dẫn từng bước được trình bày trong bài viết này đã chứng minh sự dễ dàng trong việc thiết lập và sử dụng NetApp Trident Protect với Amazon EKS, Amazon FSx for NetApp ONTAP và Amazon EBS. Cảm ơn bạn đã đọc bài viết này. Hãy để lại bất kỳ nhận xét hoặc câu hỏi nào của bạn trong phần bình luận.
