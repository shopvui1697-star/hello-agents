Ở đây chúng tôi giới thiệu Docker trong cách cài đặt cục bộ được sử dụng trong dự án, vì cách này ổn định nhất và thuận lợi nhất cho việc tiếp tục khám phá cách sử dụng n8n.

Trước tiên chúng ta vào trang chủ của docker: [Docker: Accelerated Container Application Development](https://www.docker.com/)

Chọn thiết bị đầu cuối của bạn để tải về, ở đây lấy Windows làm ví dụ minh họa.

![image-20250912025341155](./N8N_INSTALL_GUIDE/image-20250912025341272.png)

Sau khi tải về xong, bạn có thể chuyển đổi đường dẫn lưu trữ ổ đĩa, vì image (ảnh máy) thường rất lớn, hãy cố gắng đừng lưu vào ổ C.

![image-20250912032540657](./N8N_INSTALL_GUIDE/image-20250912032540657.png)

Sau đó mở dòng lệnh của bạn, nhập lệnh sau để kéo (pull) n8n về

```
docker volume create n8n_data
docker run -d --restart unless-stopped --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```

Bây giờ chúng ta đã có thể thấy n8n đang chạy trong docker rồi

![image-20250912033251997](./N8N_INSTALL_GUIDE/image-20250912033251997.png)

Nhấp vào 5678:5678 có thể vào giao diện khởi động của n8n.

![image-20250912033341666](./N8N_INSTALL_GUIDE/image-20250912033341666.png)

Sau khi vào trang, bạn có thể thấy nút mở dự án mới

![image-20250912034040656](./N8N_INSTALL_GUIDE/image-20250912034040656.png)

Có ba chức năng chính được sử dụng
![image-20250912234709064](./N8N_INSTALL_GUIDE/image-20250912234709064.png)

Sau khi mở nút thêm nút (node) mới, bạn có thể tìm kiếm node hoặc chọn node mình cần để thêm vào~

![image-20250912234748845](./N8N_INSTALL_GUIDE/image-20250912234748845.png)
