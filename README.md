# README — EV Delivery Dataset

## 1. Mô tả 

Dataset này được xây dựng cho bài toán **giao hàng bằng xe điện (EV Delivery / EV Routing)**.  
Dữ liệu cuối cùng mô tả các tuyến giao hàng của xe điện, gồm quãng đường, năng lượng tiêu thụ, SOC, sạc, traffic, số điểm giao và CO₂ reduction.

Dataset được tạo từ:

- 3 file EVRPTW benchmark: `c101C10.txt`, `r102C10.txt`, `rc102C10.txt`
- Traffic dataset Kaggle TP.HCM: `train.csv`
- Mức tiêu thụ điện tham khảo VinFast
- Giá sạc công cộng VinFast

---

## 2. Nguồn dữ liệu

### EVRPTW benchmark

Ba file EVRPTW cung cấp dữ liệu gốc về:

- Depot / kho
- Customer / điểm giao hàng
- Charging station / trạm sạc
- Tọa độ `x`, `y`
- Demand
- Time window
- Service time
- Thông số xe: `Q`, `C`, `r`, `g`, `v`
Q	Vehicle fuel tank capacity	Dung lượng pin/năng lượng tối đa của xe
C	Vehicle load capacity	Tải trọng tối đa xe chở được
r	Fuel consumption rate	Mức tiêu hao năng lượng khi xe chạy
g	Inverse refueling rate	Hệ số thời gian sạc
v	Average velocity	Vận tốc trung bình

Ký hiệu trong file:

| Ký hiệu | Ý nghĩa |
|---|---|
| `d` | depot / kho |
| `c` | customer / điểm giao |
| `f` | fuel/recharging station / trạm sạc |
| `D0` | depot |
| `C...` | customer |
| `S...` | charging station |

### Traffic dataset

Traffic được lấy từ `train.csv`.  
Một ngày được chia thành 48 khung 30 phút:

```text
period_0_00 = 00:00–00:29
period_0_30 = 00:30–00:59
...
period_23_30 = 23:30–23:59
```

Traffic được quy đổi từ LOS:

| LOS | Traffic level |
|---|---|
| A, B | Light |
| C, D | Medium |
| E, F | Heavy |

---

## 3. Cấu trúc file Excel

| Sheet | Nội dung |
|---|---|
| `Summary` | Tổng hợp kết quả theo từng file nguồn |
| `Final_Dataset` | Dataset chính, mỗi dòng là một route |
| `Route_Steps` | Chi tiết từng chặng trong route |
| `Raw_Nodes` | Dữ liệu điểm gốc từ EVRPTW |
| `Assumptions` | Giả định và công thức tính |
| `Traffic_Source` | Traffic được rút từ `train.csv` |
| `VinFast_Source_Rates` | Mức tiêu thụ điện tham khảo |
| `Unit_Conversion` | Quy đổi đơn vị sử dụng trong dataset |

---

## 4. Các khái niệm chính

| Thuật ngữ | Chú thích |
|---|---|
| `Instance ID` | Tên file nguồn, ví dụ `c101C10` |
| `Vehicle ID` | Mã xe điện được gán cho route |
| `Route ID` | Mã tuyến giao hàng |
| `StringID` | Mã điểm trong dữ liệu, ví dụ `D0`, `C44`, `S15` |
| `SOC` | State of Charge, phần trăm pin còn lại |
| `Total demand` | Tổng lượng hàng cần giao trong một route |
| `Traffic period` | Khung traffic 30 phút tương ứng với thời gian bắt đầu route |
| `Traffic LOS` | Level of Service từ traffic dataset |
| `Traffic level` | Mức traffic sau khi quy đổi: Light, Medium, Heavy |
| `CO₂ reduction` | Lượng CO₂ giảm ước tính khi dùng EV thay xe xăng/diesel |

---

## 5. Công thức sử dụng

### Khoảng cách

```text
distance = sqrt((x2 - x1)^2 + (y2 - y1)^2)
```

### Tổng quãng đường route

```text
Total route distance = tổng các segment distance trong route
```

### Năng lượng tiêu thụ

Dataset không dùng `1 km = 1 kWh`.  
Năng lượng được tính lại theo mức tiêu thụ tham khảo VinFast:

```text
Energy consumption = Total route distance × 0.20 kWh/km
```

### SOC

```text
SOC after route = Battery remaining / Battery capacity × 100
```

### Chi phí sạc

```text
Charging cost = Charging energy × 3,858 VND/kWh
```

### CO₂ reduction

```text
CO₂ reduction = Total route distance × 0.248548
```

---

## 6. Giả định chính

| Giả định | Ghi chú |
|---|---|
| 1 Euclidean distance unit ≈ 1 km | Dùng để quy đổi tọa độ benchmark sang khoảng cách dễ hiểu |
| EV consumption rate = 0.20 kWh/km | Dựa trên mức tiêu thụ tham khảo VinFast VF 8 |
| Charging price = 3,858 VND/kWh | Giá sạc công cộng VinFast |
| Traffic lấy từ LOS trong `train.csv` | A/B = Light, C/D = Medium, E/F = Heavy |
| CO₂ reduction là estimated | Không phải số đo phát thải thực tế |
| Routes are heuristic | Tuyến được dựng thủ công/hợp lý, chưa phải kết quả tối ưu solver |

---

## 7. Ghi chú quan trọng

- `Raw_Nodes` là dữ liệu gốc của các điểm, chưa phải dataset cuối.
- `Route_Steps` dùng để kiểm tra từng đoạn xe đi, pin trước/sau và sạc.
- `Final_Dataset` là bảng chính để phân tích hoặc nộp.
- `Route distance` trong `Final_Dataset` là tổng quãng đường của một route.
- `Total distance` trong `Summary` là tổng quãng đường của nhiều route trong cùng instance.
- Nếu có dòng `No records for period...`, nghĩa là traffic dataset thiếu khung giờ đó nên dùng period gần nhất.
- Các route hiện tại là heuristic, không khẳng định là tuyến tối ưu nhất.

---

## 8. Mô tả ngắn dùng trong báo cáo

Dataset được xây dựng từ ba file EVRPTW benchmark đại diện cho ba kiểu phân bố khách hàng: clustered, random và random-clustered. Dữ liệu node gốc bao gồm depot, customer, charging station, demand, time window và service time. Các tuyến giao hàng được xây dựng bằng phương pháp heuristic. Dữ liệu traffic TP.HCM được ghép theo khung 30 phút từ `train.csv` dựa trên LOS. Năng lượng tiêu thụ được ước tính theo mức 0.20 kWh/km dựa trên dữ liệu tham khảo VinFast, chi phí sạc được tính theo giá 3,858 VND/kWh, và CO₂ reduction được ước tính theo hệ số phát thải tránh được khi dùng xe điện.
