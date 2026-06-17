# LazSenti-VN: Thu thập & Tiền xử lý Dữ liệu Đánh giá Sản phẩm Mỹ phẩm trên Lazada

## Giới thiệu đề tài

Trong kỷ nguyên số hóa, thương mại điện tử tại Việt Nam, đặc biệt là ngành hàng Mỹ phẩm, đang chứng kiến sự bùng nổ về mặt dữ liệu phản hồi từ người tiêu dùng. Việc thấu hiểu nhu cầu, tâm lý khách hàng thông qua các văn bản đánh giá không chỉ giúp các thương hiệu tối ưu hóa sản phẩm mà còn hỗ trợ nền tảng thương mại điện tử xây dựng hệ thống khuyến nghị mang tính cá nhân hóa.

Tuy nhiên, các mô hình học máy hiện nay thường bất lực trước nguồn dữ liệu đầu vào bị nhiễu, thiếu sót hoặc không đồng bộ — hiện tượng "Garbage In, Garbage Out". Vì vậy, mục tiêu cốt lõi của đồ án này **không phải là tối ưu hóa thuật toán**, mà là xây dựng một hệ thống tiền xử lý dữ liệu (Data Pipeline) mạnh mẽ, minh bạch và có khả năng tái lập hoàn toàn, từ đó tạo ra một bộ dữ liệu chuẩn (Benchmark Dataset) — **LazSenti-VN** — phục vụ bài toán Phân tích Cảm xúc (Sentiment Analysis) trên đánh giá sản phẩm.

## Cấu trúc dự án

```
├── data/
│   ├── raw/                          # Dữ liệu gốc chưa qua xử lý
│   │   ├── pages-new.csv             # Thông tin sản phẩm (2.152 SP, 14 cột)
│   │   └── lazada_reviews.json       # Reviews thô dạng JSON lồng ghép (~32.136 reviews)
│   └── processed/                    # Dữ liệu đã qua tiền xử lý (output cuối)
│       ├── products_cleaned_train.csv
│       ├── products_cleaned_val.csv
│       ├── reviews_cleaned_train.csv
│       └── reviews_cleaned_val.csv
├── notebook/
│   └── DS108_ĐA.ipynb                # Toàn bộ pipeline ETL, EDA, Benchmark
├── requirements.txt                   # Thư viện sử dụng
└── README.md
```

## Hướng dẫn chạy code

### 1. Thu thập dữ liệu

#### Cách hoạt động

Quá trình thu thập được chia thành 2 giai đoạn liên tiếp, sử dụng kỹ thuật giả lập TLS/JA3 fingerprint của trình duyệt Chrome (thông qua `curl_cffi`) để vượt qua cơ chế anti-bot của Lazada.

