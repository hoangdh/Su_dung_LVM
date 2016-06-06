# Hướng dẫn sử dụng LVM cơ bản và nâng cao

###Mục lục:
[I. CƠ BẢN ](#1)
- [1. LVM LÀ GÌ? ](#1.1)
- [2. LVM DÙNG ĐỂ LÀM GÌ? ](#1.2)
- [3. CẤU TRÚC CỦA LVM ](#1.3)
- [4. CẤU HÌNH CƠ BẢN ](#1.4)
[II. NÂNG CAO ](#2)
- [1. SNAPSHOT ](#2.1)
- [2. THIN PROVISIONING](#2.2)
- [3. STRIPING I/O ](#2.3)
- [4. LVM MIGRATION ](#2.4)

<a name="1"></a>
## I. CƠ BẢN

<a name="1.1"></a>
### 1. LVM là gì?

LVM (Logical Volume Manager) là một phương pháp cho phép ấn định không gian đĩa cứng thành những Logical Volume khiến cho việc thay đổi kích thước trở nên dễ dàng (so với partition). Với kỹ thuật LVM bạn có thể thay đổi kích thước mà không cần phải sửa lại partition table của OS. Điều này thực sự hữu ích với những trường hợp bạn đã sử dụng hết phần bộ nhớ còn trống của partition và muốn mở rộng dung lượng của nó.

<a name="1.2"></a>
### 2. LVM dùng để làm gì?

LVM là kỹ thuật quản lý việc thay đổi kích thước lưu trữ của ổ cứng

-	Không để hệ thống bị gián đoạn hoạt động
-	Không làm hỏng dịch vụ
-	Có thể kết hợp Hot Swapping (thao tác thay thế nóng các thành phần bên trong máy tính)

<a name="1.3"></a>
### 3.	Cấu trúc của LVM gồm những gì?

<img src="https://camo.githubusercontent.com/713a3058b8a31f2686108f71d0ba494fc8317adb/687474703a2f2f692e696d6775722e636f6d2f556154617475622e706e67" />

- **Hard Drives**: là những ổ cứng vật lý mới được thêm vào chưa được phân vùng hay Format để sử dụng (Ví dụ: /dev/sda)
- **Parittion**: là những phân vùng của Hark Drives đã được phân định sẵn kích thước. Ví dụ: sda có dung lượng 4GB, thì ta có thể chia cho phân vùng sda1 có 3GB, sda2 có 1 GB. Hiểu nôm na, sda (Hard Drives) là cha và các Partition là con. Một cha có thể có nhiều con nhưng mỗi con chỉ có thể có 1 cha.
- **Physical volumes**: là những đĩa vật lý hoặc phân vùng đĩa của bạn chẳng hạn như /dev/hda hoặc /dev/hdb1. Khi bạn muốn sử dụng chỉ cần mount vào thôi. Đối với việc sử dụng LVM chúng ta có thể kết hợp nhiều physical volumes thành volume groups
- **Volume groups**: là một nhóm bao gồm các physycal volumes thực và dung lượng này được sử dụng để tạo ra các logical volumes, trong đó bạn có thể làm được những điều như sau : tạo, thay đổi kích thước, gỡ bỏ và sử dụng. Bạn có thể xem volume group như 1 “phân vùng ảo”
- **Logical volumes**: là những volumes cuối cùng sau khi mount vào hệ thống của mình, bạn có thể thêm vào, gỡ bỏ và thay đổi kích thước một cách nhanh chóng. Kể từ khi chúng chứa trong các volume group bạn có thể làm cho nó lơn hơn bất kỳ physical volume đơn lẻ mà bạn muốn. (ví dụ bạn có 4 ổ đĩa mỗi ổ 5GB khi bạn kết hợp nó lại thành 1 volume group 20GB, và bạn có thể tạo ra 2 logical volumes mỗi disk 10GB)

<a name="1.4"></a>
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

Tuy nhiên, lệnh `mount` chỉ dùng để mount LV tạm thời trong phiên làm việc sau khi khởi động lại sẽ mất muốn sử dụng lại thì phải mount. Ta ghi thêm vào file `/etc/fstab`

`echo "/dev/vg-demo/lv-demo   /testLV     ext3       rw,noatime    0 0"  >> /etc/fstab`

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
### BONUS
#### Đổi tên LV, VG

- Đối với LV, ta dùng lệnh với cấu trúc sau:

`lvrename <tên-vg> <tên-lv-cũ> <tên-lv-mới>`

Ví dụ: `lvrename vg-demo lv-demo demo`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/rename_zpszjsoxsxu.png" />

- Đối với VG, ta dùng lệnh với cấu trúc sau:

`gvrename <tên-vg-cũ> <tên-vg-mới>`

Ví dụ: `vgrename vg-demo demo`

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/lvm/rename_vg_zpsuzdfb3of.png" />

<a name="2"></a>
## II. Nâng cao

<a name="2.1"></a>
### 1. Snapshot

<img src="http://www.tecmint.com/wp-content/uploads/2014/08/Take-Snapshot-in-LVM.jpg" />

Snapshot là một tính năng dùng để lưu lại dữ liệu tại một thời điểm nào đó.

##### Tạo một snapshot

Thư mục trước khi tạo snapshot:

<img src="http://image.prntscr.com/image/69ae7a248c8045e88e7e6874e5fdd826.png" />

Tạo một snapshot bằng câu lệnh:

```
 lvcreate -L 1GB -s -n snap-lv-accounting /dev/vg-meditech/lv-accounting
```

- `-s`: Tạo snapshot
- `-n`: Tên của snapshot

<img src="http://image.prntscr.com/image/b45e8aed379640e5a972375140d890ec.png" />

*Lưu ý:* Câu lệnh trong hình tôi tạo snapshot với dung lượng là 3Gb nhưng LV mà tôi muốn snapshot chỉ có 2Gb nên hệ thống đã tự động giảm xuống.

Xem thông tin của Snapshot:

```
lvdisplay /dev/vg-meditech/snap-lv-accounting
```

<img src="http://image.prntscr.com/image/4a9ef51b27964fff87c3645506940179.png" />

Copy một số file:

<img src="http://image.prntscr.com/image/b8bb5ed4499a498f9265936189f32b4b.png" />

Kiểm tra sự thay đổi:

<img src="http://image.prntscr.com/image/4958f3274546464687d0a188f08fac21.png" />

##### Tăng dung lượng snapshot

Ở một mức nào đó, ổ cứng của chúng ta đầy lên và vượt quá dung lượng của snapshot mà chúng ta đã tạo ở bên trên. (Bên trên, tôi đã tạo snapshot bằng với dung lượng ổ cứng vì thế không phải lo nghĩ gì cả.)

Để tăng thêm dung lượng của snapshot ta sẽ thực hiện như sau:

```
lvextend -L +1G /dev/vg-meditech/snap-lv-accounting
```

Muốn hệ thống tự động dãn LV snapshot cho chúng ta thì phải cấu hình một chút như sau:

Đầu tiên mở file cấu hình LVM: `vi /etc/lvm/lvm.conf`

Tìm đến dòng có keyword như sau:

<img src="http://www.tecmint.com/wp-content/uploads/2014/08/LVM-Configuration.jpg" />

Giải thích: Khi dung lượng của snapshot đạt tới 70% tổng dung lượng mà chúng ta tạo ở trên, thì tự động hệ thống sẽ tăng thêm cho nó 20% dung lượng.

##### Restore lại dữ liệu tại thời điểm snapshot

Để quay về trạng thái mà khi ta tạo snapshot, dùng câu lệnh sau:

Trước đó, chúng ta phải umount thư mục

```
umount /accounting
```

Restore lại bằng lệnh:

```
lvconvert --merge /dev/vg-meditech/snap-lv-accounting
```

Sau khi thực hiện xong, snapshot sẽ bị xóa. Chúng ta thực hiện thao tác `mount` lại ổ vào thư mục và kiểm tra dữ liệu.

```
mount /dev/vg-meditech/lv-accounting /accounting
```
<a name="2.2"></a>
### 2. Thin Provisioning

<img src="http://www.tecmint.com/wp-content/uploads/2014/08/Setup-Thin-Provisioning-in-LVM.jpg" />

Chức năng này cho phép chúng ta cấp động dữ liệu cho người dùng.

Ví dụ chúng ta có một ổ cứng 15GB, cấp cho 2 người dùng mỗi người 5GB sử dụng. Một người dùng thứ 3 muốn sử dụng 5GB. Như vậy chúng ta đã hết lưu lượng khi sử dụng (Thick Volume).

Cũng trong trường hợp trên, chúng ta dùng Thin Provisioning để giải quyết vấn đề này. Thin Provisioning được hiểu nôm na là "dùng bao nhiêu thì lấy từng ấy" khác với Thick Provisioning (Cấp bao nhiêu thì lấy luôn từng ấy - cách này khá lãng phí khi người dùng không dùng hết lưu lượng được cấp). Quay trở lại với Thin Provisioning, ví dụ chúng ta có 15GB như ví dụ bên trên, khi đã cấp cho 3 người dùng 5GB mà 3 người này không lưu trữ hết dung lượng được cấp thì chúng ta có thể tận dụng khoảng không gian trống đó để cấp cho người dùng thứ 4 mà không phải gắn thêm ổ mới ngay lập tức.

Tuy nhiên, đây chỉ là cách "chữa cháy" tạm thời khi chúng ta chưa có khả năng mở rộng dung lượng ổ cứng vật lý tạm thời. Chúng ta phải mở rộng dung lượng vật lý càng sớm càng tốt, tránh rủi ro dữ liệu bị ghi đè, hoặc mất mát khi xung đột.

##### Tạo một Thin Volume Group

<img src="http://image.prntscr.com/image/2b5ce9fa1613497fad8529ce298a8938.png" >

Lưu ý: Chúng ta tạo VG Thin Provisioning như tạo một VG bình thường nhưng với tùy chọn `-s` với kích cỡ mặc định là 32M

```
vgcreate -s 32M vg-thin /dev/sdc1
```

Xem lại dung lượng của Thin VG vừa tạo bằng lệnh `vgs`

<img src="http://image.prntscr.com/image/012c427b74024c4da9190319b941faa7.png" >

##### Tạo một LV Thin Pool từ Thin VG

```
# lvcreate -L 6G --thinpool thin-meditech vg-thin
```

<img src="http://image.prntscr.com/image/0cb644866bcf4718abe924b2e8b1dcfc.png" >

Xem thông tin của LV Thin vừa tạo

<img src="http://image.prntscr.com/image/4a592e7711c34ca296923e8a4c79536b.png" >

##### Tạo các Thin Volume

```
lvcreate -V 2G --thin -n thin-si1 vg-thin/thin-meditech
lvcreate -V 2G --thin -n thin-si2 vg-thin/thin-meditech
lvcreate -V 2G --thin -n thin-si3 vg-thin/thin-meditech
lvcreate -V 2G --thin -n thin-si4 vg-thin/thin-meditech
```

<img src="http://image.prntscr.com/image/967064587ba84d6091892bf6d48094af.png" >

Xem lại thông tin của các LV vừa tạo

<img src="http://image.prntscr.com/image/04e099c123fa445d886026d817c8f9c2.png" >

##### Tạo thư mục, Format và Mount sử dụng

Tạo thư mục, ở đây tôi sẽ tạo thư mục ở `/mmt`:

```
mkdir /mnt/si{1..4}
```

<img src="http://image.prntscr.com/image/6db260d4e28048b984500ff194e049c1.png" >

Format các LV:

```
mkfs.ext3 /dev/vg-thin/thin-si1 && mkfs.ext3 /dev/vg-thin/thin-si2 && mkfs.ext3 /dev/vg-thin/thin-si3  mkfs.ext3 /dev/vg-thin/thin-si4
```

Mount lên các thư mục vừa tạo:

<img src="http://image.prntscr.com/image/35069393f8ec44969f19c216ea357119.png" >

```
mount /dev/vg-thin/thin-si1 /mnt/si1
mount /dev/vg-thin/thin-si2 /mnt/si2
mount /dev/vg-thin/thin-si3 /mnt/si3
mount /dev/vg-thin/thin-si4 /mnt/si4
```

Tôi vừa tạo 4 LV thin có dung lượng là 2GB với một ổ VG chỉ có 6GB. Tuy nhiên, như tôi đã nói ở trên là cách này chỉ để "chữa cháy" khi phải cấp cho một tài nguyên dùng tạm, cần phải thêm ổ cứng vật lý ngay.

<a name="2.3"></a>
### Stripping IO

<img src="http://www.tecmint.com/wp-content/uploads/2014/09/LVM-Striping.jpeg">

Phân chia dữ liệu đồng đều trên các đĩa (PV).

##### Thêm các PV

Ở đây, tôi dùng 2 ổ 8GB.

```
pvcreate /dev/sd[c-d]1 -v
pvs
```

<img src="http://image.prntscr.com/image/f39b8377f9a846eba2f8b0b18c038c2c.png">

##### Tạo VG với tên vg-strip

```
vgcreate -s 16M vg-strip /dev/sd[c-d]1 -v
vgs vg-strip
```

- `-s`: Xác định kích cỡ vật lý của VG
- `-v`: Verbose: hiển thị thông tin làm việc của lệnh

<img src="http://image.prntscr.com/image/a3616ec880ea492b946cd56a3f5bbd4c.png">

##### Tạo LV xác định dung lượng và số Tripping

```
lvcreate -L 1GB -n lv-si-strip -i2 vg-strip
```

- `-L`: Kích cỡ của LV
- `-n`: Tên của LV
- `-i`: Số lượng Stripping

<img src="http://image.prntscr.com/image/e0e90d6935be4fc891335755c3a96cc6.png" >

Ở hình trên, chúng ta có thể thấy kích cỡ mặc định của strip là 64KB, chúng ta có thể thay đổi chúng bằng tùy chọn `-I`.

Để xem chi tiết về LV vừa tạo ta thêm `-m` vào câu lệnh:

```
lvdisplay vg-strip/lv-si-strip -m
```

<img src="http://image.prntscr.com/image/b3d8cd6153424d599331e25743c45fd9.png" >

- `1`: Số tripe
- `2`: Kích thước mặc định của stripe

Tiếp theo là format và mount lên sử dụng.

<a name="2.4"></a>
### LVM Migration

Tính năng này vô cùng nổi bật và hữu ích, chúng ta có thể di chuyển dữ liệu sang disk mới mà không mất mát dữ liệu và downtime.

##### Thêm ổ mới vào

<img src="http://image.prntscr.com/image/063f50cb13b64c0d958f1f8b341e4f88.png">

Hiện tại, tôi muốn thay sdb1 (8GB) bằng sdc1 (10GB) mà không làm mất dữ liệu hay downtime.

<img src="http://image.prntscr.com/image/8f4a642e2ad3458a8ef156f0aceda250.png">

Xem lại thông tin:

```
vgdisplay vg-meditech -v
```

<img src="http://image.prntscr.com/image/ae34359f4dae45f4b33c14d50792658c.png">

##### Chuyển dữ liệu dùng phương thức Mirroring, sử dụng câu lệnh `lvconvert`

```
lvconvert -m 1 /dev/vg-meditech/lv-si /dev/sdb1
```

- `-m`: Mirror
- `1`: Thêm một Morrior đơn (sdb1)

<img src="http://image.prntscr.com/image/c31607f26533429db228606a9074b38f.png" >

##### Tháo bỏ ổ cũ (sdc1) ra khỏi VG

```
lvconvert -m 0 /dev/vg-meditech/lv-si /dev/sdc1
```

<img src="http://image.prntscr.com/image/731fe3c874234b1485f1568900d5fc4c.png">

```
vgreduce /dev/vg-meditech /dev/sdc1
```

<img src="http://image.prntscr.com/image/c8f6bf668dcb40a2ab05b714f88d4e1a.png">

Kiểm tra lại thì sdc1 (8GB) đã bị xóa khỏi LV và thay vào đó sdb1 (10GB).

<img src="http://image.prntscr.com/image/97e153df7751456bb6fa75233aeb28f1.png">



