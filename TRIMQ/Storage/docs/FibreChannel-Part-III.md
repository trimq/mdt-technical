# Fibre Channel SAN phần III: Redundancy và Multipathing

Bài viết này là phần cuối cùng của series bài về Fibre Channel. Xem các bài viết trước theo link dưới đây

[Phần I: Địa chỉ FCP và WWPN](https://github.com/trimq/mdt-technical/blob/master/TRIMQ/Storage/docs/FibreChannel-Part-I.md)

[Phần II: ZONING, LUN MASKING và FABRIC LOGIN](https://github.com/trimq/mdt-technical/blob/master/TRIMQ/Storage/docs/FibreChannel-Part-II.md)

## Mục lục

- [1. Fibre Channel Redundancy](#1)




-----------------------------------------------

<a name="1"></a>

### 1. Fibre Channel Redundancy

Các máy chủ truy cập được vào hệ thống lưu trữ sẽ luôn luôn là nhiệm vụ rất quan trọng của doanh nghiệp. Bởi vậy, chúng ta không muốn hỏng hóc ở bất cứ điểm nào. Do vậy Redundant Fibre Channel được đưa ra. Chúng ta hiểu như là hệ thống sẽ có Fabric A và Fabric B hoặc là SAN A và SAN B. Mỗi server và các hệ thống lưu trữ nên được kết nối với cả 2 fabrics với các ports HBA dự phòng.

Các switch phân tán của Fibre Channel sẽ chia sẻ thông tin với nhau (Domain ID, cở sở dữ liệu FCNS và zoning). Khi chúng ta cấu hình zoning trong một fabric, chỉ cần cấu hình trên một Switch, nó sẽ tự động phân phối đến các Switch khác. Điều này là rất tiện lợi nhưng cũng có một vài nhược điểm ở đây. Bởi vì khi ta cấu hình sai thì nó sẽ nhân bản ra các switch khác trong fabric. Nếu một lỗi xảy ra trong fabric A được truyền sang fabric B, nó sẽ làm cho cả 2 fabric bị DOWN và sẽ mất kết nối của series vào máy chủ lưu trữ. Điều này rất nguy hiểm.

Vì lí do như trên, Switch ở các side khác nhau trong fabric không được kết nối chéo với nhau. Cả 2 side của fabric đều được giữ riêng biệt. Điều này khác với cách làm việc của Ethernet, nơi mà chúng ta thường kết nối các Switch lại với nhau.

Trong mạng Fibre Channel, chúng tôi có 2 fabrics. Các máy chủ bao gồm cả máy chủ lưu trữ được kết nối với 2 fabric là Fabric A và Fabric B. Nhưng các Switch của các fabric thì không được kết nối với nhau. Các Switch sẽ riêng rẽ cho 2 fabric.

Trong ví dụ minh họa dưới đây, Server 1 sẽ có 2 port HBA để đảm bảo tính dự phòng. Port đầu tiên sẽ kết nối đến Fabric A, port thứ 2 sẽ kết nối đến Fabric B.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-01-768x443.jpg">

Trong ví dụ trên, 2 fabric được giữ riêng rẽ, ngăn cách với nhau bởi dấu gạch đỏ ở giữa. Host sẽ được kết nối đến 2 fabric nhưng 2 fabric đó là riêng rẽ với nhau. Điều này đồng nghĩa với việc, nếu cấu hình của một trong 2 fabric bị lỗi, fabric còn lại vẫn sẽ hoạt động bình thường. Host có thể mất kết nối với fabric A nhưng vẫn còn kết nối với fabric B.

Trong trường hợp chúng ra có 2 Controller đểm đảm bảo tính dự phòng cho hệ thống lưu trữ, mô hình mạng sẽ như sau:

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-02-768x505.jpg">

Cũng giống như mô hình trước, fabric A và fabric B được giữ riêng rẽ với nhau, Server 1 và Server 2 kết nối với cả 2 fabric. Ở trên cùng của mô hình, tôi có 2 Controller của hệ thống lưu trữ được tách ra riêng rẽ để đảm bảo tính dự phòng. Controller lưu trữ được kết nối với 2 fabric.

Trên một Switch tại fabric A, cấu hình zone bao gồm fcalias S1-A (HBA port của Server 1 kết nối tới fabric A), fcalias Controller 1-A (HBA port của Controller 1 kết nối tới mạng fabric A) và fcalias Controller 2-A (HBA port của Controller 2 kết nối tới mạng fabric A). Server 1 có thể kết nối với 2 bộ lưu trữ khác nhau.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-03-768x427.jpg">

Tương tự như với Fabric A. Cấu hình Zone cho Server 2 bao gồm fcalias S2-A (HBA port của Server 2 kết nối tới fabric A), fcalias Controller 1-A (HBA port của Controller 1 kết nối tới mạng fabric A) và fcalias Controller 2-A (HBA port của Controller 2 kết nối tới mạng fabric A). Lưu ý rằng, các Server 2 zone bao gồm các port HBA tương tự trên các Controller mà Server 1 đang kết nối vào, chỉ là thay đổi Server.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-04-768x424.jpg">

Sau đó, tôi tạo các Server thành 1 Zone đặt tên là Zoneset-A, bao gồm Server 1 và Server 2. Để tiết kiệm thời gian, tôi sẽ cấu hình trên 1 trong 2 Switch trong fabric A, sau đó nó sẽ truyền bá tới các Switch khác trong cùng fabric A.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-05-768x429.jpg">

Tôi cũng cần cấu hình Switch trong fabric. Tôi cấu hình Zone cho Server 1 gồmfcalias S1-B (HBA của Server kết nối vào fabric B), fcalias Controller 1-B (HBA port của Controller 1 kết nối vào fabric B) và fcalias Controller 2-A (HBA port của Controller 2 kết nối vào fabric B).

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-06-768x426.jpg">

Tiếp theo cũng cần cấu hình Zone cho Server 2. Đặt tên zone là Server 2, bao gồm fcalias S2-B (port HBA của Server 2 kết nối vào fabric B), fcalias Controller 1-B (port HBA của Controller 1 kết nối vào fabric B) và fcalias Controller 2-A (HBA port của Controller 2 kết nối vào fabric B)

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-07-768x409.jpg">


Sau đó, tôi tạo các Server thành 1 Zone đặt tên là Zoneset-B, bao gồm Server 1 và Server 2. Để tiết kiệm thời gian, tôi sẽ cấu hình trên 1 trong 2 Switch trong fabric B, sau đó nó sẽ truyền bá tới các Switch khác trong cùng fabric B.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/FC-08-768x399.jpg">


