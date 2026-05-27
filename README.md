# Giới thiệu đề tài
Trong kỷ nguyên số hóa, thương mại điện tử tại Việt Nam, đặc biệt là ngành hàng Mỹ phẩm , đang chứng kiến sự bùng nổ về mặt dữ liệu phản hồi từ người tiêu dùng. Việc thấu hiểu nhu cầu, tâm lý khách hàng thông qua các văn bản đánh giá không chỉ giúp các thương hiệu tối ưu hóa sản phẩm mà còn hỗ trợ nền tảng Thương mại điện tử xây dựng hệ thống khuyến nghị mang tính cá nhân hóa.
Tuy nhiên, hiện nay các mô hình học máy hoàn toàn bất lực trước nguồn nguyên liệu đầu vào bị ô nhiễm, nhiễu loạn hoặc không đồng bộ. Vì vậy, mục tiêu cốt lõi của đồ án này là tạo ra một bộ dữ liệu chuẩn (Benchmark Dataset).
# Cấu trúc dự án
- data/raw/: Chứa file dữ liệu gốc chưa qua xử lí(lazada_review.json, page_new.csv)
- data/processed/: Chứa file dữ liệu đã qua tiền xử lí
- Notebook: Chứa file jupyter notebook thực hiện toàn bộ quá trình ETL và phân tích
- requirements: Các thư viện sử dụng
# Hướng dẫn chạy code
## thu thập dữ liệu
## Tiền xử lí dữ liệu
- Bước 1:Thiết lập môi trường
- Bước 2: Chuẩn bị dữ liệu
- Bước 3: Chạy các quy trình được tích hợp trong file DS108_ĐA_(1) (3).ipynb
  Data ingestion: Load dữ liệu thô ban đầu
  Preprocessing: Chạy các cell code là sạch văn bản, xử lí dữ liệu mất và chuẩn hóa đặt trưng
  Merging: Chạy cell merge để gán nhãn review_score cho bảng sản phẩm
  Benchmarking: Chạy mô hình Random Forest để tạo bảng kết quả
- Bước 4: Kiểm tra các file dữ liệu đầu ra và kết quả F1-score
