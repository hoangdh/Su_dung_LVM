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

#### 4.2 Tạo các **Partition**
Kiểm tra Hệ thống đã nhận các ổ đã gắn vào hay chưa

`lsblk`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/lietke_zpsdatdtutb.png" />

Tạo ra các **Partition** bằng lệnh ``fdisk``, gắn nhãn (Lable) 8e (LVM) cho Partition. Thao tác trên **sdb**, các ổ còn lại làm tương tự.
	
<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/chiao1_zpsqd61jir9.png" />

Bấm **m** để xem các hướng dẫn

Bấm **n** để tạo mới 1 Partition

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/chiao2_zpsh88jc6jx.png" />

Bấm ``p`` để tạo primary partition (Mỗi ổ chỉ tạo được tối đa 4 primary partititon), tùy vào mục đích sử dụng mà chúng ta chia Hard-drives thành các partition có dung lượng khác nhau. Ở ví dụ này, chỉ chia 1 partition chính. Sử dụng giá trị mặc định để lấy tất cả dung lượng của ổ.

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/chiao3_zpslnqihyaf.png" />

Thay đổi lable của partition

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/chiao4_zpsdt0ajuta.png" />

Bấm `w` để lưu lại những gì ta vừa thiết đặt.

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/chiao5_zpsa80cay5i.png" />
#### 4.3 Tạo Physical Volume (PV)

Tạo một **PV** bao gồm 3 **Partition** (`/dev/sdb1` `/dev/sdc1` và `/dev/sdd1`) vừa tạo ở bước trên như sau:

`pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/pv1_zpsbdccojhv.png" />

#### 4.4 Tạo Group Volume (GV)

Từ 3 PV ở trên, ta có tạo một GV bằng lệnh

`gvcreate vg-demo /dev/sdb1 /dev/sdc1 /dev/sdd1`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/vg1_zps5rb3yqgk.png" />

Kiểm tra xem `vg-demo` đã được tạo hay chưa bằng lệnh `vgs`, kiểm tra các PV đã nằm trong VG chưa bằng lệnh `pvs`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/vg2_zps2iegrqwb.png" />

#### 4.4 Tạo Logical Volume (LV)

Một GV có thể tạo được dùng nhiều LV với kích cỡ khác nhau nhưng không thể vượt quá size của GV bằng lệnh sau:

 `lvcreate -L 5G -n lv-demo vg-demo`
 
 - `-L` : Chỉ ra dung lượng của LV
 - `-n` : Chỉ ra tên của LV
 - `lv-demo` : Tên của LV
 - `vg-demo` : Tên của VG

Tạo và kiểm tra lại: `lvs`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/lv1_zpse2smpfqp.png" />

#### 4.5 Sử dụng LV

##### a. Format

Ta có thể Format LV ở các định dạng ext2, ext3, xfs,... tùy theo mục đích sử dụng.

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/format_zpsityxxd1n.png" />

##### b. Mount và sử dụng

Tạo một folder `/testLV` để mount và sử dụng LV vừa format
`mkdir /testLV`

Mount

`mount /dev/vg-demo/lv-demo /testLV`

Dùng `df -h` để kiểm tra xem đã nhận đầy đủ dung lượng và đúng thư mục hay chưa

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/mount_zps6i5qklyh.png" />

#### 5. Thay đổi dung lượng của LV và VG trong LVM

##### 5.1 Với LV

Sau khi sử dụng một thời gian, không gian đĩa bị thu hẹp dần. Vì thế chúng ta cần tăng không gian của LV để sử dụng tiếp mà dữ liệu cũ vẫn được bảo toàn, nguyên vẹn. Làm theo hướng dẫn sau để tăng kích thước của LV.
#####*Tăng dung lượng*
*Kiểm tra dung lượng của VG*

`vgdisplay`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/mount_zps6i5qklyh.png" />

Như ta thấy ở hình, VG còn trống ~45 GB như vậy có thể cấp phát và tạo mới các LV một cách dễ dàng như sau:

`lvextend -L +1G /dev/vg-demo/lv-demo`

`-L` là tùy chọn để tăng kích thước LV

`+1G` định nghĩa cho hệ thống hiểu là tăng thêm, ở đây ta có thể tăng theo B (tối thiểu 512B), M, G

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/lvextend_zpszgah7qjp.png" />

Lúc này, hệ thống chưa nhận được dung lượng vừa thêm cho LV, để làm được điều này chúng ta phải thực hiện câu lệnh

`resize2fs /dev/vg-demo/lv-demo`

Và kiểm tra lại bằng lệnh:

`# dh -h`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/resize2fs_zpsodkndkmg.png" />

