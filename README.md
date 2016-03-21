# Hướng dẫn sử dụng LVM cơ bản

### 1. LVM là gì?

LVM (Logical Volume Manager) là một phương pháp cho phép ấn định không gian đĩa cứng thành những Logical Volume khiến cho việc thay đổi kích thước trở nên dễ dàng (so với partition). Với kỹ thuật LVM bạn có thể thay đổi kích thước mà không cần phải sửa lại partition table của OS. Điều này thực sự hữu ích với những trường hợp bạn đã sử dụng hết phần bộ nhớ còn trống của partition và muốn mở rộng dung lượng của nó.

### 2. LVM dùng để làm gì?

LVM là kỹ thuật quản lý việc thay đổi kích thước lưu trữ của ổ cứng

-	Không để hệ thống bị gián đoạn hoạt động
-	Không làm hỏng dịch vụ
-	Có thể kết hợp Hot Swapping (thao tác thay thế nóng các thành phần bên trong máy tính)

### 3.	Cấu trúc của LVM gồm những gì?

<img src="https://camo.githubusercontent.com/713a3058b8a31f2686108f71d0ba494fc8317adb/687474703a2f2f692e696d6775722e636f6d2f556154617475622e706e67" />

- **Hard Drives**: là những ổ cứng vật lý mới được thêm vào chưa được phân vùng hay Format để sử dụng (Ví dụ: /dev/sda)
- **Physical volumes**: là những đĩa vật lý hoặc phân vùng đĩa của bạn chẳng hạn như /dev/hda hoặc /dev/hdb1. Khi bạn muốn sử dụng chỉ cần mount vào thôi. Đối với việc sử dụng LVM chúng ta có thể kết hợp nhiều physical volumes thành volume groups
- **Volume groups**: là một nhóm bao gồm các physycal volumes thực và dung lượng này được sử dụng để tạo ra các logical volumes, trong đó bạn có thể làm được những điều như sau : tạo, thay đổi kích thước, gỡ bỏ và sử dụng. Bạn có thể xem volume group như 1 “phân vùng ảo”
- **Logical volumes**: là những volumes cuối cùng sau khi mount vào hệ thống của mình, bạn có thể thêm vào, gỡ bỏ và thay đổi kích thước một cách nhanh chóng. Kể từ khi chúng chứa trong các volume group bạn có thể làm cho nó lơn hơn bất kỳ physical volume đơn lẻ mà bạn muốn. (ví dụ bạn có 4 ổ đĩa mỗi ổ 5GB khi bạn kết hợp nó lại thành 1 volume group 20GB, và bạn có thể tạo ra 2 logical volumes mỗi disk 10GB)

### 4. Hướng dẫn LVM cơ bản
#### 4.1 Chuẩn bị

- Hệ điều hành LINUX
- Thêm các ổ cứng vào hệ thống máy ảo

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/chuanbi_zpszxgqhonz.png" />
