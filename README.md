# Giới thiệu đề tài
Trong kỷ nguyên số hóa, thương mại điện tử tại Việt Nam, đặc biệt là ngành hàng Mỹ phẩm , đang chứng kiến sự bùng nổ về mặt dữ liệu phản hồi từ người tiêu dùng. Việc thấu hiểu nhu cầu, tâm lý khách hàng thông qua các văn bản đánh giá không chỉ giúp các thương hiệu tối ưu hóa sản phẩm mà còn hỗ trợ nền tảng Thương mại điện tử xây dựng hệ thống khuyến nghị mang tính cá nhân hóa.
Tuy nhiên, hiện nay các mô hình học máy hoàn toàn bất lực trước nguồn nguyên liệu đầu vào bị ô nhiễm, nhiễu loạn hoặc không đồng bộ. Vì vậy, mục tiêu cốt lõi của đồ án này là tạo ra một bộ dữ liệu chuẩn (Benchmark Dataset).
# Cấu trúc dự án
- data/raw/: Chứa file dữ liệu gốc chưa qua xử lí(lazada_review.json, page_new.csv)
- data/processed/: Chứa file dữ liệu đã qua tiền xử lí
- Notebook: Chứa file jupyter notebook thực hiện toàn bộ quá trình ETL và phân tích
- requirements: Các thư viện sử dụng
# Hướng dẫn chạy code
## Thu thập dữ liệu
### ===== Cách hoạt động của file =====
1. Đọc danh sách sản phẩm: Mở file pages-new.csv để lấy danh sách các mã sản phẩm (cột product_id) cần lấy đánh giá.
2. Kiểm tra tiến độ cũ: Đọc file lazada_reviews.json (nếu có) để xem sản phẩm nào đã được lấy dữ liệu rồi thì sẽ tự động bỏ qua. Điều này giúp không bị mất dữ liệu hoặc không phải chạy lại từ đầu nếu chương trình bị ngắt giữa chừng.
3. Thu thập theo từng mức sao: Với mỗi sản phẩm, code sẽ lặp qua các mức đánh giá từ 1 sao đến 5 sao. Nó lật từng trang bình luận (20 bình luận/trang) cho đến khi lấy đủ tối đa 100 bình luận cho mỗi mức sao (giới hạn bởi biến MAX_PER_STAR).
4. Lọc thông tin: Nó không lấy toàn bộ cục dữ liệu khổng lồ của Lazada mà chỉ trích xuất các trường quan trọng: Mã người dùng, mã sản phẩm, thời gian đánh giá, số lượt hữu ích, số sao, và nội dung đánh giá.
5. Lưu trữ: Lưu dữ liệu ngay lập tức vào file lazada_reviews.json sau khi hoàn thành mỗi một sản phẩm.
6. Nhận diện bị chặn: Nếu hệ thống Lazada phát hiện bot và trả về mã HTML (bắt đăng nhập hoặc xác minh) thay vì dữ liệu JSON, code sẽ báo lỗi "BỊ CHẶN" và dừng lại an toàn để bạn thay Cookie mới.
### ===== Hướng dẫn sử dụng =====
- Bước 1: Cài đặt thư viện cần thiết
  Script này sử dụng một số thư viện ngoài.Mở Terminal hoặc Command Prompt và     chạy lệnh sau để cài đặt:
  pip install requests pandas

- Bước 2: Chuẩn bị file đầu vào
  Tạo một file tên là pages-new.csv để cùng thư mục với file reviews.py. File     CSV này bắt buộc phải có một cột tên là product_id chứa các mã số sản phẩm      của Lazada.
  Ví dụ nội dung file CSV:
  product_id
  279632605
  123456789

-Bước 3: Lấy và cập nhật Cookie
  Lazada chống crawl dữ liệu rất mạnh, thế nên phải thay đổi cookies để có       chương trình tiếp tục chạy. Cookie trong code hiện tại chắc chắn đã hết hạn.     Bạn lấy cái mới bằng cách:
   Mở trình duyệt (Chrome/Edge)
   
  Vào trang của một sản phẩm bất kỳ, cuộn xuống phần đánh giá.

  Nhấn phím F12 để mở Developer Tools, chuyển sang tab Network.
  
  Bấm sang các trang đánh giá tiếp theo (trang 2, 3...) trên giao diện trang      web.
  
  Tìm trong tab Network một request có tên bắt đầu bằng getReviewList.... Bấm     vào đó.
  
  Cuộn xuống phần Request Headers, tìm dòng Cookie:.
  
  Copy toàn bộ đoạn text dài dằng dặc đằng sau chữ Cookie: và dán đè vào biến     COOKIES trong file code của bạn.

- Bước 4: Chạy script
  Mở Terminal, di chuyển đến thư mục chứa file và gõ lệnh:
  python reviews.py

## Tiền xử lí dữ liệu
- Bước 1:Thiết lập môi trường

- Bước 2: Chuẩn bị dữ liệu

- Bước 3: Chạy các quy trình được tích hợp trong file DS108_ĐA_(1) (3).ipynb
  Data ingestion: Load dữ liệu thô ban đầu

  Preprocessing: Chạy các cell code là sạch văn bản, xử lí dữ liệu mất và chuẩn hóa đặt trưng
  Merging: Chạy cell merge để gán nhãn review_score cho bảng sản phẩm
  
  Benchmarking: Chạy mô hình Random Forest để tạo bảng kết quả
  
- Bước 4: Kiểm tra các file dữ liệu đầu ra và kết quả F1-score