#####*Giảm dung lượng LV*

###*Khuyến cáo*: Điều này sẽ làm mất hết dữ liệu hiện tại, bởi vậy khi *Giảm dung lượng LV*, chúng ta cần *Backup* lại toàn bộ dữ liệu hiện tại trước khi thao tác.

Trước khi thực hiện tao tác giảm dung lượng LV, ta cần phải Unmount LV cần thay đổi:

`umount /testLV`

Sau đó, thực hiện thay đổi dung lượng bằng lệnh và xác nhận với hệ thống:

`lvreduce -L 3G /dev/vg-demo/lv-demo`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/reduce_zpsrlex6mcz.png" />

Format lại LV:

`mkfs.etx3 /dev/vg-demo/lv-demo`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/formatlv_zpsnlr9c6jz.png" />

Và mount lại để sử dụng và kiểm tra xem đã đúng với yêu cầu của mình chưa:

`mount /dev/vg-demo/lv-demo /testLV`

`df -h`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/mount-df_zpsyhy7xckc.png" />

##### 5.2 Với VG

Việc thay đổi không gian của VG đồng nghĩa với việc thêm hoặc bớt đi các PV. Ở trên, ta đã cho tất cả các PV vào trong `vg-demo`. Vì thế, ta sẽ giảm đi nó đi trước sau đó thêm lại. :D

##### Giảm dung lượng:

Ta kiểm tra PV nào đang ở trong `vg-demo` bằng lệnh

`pvs`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/pvs_zpsfhebt58v.png" />

Bỏ `/dev/sdc1` ra khỏi `vg-demo`

`vgreduce vg-demo /dev/sdc1`

Và kiểm tra lại bằng:

`pvs`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/sdc1_zpscwbe8tso.png" />

##### Tăng dung lượng:

*Ta hiểu bản chất của việc tăng dung lượng là thêm mới một PV vào trong VG.*

Bước trên, ta đã bỏ `/dev/sdc1` ra khỏi `vg-demo` thì bây giờ, chúng ta sẽ thêm nó lại vào bằng lệnh:

`vgextend /dev/vg-demo /dev/sdc1`

Kiểm tra lại:

`pvs`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/ex_zpswtnz2esv.png" />

#### 6. Xóa LV, VG, PV

Ngược lại với quá trình tạo, thay vì ta phải tạo các PV xong đến VG và cuối cùng là "phân chia" các LV thì ta phải xóa các LV, sau đó đến VG và cuối cùng là các PV. Quy trình lần lượt như sau:

##### 6.1 Xóa LV

Đầu tiên, ta phải `umount` nó trước sau đó xóa PV:

`umount /dev/vg-demo/lv-demo`

Xóa PV vừa `umount`

`lvremove /dev/vg-demo/lv-demo`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/lvremove_zpsco3ba8tv.png" />

##### 6.2 Xóa VG

Sau khi xóa PV xong, ta có thể xóa VG bằng lệnh:

`vgremove /dev/vg-demo`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/vgremove_zpsijgtubej.png" />

##### 6.3 Xóa PV

Cuối cùng đến PV, ta dùng lệnh: 

`pvremove /dev/sda1 /dev/sdb1 /dev/sdc1`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/pvremove_zpszbdj08bi.png" />
