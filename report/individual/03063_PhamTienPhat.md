# Báo cáo thực hành Day 3

- Mã học viên theo lớp: 2A202603063
- Ngày / CVAT local: 11–12/11/2026 / CVAT v2.74.1 chạy local bằng Docker
- Công cụ đã dùng:
  - CVAT: Polygon và Brush để vẽ
  - `scripts/inspect_submissions.py` và `scoring/score.py` để tự kiểm.

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh [LMS](https://aithucchien.vinuni.edu.vn) đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Nếu export lỗi, ghi task, dữ liệu đã Save đến đâu và lỗi đã báo coach.

Cả 9 ZIP đều export được và `inspect_submissions.py` báo `OK`. Nhãn ban đầu của mọi task là gợi ý từ tool tự gán nhãn, import vào CVAT, sau đó kiểm và sửa tay. Phần đã sửa tay: ở `medium_instance`, ảnh `000000181542.jpg`, tôi chỉnh lại mask người lái xe máy ở mép phải cho bao cả phần chân (xem mục 3).

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg`, người phụ nữ mặc áo dài đang đi bộ qua vạch kẻ đường, ở giữa-trái ảnh (khoảng x 187–307, y 131–460 trên ảnh 640×600), phía trước chiếc xe máy bên trái. Vẽ tay bằng Polygon (70 điểm) trong CVAT, chưa import gợi ý.
- Class và quy tắc tôi dùng để chọn biên:
  - Class `person`, chỉ vẽ phần nhìn thấy: từ đầu, tóc tới vạt áo dài và bàn chân chạm vạch kẻ.
  - Biên dừng ở mép người, không ăn sang xe máy và người lái phía sau. Phần xe máy bị người che không tính vào mask `person`.
  - Người này là một instance riêng, không gộp với người lái xe phía sau.
- Gợi ý khi vẽ object này: **không dùng**. Person này tôi tự vẽ tay hoàn toàn.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `medium_instance` / `000000181542.jpg` / người lái xe máy ở mép phải ảnh, phần cẳng chân và bàn chân đặt trên gác chân (khoảng x 600–639, y 380–464).
- Lỗi thuộc loại: gộp-tách. Một người bị tách thành hai instance `person`.
- Bằng chứng tôi nhìn thấy:
  - Mask người lái dừng ở hông.
  - Phần chân là một object `person` riêng, có viền riêng.
  - Phần chân và thân nối liền nhau, cùng một người ngồi trên xe.
- Quy tắc và hành động sửa:
  - Quy tắc: mỗi vật đếm được là một instance. Các phần nhìn thấy của cùng một người phải nằm chung một mask.
  - Tôi nới polygon của người lái xuống bao cả phần chân.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `hard_panoptic` / `000000460147.jpg`, lòng đường nhiều làn ở giữa ảnh (hai bên dải phân cách cây xanh) | `road` hay `sidewalk` (ground truth gốc COCO gán phần lớn vùng này là `pavement` → `sidewalk`) | Có vạch kẻ làn, mũi tên chỉ hướng trên mặt đường, ô tô và xe tải đang chạy trên đó. Phần vỉa hè thật nằm bên trái, cao hơn mặt đường, sát nhà. Quy tắc lớp: road/sidewalk phân theo chức năng | Giữ `road` cho lòng đường, `sidewalk` chỉ cho dải bên trái. Hỏi coach: khi ground truth gốc khác quy tắc chức năng thì chấm theo bên nào? |
| 2. `cp4_curb` / `7d83710e-4697c3b2.jpg`, dải đất ướt màu tối ngay dưới ô cỏ và gốc cây, giữa bó vỉa và vỉa hè lát gạch | `sidewalk` (thuộc phần cho người đi, phía trong bó vỉa) hay để trống (không phải mặt lát) | Nằm phía trong bó vỉa, cùng độ cao với vỉa hè, không có xe chạy. Task chỉ có hai class `road`/`sidewalk` | Gán `sidewalk` vì ranh road–sidewalk là bó vỉa; ô cỏ để trống. Hỏi coach: dải đất/cỏ giữa bó vỉa và vỉa hè có nên tính `sidewalk` không? |
| 3. `cp5_occlusion` / `000000336232.jpg`, xe máy góc dưới trái có người lái che phần giữa xe, thùng hàng gắn sau yên | Thùng hàng là một phần `motorcycle` hay vật riêng không có class; xe bị người che là một hay hai mảng | Thùng gắn cố định vào xe, chuyển động cùng xe. Quy tắc lớp: vật bị che vẫn là một instance, chỉ vẽ phần nhìn thấy | Một instance `motorcycle` gồm thân xe + thùng hàng; người lái là `person` riêng, không vẽ phần xe bị chân người che |
