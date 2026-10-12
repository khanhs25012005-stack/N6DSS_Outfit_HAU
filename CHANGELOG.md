# CHANGELOG

## Kiến trúc hiện trạng

Flask ở `web/app.py` phục vụ giao diện và gọi FastAPI qua HTTP. FastAPI nạp model và `data/mon_do.csv` lúc import, dùng `api/recommender.py` để tạo ứng viên/xếp hạng, và SQLite lưu tài khoản, lịch sử cùng phản hồi. `model/train_export.py` sinh dữ liệu và artifact từ workbook nguồn. Giao diện `/style` đã thu tiêu chí; recommender đã có độ tương đồng nội dung cơ bản, nhưng trước thay đổi điểm nội dung chưa được trộn vào điểm xếp hạng cuối. Phản hồi đã được lưu nhưng chưa cập nhật phong cách ưu tiên.

## Thay đổi

- `api/recommender.py`: chuẩn hoá nhãn giới tính bỏ dấu/không phân biệt hoa thường; trộn điểm nội dung và RF–KNN với α = 0.35; bảo đảm các bộ được chọn không dùng chung bất cứ món thành phần nào.
- `api/main.py`: bỏ gọi dịch vụ giải thích Gemini khỏi luồng gợi ý và bỏ trường cấu hình Gemini khỏi health endpoint. Model/catalog tiếp tục được nạp một lần khi khởi động.
- `api/storage.py`: phản hồi thích/mặc lưu phong cách đầu tiên của bộ vào hồ sơ ưu tiên SQLite.
- `web/templates/*.html`: thay tên hiển thị thương hiệu và thuật ngữ; phần chi tiết dùng lời giải thích theo quy tắc hiện có, không gọi dịch vụ ngoài.
- `web/templates/results.html`, `web/templates/detail.html`: thêm phiên bản cache cho ảnh tĩnh.
- `web/static/images/images/SP0337.jpg`: thay nội dung bằng ảnh `SP0337_2.jpg` được đính kèm; giữ nguyên đường dẫn và tên file.

## Kiểm tra

- Content-Based Filtering: đã có trước thay đổi, trong `api/recommender.py` (`_content_similarity`), và tiêu chí được thu ở `web/templates/style.html`; trước đây chưa có điểm nội dung hiển thị riêng trong giao diện.
- Không thể chạy `python -m pytest -q`: Python trong PATH bị chặn bởi WindowsApps; Python runtime đi kèm Codex không có `pytest`. Chưa chạy `tests/run_api_scenarios.py` vì thiếu runtime dự án.
- Chưa có phép đo trước/sau đáng tin cậy: môi trường hiện tại không có dependency dự án để khởi chạy API benchmark. Không ghi số liệu suy đoán.
- Các thay đổi chưa được xác nhận bằng chạy thực tế. Ngưỡng MIN_SCORE/MIN_ITEMS chưa được áp dụng; giới tính Unisex ưu tiên danh mục Unisex và chỉ lấy Nam khi thiếu bộ đủ khác nhau; α mặc định 0.35.

## Chạy dự án

Cài đặt dependency theo `requirements.txt`, sau đó chạy `python run_dev.py`. API docs: `/docs`. Kiểm tra theo hướng dẫn repo: `python -m pytest -q` và `python tests/run_api_scenarios.py`.

## Sửa phần giải thích và thanh điểm

### File đã sửa

- `api/recommender.py`: bổ sung điểm phù hợp sở thích và tiêu chí đã chọn từ màu, phong cách, hoạt động, khoảng nhiệt độ, giới tính đã lọc, dịp, chất liệu khi có ghi chú phù hợp; tạo lời nhận xét theo điểm cao nhất/thấp nhất và thuộc tính món tiêu biểu. Điểm cuối vẫn là điểm xếp hạng nội dung kết hợp RF–KNN theo `CONTENT_WEIGHT = 0.35` hiện có.
- `web/app.py`: chuẩn hóa điểm hiển thị thành số hữu hạn, làm tròn về số nguyên và kẹp trong [0, 100]; dùng cùng giá trị chuẩn cho nhãn, độ rộng và ARIA; ẩn hàng sở thích nếu không có đầu vào sở thích.
- `web/templates/detail.html`: thêm hàng sở thích/tiêu chí, ARIA progressbar và dùng một giá trị cho nhãn cùng fill; sửa chú thích giải thích.
- `web/static/css/style.css`: nguyên nhân gốc là `.bar-fill` không có width nên phần tử block chiếm toàn bộ track. Đặt mặc định width 0 và transition; width đúng được gán từ giá trị chuẩn duy nhất trong template.

### Công thức

- Sở thích: trung bình các mức khớp có dữ liệu giữa phong cách đã chọn, màu đã chọn và từ khóa trong ghi chú với thuộc tính màu/phong cách/loại/chất liệu/dịp của món. Không có dữ liệu thì không hiển thị hàng.
- Tiêu chí: trung bình có trọng số của giới tính (đã qua lọc cứng), hoạt động, nhiệt độ, phong cách, màu, dịp và ghi chú/chất liệu.
- Điểm xếp hạng: giữ điểm cuối hiện có từ `CONTENT_WEIGHT = 0.35` và điểm RF–KNN.
- Thanh: lấy giá trị điểm, xử lý null/NaN thành 0, làm tròn cùng cách với nhãn, clamp về [0, 100], và dùng giá trị đó cho `aria-valuenow` cùng `width`.

### Kiểm tra lượt sửa này

- Python compileall: thành công.
- Không thể chạy pytest/API hoặc mở trang để kiểm tra DOM và responsive: Python hệ thống bị chặn, runtime kèm Codex thiếu pytest và dependency dự án. Chưa xác nhận ảnh chụp hoặc kích thước tính toán trong trình duyệt.