**Giai đoạn 1 — Thu thập sản phẩm:**
1. Gửi request lặp qua hệ thống phân trang (pagination) của từng gian hàng thương hiệu (Paula's Choice, Sulwhasoo, Shiseido, Obagi, ...).
2. Trích xuất thông tin sản phẩm: `product_id`, `product_name`, `brand_name`, `seller_name`, `price_current`, `price_original`, `sold_count`, `review_count`, `in_stock`, `rating_score`, `location`, ...
3. Lưu kết quả vào `pages-new.csv`.

**Giai đoạn 2 — Thu thập đánh giá:**
1. Đọc danh sách `product_id` từ `pages-new.csv` đã thu thập ở Giai đoạn 1.
2. Kiểm tra tiến độ cũ trong `lazada_reviews.json` (nếu có) để bỏ qua sản phẩm đã thu thập — tránh mất dữ liệu khi chương trình bị ngắt giữa chừng.
3. Với mỗi sản phẩm, quét vòng lặp qua từng mức đánh giá (1 đến 5 sao), lật từng trang bình luận (20 bình luận/trang) cho đến khi đạt tối đa 100 bình luận/mức sao (biến `MAX_PER_STAR`).
4. Trích xuất các trường: `user_id`, `product_id`, `review_time`, `review_helpfulness`, `review_score`, `review_text`.
5. Lưu ngay vào `lazada_reviews.json` sau khi hoàn thành mỗi sản phẩm.
6. Nếu hệ thống trả về HTML (bắt đăng nhập/xác minh) thay vì JSON, code báo lỗi "BỊ CHẶN" và dừng an toàn để thay Cookie mới.

#### Hướng dẫn sử dụng

**Bước 1 — Cài đặt thư viện:**
```bash
pip install requests pandas curl_cffi
```

**Bước 2 — Chuẩn bị file đầu vào:**
Đặt file `pages-new.csv` cùng thư mục với script thu thập reviews. File bắt buộc phải có cột `product_id`.

```
product_id
279632605
123456789
```

**Bước 3 — Lấy và cập nhật Cookie:**
Do Lazada có cơ chế chống crawl mạnh (phân tích TLS/JA3 fingerprint), cần cập nhật Cookie định kỳ:
1. Mở trình duyệt (Chrome/Edge), vào trang sản phẩm bất kỳ, cuộn xuống phần đánh giá.
2. Nhấn `F12` mở Developer Tools → tab **Network**.
3. Bấm sang các trang đánh giá tiếp theo (trang 2, 3, ...).
4. Tìm request bắt đầu bằng `getReviewList...`, bấm vào đó.
5. Cuộn xuống **Request Headers**, tìm dòng `Cookie:`.
6. Copy toàn bộ giá trị sau `Cookie:` và dán vào biến `COOKIES` trong script.

**Bước 4 — Chạy script:**
```bash
python reviews.py
```

### 2. Tiền xử lý dữ liệu

Toàn bộ pipeline được thực hiện trong notebook `DS108_ĐA.ipynb`, tuân thủ **nguyên tắc vàng: Split trước, mọi tính toán thống kê chỉ fit() trên Train** — nhằm triệt tiêu hoàn toàn Data Leakage.

**Bước 1 — Thiết lập môi trường:**
```bash
pip install -r requirements.txt
```
Bao gồm: `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `emoji`, `google-generativeai` (cho phần kiểm định AI).

**Bước 2 — Chuẩn bị dữ liệu:**
Đặt `pages-new.csv` và `lazada_reviews.json` vào thư mục `data/raw/`.

**Bước 3 — Chạy notebook theo đúng thứ tự các cell:**

| # | Giai đoạn | Nội dung thực hiện |
|---|---|---|
| 1 | **Data Ingestion** | Load `pages-new.csv` và `lazada_reviews.json`; xử lý cấu trúc JSON lồng ghép (`byStars`) bằng `re.sub` + `ast.literal_eval`; hợp nhất 2 nguồn theo khóa `product_id` |
| 2 | **Data Cleaning** | Loại bỏ bản ghi thiếu `product_id`; đối soát `price_original` (gán = `price_current` nếu thiếu/≤0); tính `discount_percentage`; chuẩn hóa `in_stock` → 0/1; chuẩn hóa `product_name` (lowercase, strip); loại `user_id = 0`; xóa trùng lặp |
| 3 | **Text Cleaning** | Hàm `clean_all_text()`: lowercase, xóa URL, khử emoji (`emoji.demojize`), chuẩn hóa viết tắt tiếng Việt (`vnm_abbrev_dict`), xóa ký tự lặp; hàm `clean_review_time()` chuẩn hóa ngày về `dd/mm/yyyy` |
| 4 | **Data Splitting** | Chia Train/Val theo `unique_product_ids` (80/20, `random_state=42`) **trước khi** tính bất kỳ thống kê nào — đảm bảo Zero Data Leakage |
| 5 | **Feature Engineering** | Kiểm tra skewness 3 biến số (`price_current`, `sold_count`, `review_count`); áp dụng `np.log1p` cho biến lệch nặng (`\|skewness\| > 1.0`); fit `MinMaxScaler` chỉ trên Train rồi `transform()` cả hai tập; tính `star_1~5_count` **chỉ từ `df_rev_train`** rồi merge vào bảng sản phẩm; mã hóa `label = review_score - 1` |
| 6 | **Xử lý Null Text** | Thêm cột `has_text` (0/1) đánh dấu trước khi điền; điền chuỗi rỗng cho `review_text` null (không xóa dòng, để bảo toàn phân phối nhãn) |
| 7 | **Downsampling** | Cân bằng nhãn tập Train với ngưỡng tối đa 5.000 mẫu/nhãn; nhãn thiểu số (2, 3 sao) giữ nguyên toàn bộ; tập Val giữ nguyên phân phối tự nhiên |
| 8 | **EDA** | Vẽ và phân tích các biểu đồ: phân phối trước/sau Log Transform, Correlation Heatmap, Violin plot (giá theo tồn kho), Scatter (discount vs rating), tỉ lệ text theo sao, phân tích cấu trúc thị trường (Gini, Lorenz Curve), phân tích top vấn đề trong review 1–2 sao |
| 9 | **Benchmarking** | Huấn luyện `RandomForestClassifier` trên tập Train đã xử lý; đánh giá qua Classification Report và Confusion Matrix |
| 10 | **AI-Annotated Audit** | Lấy mẫu từ tập Val, gửi qua Gemini API (Few-shot Prompting) để gán nhãn độc lập; tính Cohen's Kappa giữa nhãn gốc và nhãn AI |
| 11 | **Sanity Checks** | Chạy `run_production_sanity_checks()`: kiểm tra không Data Leakage, miền giá trị `[0,1]`, `product_id` unique, không null sau xử lý, `has_text` hợp lệ |
| 12 | **Export** | Xuất 4 file CSV cuối vào `data/processed/`: `products_cleaned_train.csv`, `products_cleaned_val.csv`, `reviews_cleaned_train.csv`, `reviews_cleaned_val.csv` |

**Bước 4 — Kiểm tra kết quả:**
- Xác nhận 4 file CSV đầu ra trong `data/processed/` không có lỗi (chạy lại `run_production_sanity_checks()` nếu cần).
- Kết quả Benchmark tham chiếu: **Accuracy = 0.84**, **Weighted F1-Score = 0.81**.
- Kết quả kiểm định AI tham chiếu: **Cohen's Kappa = −0.0297** (xem giải thích nguyên nhân trong báo cáo, mục Thảo luận).

## Mô tả các cột dữ liệu đầu ra

### Bảng Products (`products_cleaned_train.csv` / `products_cleaned_val.csv`)

| Cột | Kiểu | Ý nghĩa |
|---|---|---|
| `product_id` | int | Khóa định danh duy nhất của sản phẩm |
| `product_name` | string | Tên sản phẩm (đã lowercase, strip) |
| `brand_id`, `brand_name` | int, string | Mã và tên thương hiệu |
| `seller_id`, `seller_name` | int, string | Mã và tên người bán |
| `price_current`, `price_original` | float | Giá hiện tại / giá gốc (VND) |
| `sold_count`, `review_count` | int | Số lượng đã bán / số lượt đánh giá |
| `in_stock` | int {0,1} | Tình trạng tồn kho |
| `rating_score` | float | Điểm đánh giá trung bình |
| `location` | string | Tỉnh/thành của người bán |
| `discount_percentage` | float | Tỉ lệ giảm giá = (giá gốc − giá hiện tại) / giá gốc |
| `price_current_processed`, `sold_count_processed`, `review_count_processed` | float [0,1] | Sau Log Transform + MinMaxScaler |
| `star_1_count` ... `star_5_count` | int | Số lượng đánh giá theo từng mức sao (tính chỉ từ Train) |

### Bảng Reviews (`reviews_cleaned_train.csv` / `reviews_cleaned_val.csv`)

| Cột | Kiểu | Ý nghĩa |
|---|---|---|
| `product_id` | int | Khóa liên kết với bảng Products |
| `user_id` | int | Mã định danh người dùng (ẩn danh) |
| `review_time` | string | Thời gian đánh giá, định dạng `dd/mm/yyyy` |
| `review_score` | int {1..5} | Điểm sao gốc do người dùng chấm |
| `review_helpfulness` | int | Số lượt hữu ích |
| `review_text` | string | Văn bản đánh giá gốc |
| `review_text_cleaned` | string | Văn bản đã làm sạch |
| `label` | int {0..4} | Nhãn mã hóa = `review_score - 1` |

> **Lưu ý kỹ thuật:** Cột `star_1~5_count` trong tập Val có thể bằng 0 ở một số sản phẩm — đây là hành vi đúng thiết kế (sản phẩm đó chưa có review nào trong tập Train), không phải lỗi dữ liệu.

## Tài liệu liên quan

- **Codebook / Data Dictionary**: chi tiết kiểu dữ liệu và miền giá trị từng cột — xem Phụ lục báo cáo.
- **Datasheets for Datasets** (theo chuẩn Gebru et al., 2021): Motivation, Composition, Collection Process, Preprocessing, Uses, Distribution, Maintenance, Ethical Considerations — xem Phụ lục B báo cáo.

## Giấy phép sử dụng

Bộ dữ liệu chỉ phục vụ mục đích nghiên cứu học thuật phi lợi nhuận (Academic Use Only). Không sử dụng để xác định danh tính cá nhân hoặc tái phân phối thương mại.
