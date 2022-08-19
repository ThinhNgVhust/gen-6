# Làm thế nào để mở rộng qui mô trang web? Ví dụ đáp ứng nhu cầu sử dụng cho khoảng vài triệu người dùng?

## Scalability

---

Sau khi viết mã cho ứng dụng web, cuối cùng thì chúng ta vẫn phải triển khai nó trên một máy chủ vật lý nào đó **servers** để mọi người từ mọi nơi có thể truy cập vào được trang web. Ngoài ra chúng ta có thể thuê máy chủ của các công ty công nghệ lớn như GG, Amazone... (hay còn gọi là dịch vụ đám mây)  

Một vài ưu và nhược điểm khi lựa chọn các phương án trên:   
* **Khả năng tùy biến**: Nếu sở hữu máy chủ riêng thì giúp khả năng tùy biến của chúng ta trở nên tốt hơn.   
* **Sự đơn giản hóa**: Chuyện sẽ trở nên đơn giản hơn rất nhiều nếu ta sử dụng dịch vụ đám mây, chúng ta không cần phải cài đặt quá nhiều.   
* **Chi phí**: Tùy vào qui mô công ty và mục đích sử dụng. Chi phí thuê dịch vụ đám mây sẽ đắt hơn sở hữu máy chủ, vì thực chất bên cung cấp dịch vụ cũng đang có những máy chủ tương tự và họ cho thuê cần phải tạo ra lợi nhuận. Tuy nhiên nếu doanh nghiệp nhỏ nếu chi phí sở hữu và maintain, thuê chuyên viên sẽ tốn kém. Không có 1 phương án tối ưu cho mọi hoàn cảnh. 
* Khả năng mở rộng **Scalability**: Mở rộng sẽ dễ dàng hơn khi sử dụng dịch vụ đám mây. 

Khi user gửi 1 request tới sever, server sẽ phải trả lại 1 response. Nhưng trên thực tế, tại 1 lúc gần như đồng thời sever sẽ phải nhận rất nhiều request cùng 1 lúc như dưới đây: 
![image](imgs/imcoming_request.JPG)  
---
Đây chính là vấn đề mà ta sẽ gặp phải: 
Nếu sử dụng 1 sever đơn lẻ duy nhất, sẽ có giới hạn số lượng request tối đa mà máy chủ có thể hanld được mà không làm crash phần mềm. (Có thể sử dụng **benchmarking** để kiểm tra giới hạn này) nếu lượng request vượt trên giới hạn này thì có thể có 2 cách xử lý: 
1. **Vertical Scaling**: Thay máy chủ của bạn bằng 1 máy chủ khác có cấu hình tốt hơn. Cách này có giới hạn, nếu lượng request gửi đến rất lớn thì có thể máy chủ với cấu hình tốt nhất cũng không có khả năng đáp ứng được.    
2. **Horizontal Scaling**: Sử dụng thêm nhiều máy chủ nữa để chia ra handle các request.   

## Load Balancing   

Khi sử dụng phương pháp thứ 2, sẽ có vấn đề phát sinh là: **Ta phân chia nhiệm vụ xử lý các request như nào?**. Câu trả lời ở đây là sử dụng [Load Balancer](https://www.nginx.com/resources/glossary/load-balancing/). Một thiết bị phần cứng có trách nhiệm tiếp nhận các imcoming requests và phân phối các request này đến các sever.
[image](imgs/load_balancer.JPG)   

Có một vài cách để Load balancer phân chia nhiệm request tới các server:
* **Random**: LB sẽ chọn ngẫu nhiên một server để xử lý 1 request tới.
* **Round-Robin**: Chia đều mỗi request tới cho mỗi server theo thứ tự và quay vòng. Ví dụ có 3 server: A,B,C. Request 1, A xử lý. 2, B xử lý. 3, C xử lý và 4 lại là A xử lý.   
* **Fewest Connection**: LB sẽ tìm kiếm server nào hiện tại đang có ít request đang xử lý nhất và chỉ định server đó xử lý request. (Nghe kiểu giống heap quá.) Như vậy sẽ mất thời gian để LB tính toán tìm ra server nào.   
Đọc đến đây tôi cảm thấy LB cứ như cô giáo, và mỗi học sinh là 1 sever tương ứng request ở đây là gọi trả bài. Cô có thể chọn ngẫu nhiên(**Robin**), chọn để thằng nào cũng phải lên bảng số lần như nhau(**round-robin**), hoặc chọn thằng nào ít được lên bảng nhất.(**Fewest**) :))   

Không có phương án nào tốt nhất ở mọi hoàn cảnh. Tùy hoàn cảnh mà chọn phương pháp!   
-------
Một vấn đề nữa đó chính là **session**. Khi user gửi request, LB sẽ chỉ định 1 server(tạm gọi là sever A) nào đó để handle request này, session này được lưu trữ ở server A, nhưng user đó lại gửi 1 request mới, mà không muốn phải nhập lại thông tin trong trường hợp LB giao rq này cho sever khác không phải A???? Dưới đây là 1 số giải pháp:     
1. **Sticky Sessions**: Khi user truy cập vào 1 trang, LB sẽ nhớ sever nào xử lý user này. Lần tới khi user truy cập lại, LB sẽ tìm đúng sever đã xử lý cho user đó. Một vấn đề có thể gặp phải là có quá nhiều user cùng đc xử lý tại 1 sever -> Sever có thể bị crash.   
2. **Database Sessions**':Sessions sẽ được lưu ở database.  Ok, LB cứ gửi ngẫu nhiên xuống cho các sever mà không cần quan tâm sever nào làm việc với user trước đó. 1 vấn đề mà ta có thể gặp phải là tốn thời gian để tính toán đọc, ghi dữ liệu.
3. **Client-Side Sessions**: Thay vì lưu thông tin session ở phía sever. Ta lưu thông tin ở phía client thông qua cookie.  

-------


## Autoscaling   
Là việc tăng, giảm số lượng server để phục vụ các request. 
### Sever failure.   
#### Single Point Failure 
Là việc mà chỉ vì một phần cứng bị hỏng dẫn tới hoảng toàn bộ hệ thống. Ví dụ nếu scaling theo Vertical- thay server có cấu hình tốt hơn. Thế nhưng khi server crash thì có thể ngưng toàn bộ hệ thống. Ngược lại Horizontal Scaling nếu 1 sever hỏng thì vẫn còn các sever còn lại đón nhận request từ LB. Ngoài ra LB có thể phát hiện server nào bị hỏng thông qua việc nhận tín hiệu **heartbeat** từ server. 