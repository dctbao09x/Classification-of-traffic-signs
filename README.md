
# Traffic Sign Detection and Classification  
---

## 1. Giới thiệu bài toán

Trong bối cảnh giao thông ngày càng phức tạp, việc đảm bảo an toàn cho người tham gia giao thông đòi hỏi các hệ thống hỗ trợ có khả năng nhận diện nhanh và chính xác biển báo giao thông. Quan sát bằng mắt thường của con người bị hạn chế bởi tốc độ di chuyển, điều kiện thời tiết, ánh sáng và mức độ tập trung của tài xế.  

Bài toán **Traffic Sign Detection and Classification** sử dụng xử lý ảnh và trí tuệ nhân tạo nhằm tự động tìm và phân loại biển báo giao thông, góp phần nâng cao độ an toàn và hỗ trợ ra quyết định trong các hệ thống giao thông thông minh.

### Hai bước chính
1. **Detection (Tìm kiếm / định vị)**  
   Xác định vị trí biển báo trong ảnh hoặc khung hình video, phân biệt với nền (cây cối, xe cộ, công trình).

2. **Classification (Phân loại)**  
   Nhận diện loại biển báo (cấm, chỉ dẫn, nguy hiểm, hiệu lệnh, …) để phục vụ cảnh báo hoặc điều khiển.

### Ứng dụng
- **ADAS (Advanced Driver Assistance Systems)**  
- **Xe tự hành (Autonomous Vehicles)**  
- **Giám sát & quản lý giao thông thông minh**


## 2. Phương pháp thực hiện
Pipeline được xây dựng gồm 3 giai đoạn chính:

### (a) Tiền xử lý & xác định đối tượng (Localize)

- Dữ liệu đầu vào:
  - Ảnh màu `.png`
  - Annotation `.xml` theo chuẩn **Pascal VOC**
- Trích xuất bounding box `(xmin, ymin, xmax, ymax)` từ file XML.
- Crop ảnh biển báo từ ảnh gốc.
- Chuyển đổi không gian màu **RGB → HSV** để làm nổi bật màu đặc trưng (đỏ, xanh).
- Áp dụng:
  - Thresholding theo kênh màu
  - Morphological Operations để loại nhiễu
- Sử dụng **Contour Detection** để xác định chính xác vùng biển báo.
- Lưu ảnh con đã xử lý.


### (b) Trích xuất đặc trưng (Feature Extraction)

Các đặc trưng được sử dụng:

1. **HOG (Histogram of Oriented Gradients)**  
   - Mô tả hình dạng, biên và cấu trúc hình học.
2. **Histogram màu**  
   - Mô tả phân bố màu sắc, hỗ trợ phân biệt các nhóm biển báo.
3. **Ảnh xám vector hóa**  
   - Phiên bản baseline để so sánh.


### (c) Phân loại (Classification)

Hai mô hình học máy được triển khai:

1. **Softmax Regression**
   - Mô hình tuyến tính đa lớp
   - Vai trò: baseline

2. **SVM (Support Vector Machine)**
   - Sử dụng kernel để xử lý dữ liệu phi tuyến
   - Hiệu quả hơn với đặc trưng HOG

**Đầu vào**: vector đặc trưng  
**Đầu ra**: nhãn lớp (speedlimit, no_parking, turn_left, …)


## 3. Thí nghiệm và Kết quả

### 3.1 Thí nghiệm

- **Dataset**: Ảnh giao thông thực + annotation Pascal VOC  
- **Chia dữ liệu**:
  - 70% Training
  - 20% Validation
  - 10% Test

**Quy trình**:
1. Crop biển báo + tiền xử lý HSV, thresholding, contour.
2. Trích xuất HOG & histogram màu.
3. Huấn luyện Softmax và SVM.

**Cấu hình**:
- Resize ảnh về `64×64`
- Thư viện:
  - `OpenCV (cv2)`
  - `scikit-image (HOG)`
  - `scikit-learn (SVM, Logistic Regression)`


### 3.2 Kết quả

#### Softmax Regression
- Training accuracy: ~90–95%
- Validation accuracy: ~15–20%
- Hiện tượng **overfitting**
- Khả năng khái quát kém

#### SVM
- Training accuracy: >95%
- Validation accuracy: ~40–50%
- Đặc trưng **HOG** cho kết quả ổn định hơn histogram màu


## 4. Phân tích mô hình hiện tại

### 4.1 Ưu điểm

- **Chi phí tính toán thấp**  
  Phù hợp hệ thống nhúng, Raspberry Pi, real-time.
- **Cấu trúc rõ ràng, dễ debug**  
  Tách biệt tiền xử lý – trích xuất – phân loại.
- **Ổn định với biến đổi hình học**  
  HOG xử lý tốt xoay, nghiêng, thay đổi kích thước.


### 4.2 Hạn chế

- **Độ bền vững thấp với môi trường thực**
  - Nhạy với ánh sáng
  - Che khuất
  - Nhiễu
- **Phụ thuộc mạnh vào chất lượng HOG**
- **Khó phân biệt các lớp tương đồng về hình dạng**


### 4.3 So sánh với Deep Learning (CNN)

| Tiêu chí | HOG + SVM | CNN |
|--------|----------|-----|
| Trích xuất đặc trưng | Thủ công | Tự động |
| Độ chính xác | Trung bình | Cao |
| Tài nguyên | Thấp | Cao (GPU) |
| Khả năng mở rộng | Hạn chế | Tốt |


## 5. Kết luận & Hướng phát triển

Hệ thống HOG + Softmax/SVM phù hợp cho mục đích học thuật và các ứng dụng tài nguyên hạn chế. Tuy nhiên, để triển khai thực tế trong môi trường giao thông phức tạp, cần chuyển sang các mô hình học sâu.

### 5.1 Hướng phát triển ngắn hạn

- Kết hợp **HOG + Histogram màu**
- **Data Augmentation**:
  - Xoay
  - Lật
  - Thay đổi độ sáng, tương phản
  - Thêm nhiễu


### 5.2 Hướng phát triển dài hạn

- **CNN cho Classification**
  - LeNet, VGG
- **Object Detection end-to-end**
  - YOLO
  - Faster R-CNN
  - SSD  

Cho phép phát hiện + phân loại đồng thời với toạ độ chính xác.


## Công nghệ sử dụng

- Python  
- OpenCV  
- scikit-learn  
- scikit-image  
### 6. Tài Nguyên
Hoàn toàn có sẵn nguồn tài nguyên để thực hành!
=> Vào Switch branches/tags chọn "master"


## Ghi chú

Dự án tập trung vào **pipeline cổ điển** (traditional computer vision + machine learning) nhằm phục vụ mục tiêu học tập và phân tích hệ thống, không tối ưu cho deployment quy mô lớn.

