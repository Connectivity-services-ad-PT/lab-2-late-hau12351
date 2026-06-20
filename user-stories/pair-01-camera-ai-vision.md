# User Story — Pair 01 Camera Stream → AI Vision

## 1. Mechanism

REST synchronous communication.

Camera Stream gọi trực tiếp AI Vision thông qua HTTP REST API khi phát hiện chuyển động từ camera.

---

## 2. Context

Trong hệ thống Smart Campus, Camera Stream chịu trách nhiệm thu nhận luồng video từ camera giám sát.

Khi phát hiện chuyển động (motion detection), Camera Stream tạo snapshot và gửi metadata của ảnh sang AI Vision để thực hiện nhận diện đối tượng.

AI Vision phân tích ảnh và trả về kết quả nhận diện như người, phương tiện hoặc đối tượng chưa xác định cùng mức độ rủi ro.

---

## 3. Consumer Need

Consumer là Camera Stream.

Camera Stream cần:

* Gửi thông tin ảnh vừa chụp.
* Nhận kết quả nhận diện trong thời gian ngắn.
* Lấy detectionId để truy vấn lại khi cần.
* Nhận confidence và riskLevel để chuyển tiếp sang Core Business Service.
* Có khả năng kiểm tra trạng thái AI Vision trước khi gửi yêu cầu.

---

## 4. Provider Responsibility

Provider là AI Vision.

AI Vision cần:

* Nhận request phân tích ảnh.
* Kiểm tra dữ liệu đầu vào.
* Thực hiện nhận diện đối tượng.
* Trả kết quả chuẩn hóa.
* Cung cấp endpoint kiểm tra sức khỏe hệ thống.
* Cung cấp endpoint tra cứu kết quả nhận diện.
* Cung cấp endpoint thông tin model đang sử dụng.

---

## 5. Main Endpoints

### POST /vision/detect

Camera Stream gửi metadata ảnh để AI Vision xử lý.

Request:

```json
{
  "camera_id": "cam-gate-a",
  "image_url": "http://camera-stream/static/frame001.jpg",
  "timestamp": "2026-05-02T09:10:00Z",
  "correlation_id": "CORR-001"
}
```

Response:

```json
{
  "detectionId": "DET-001",
  "riskLevel": "medium",
  "result": {
    "objectType": "person",
    "confidence": 0.91
  }
}
```

---

### GET /vision/detections/{detectionId}

Truy vấn kết quả nhận diện.

---

### GET /vision/models/info

Lấy thông tin model AI hiện tại.

---

### GET /health

Kiểm tra trạng thái hoạt động của AI Vision.

---

## 6. Business Rules

1. Mỗi request phải có camera_id.
2. Mỗi request phải có timestamp hợp lệ.
3. image_url phải là URL truy cập được.
4. detectionId phải duy nhất.
5. AI Vision phải trả confidence trong khoảng từ 0 đến 1.
6. riskLevel chỉ nhận một trong ba giá trị:

   * low
   * medium
   * high

---

## 7. Error Cases

### Invalid Request

* Thiếu camera_id.
* Thiếu timestamp.
* image_url không hợp lệ.

HTTP Status:

```text
400 Bad Request
```

### Unauthorized

* Thiếu Bearer Token.
* Token không hợp lệ.

HTTP Status:

```text
401 Unauthorized
```

### Not Found

* detectionId không tồn tại.

HTTP Status:

```text
404 Not Found
```

### Internal Error

* AI model lỗi.
* Downstream service lỗi.

HTTP Status:

```text
500 Internal Server Error
```

---

## 8. Negotiation Issues

### Issue 1

Ảnh được gửi dưới dạng URL hay upload file trực tiếp?

Quyết định:

Sử dụng image_url.

Lý do:

Giảm kích thước request và đơn giản hóa tích hợp.

---

### Issue 2

Kết quả trả đồng bộ hay bất đồng bộ?

Quyết định:

Trả đồng bộ.

Lý do:

Camera Stream cần phản hồi nhanh để chuyển tiếp cho Core Business.

---

### Issue 3

Có cần correlation_id không?

Quyết định:

Có.

Lý do:

Hỗ trợ tracing và debugging giữa các service.

---

## 9. Acceptance Criteria

* Camera Stream gọi thành công POST /vision/detect.
* AI Vision trả detectionId hợp lệ.
* GET /vision/detections/{id} hoạt động.
* GET /vision/models/info hoạt động.
* GET /health hoạt động.
* Contract pass Spectral lint.
* Prism mock server chạy thành công.
