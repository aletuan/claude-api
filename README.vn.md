# Thí Nghiệm Claude API

Framework đánh giá prompt nâng cao sử dụng Anthropic Claude API để tự động tạo và đánh giá test case.

## Cấu Trúc Dự Án

- `001_prompting.ipynb` - Framework đánh giá prompt nâng cao với tự động tạo test case
- `dataset.json` - Dataset test được tạo cho đánh giá prompt (các tác vụ dinh dưỡng/lập kế hoạch bữa ăn)

## Tính Năng

### Framework Đánh Giá Prompt Nâng Cao (`001_prompting.ipynb`)

Một hệ thống toàn diện để tự động tạo, chạy và đánh giá các prompt:

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Quy Trình 001_prompting.ipynb                  │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   THIẾT LẬP      │    │  TẠO DATASET     │    │  ĐÁNH GIÁ        │
│   TÁC VỤ         │    │                  │    │                  │
│ • task_desc      │───▶│ • Tạo ý tưởng    │───▶│ • Chạy prompt    │
│ • input_spec     │    │ • Tạo test case  │    │ • Chấm điểm      │
│ • num_cases      │    │ • Lưu JSON       │    │ • Tính kết quả   │
└──────────────────┘    └──────────────────┘    └──────────────────┘
         │                        │                        │
         ▼                        ▼                        ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Định nghĩa input:│    │ Tự động tạo      │    │ Báo cáo HTML +   │
│ • height: "185"  │    │ kịch bản test    │    │ Kết quả JSON     │
│ • weight: "110"  │    │ với tiêu chí     │    │ • Điểm 1-10      │
│ • goal: "muscle" │    │ đánh giá         │    │ • Tỷ lệ đạt/trượt│
│ • restriction    │    │                  │    │ • Phản hồi chi tiết
└──────────────────┘    └──────────────────┘    └──────────────────┘

Chi Tiết Quy Trình:
1. THIẾT LẬP: Định nghĩa tác vụ & cấu trúc input
2. Ý TƯỞNG: AI tạo các kịch bản test đa dạng
3. TEST CASE: Chuyển ý tưởng → test case có cấu trúc với tiêu chí
4. CHẠY: Thực thi hàm prompt của bạn trên từng case
5. CHẤM ĐIỂM: AI đánh giá output dựa trên tiêu chí
6. BÁO CÁO: Tạo báo cáo đánh giá toàn diện
```

**Thành Phần Chính:**
- **Tự Động Tạo Test**: Tạo các kịch bản đa dạng dựa trên mô tả tác vụ
- **Xử Lý Song Song**: Tạo và đánh giá test case song song
- **Đánh Giá Có Cấu Trúc**: Khớp tiêu chí giải pháp để chấm điểm chính xác
- **Báo Cáo Phong Phú**: Báo cáo HTML với chấm điểm trực quan và phản hồi chi tiết

## Logic Triển Khai Chi Tiết

### 1. Logic Tạo Dataset (`generate_dataset`)

Quá trình tạo dataset theo phương pháp hai giai đoạn:

#### Giai đoạn 1: Tạo Ý Tưởng (`generate_unique_ideas`)
```python
# Tạo các ý tưởng kịch bản đa dạng dựa trên mô tả tác vụ
ideas = self.generate_unique_ideas(task_description, prompt_inputs_spec, num_cases)
```

**Quy Trình:**
1. **Xây Dựng Prompt**: Tạo prompt có cấu trúc yêu cầu Claude tạo `num_cases` ý tưởng test độc đáo
2. **Render Template**: Sử dụng mô tả tác vụ và đặc tả input để tạo kịch bản nhận biết ngữ cảnh
3. **Trích Xuất JSON**: Phân tích phản hồi của Claude để trích xuất danh sách ý tưởng sử dụng stop sequences
4. **Đảm Bảo Đa Dạng**: System prompt nhấn mạnh tạo kịch bản riêng biệt, không trùng lặp

**Ví Dụ Output:**
```json
[
  "Dinh dưỡng vận động viên cường độ cao (cử tạ Olympic)",
  "Lập kế hoạch bữa ăn vận động viên sức bền (marathon)",
  "Bữa ăn vận động viên sinh viên tiết kiệm chi phí"
]
```

#### Giai đoạn 2: Tạo Test Case (`generate_test_case`)
```python
# Chuyển đổi mỗi ý tưởng thành test case có cấu trúc với input và tiêu chí
test_case = self.generate_test_case(task_description, idea, prompt_inputs_spec)
```

**Quy Trình:**
1. **Xác Thực Input**: Đảm bảo chỉ sử dụng các input key được phép từ `prompt_inputs_spec`
2. **Mở Rộng Kịch Bản**: Lấy mỗi ý tưởng và tạo giá trị input thực tế
3. **Định Nghĩa Tiêu Chí**: Tạo 1-4 tiêu chí đánh giá cụ thể, có thể đo lường
4. **Đảm Bảo Cấu Trúc**: Sử dụng xác thực JSON schema để đảm bảo format output nhất quán

**Ví Dụ Test Case:**
```json
{
  "prompt_inputs": {
    "height": "185",
    "weight": "110",
    "goal": "tăng cơ",
    "restriction": "ăn chay"
  },
  "solution_criteria": [
    "Bao gồm các lựa chọn ăn chay giàu protein",
    "Cung cấp thặng dư calo để tăng cơ",
    "Xác định thời gian ăn xung quanh tập luyện"
  ],
  "task_description": "Viết kế hoạch bữa ăn 1 ngày gọn gàng, súc tích cho một vận động viên",
  "scenario": "Dinh dưỡng vận động viên cường độ cao (cử tạ Olympic)"
}
```

#### Xử Lý Song Song
- Sử dụng `ThreadPoolExecutor` với `max_concurrent_tasks` có thể cấu hình
- Xử lý nhiều test case đồng thời để tạo nhanh hơn
- Báo cáo tiến độ tại các mốc 20%
- Xử lý exception một cách khéo léo mà không dừng toàn bộ quá trình

### 2. Logic Đánh Giá (`run_evaluation`)

Hệ thống đánh giá xử lý từng test case thông qua hàm prompt của bạn và chấm điểm kết quả:

#### Giai Đoạn Thực Thi (`run_test_case`)
```python
# Chạy hàm prompt của bạn và chấm điểm output
result = self.run_test_case(test_case, run_prompt_function, extra_criteria)
```

**Quy Trình:**
1. **Thực Thi Prompt**: Gọi `run_prompt_function` của bạn với input test case
2. **Bắt Output**: Lưu trữ phản hồi model thô để đánh giá
3. **Chấm Điểm**: Đưa output qua quá trình đánh giá có cấu trúc

#### Giai Đoạn Chấm Điểm (`grade_output`)
```python
# Sử dụng Claude để đánh giá chất lượng output dựa trên tiêu chí
model_grade = self.grade_output(test_case, output, extra_criteria)
```

**Quy Trình Chấm Điểm:**
1. **Lắp Ráp Ngữ Cảnh**: Kết hợp tác vụ gốc, input, output và tiêu chí vào prompt đánh giá
2. **Hướng Dẫn Chấm Điểm**: Sử dụng thang điểm 1-10 với ngưỡng cụ thể:
   - 1-3: Không đáp ứng yêu cầu bắt buộc
   - 4-6: Đáp ứng bắt buộc nhưng có thiếu sót đáng kể
   - 7-8: Đáp ứng hầu hết tiêu chí với vấn đề nhỏ
   - 9-10: Đáp ứng tất cả tiêu chí một cách toàn diện
3. **Phản Hồi Có Cấu Trúc**: Trả về JSON với điểm mạnh, điểm yếu, lý luận và điểm số
4. **Thực Thi Tiêu Chí Bổ Sung**: Yêu cầu bắt buộc tùy chọn gây ra thất bại tự động nếu vi phạm

**Ví Dụ Đánh Giá:**
```json
{
  "strengths": ["Bao gồm tất cả macro cần thiết", "Cung cấp khẩu phần cụ thể"],
  "weaknesses": ["Thiếu tính toán calo chính xác", "Thiếu chi tiết thời gian ăn"],
  "reasoning": "Nội dung dinh dưỡng tốt nhưng thiếu độ chính xác trong lập kế hoạch calo",
  "score": 6
}
```

#### Theo Dõi Tiến Độ
- Thực thi song song sử dụng thread pool
- Báo cáo tiến độ thời gian thực tại khoảng 20%
- Tính toán và hiển thị điểm trung bình khi hoàn thành
- Xử lý lỗi mà không dừng đánh giá các test case khác

### 3. Xây Dựng Báo Cáo HTML (`generate_prompt_evaluation_report`)

Báo cáo HTML cung cấp trực quan hóa toàn diện kết quả đánh giá:

#### Cấu Trúc Báo Cáo
```html
<!DOCTYPE html>
<html>
<head>
  <!-- CSS responsive cho giao diện chuyên nghiệp -->
</head>
<body>
  <div class="header">
    <!-- Thống kê tóm tắt và các chỉ số chính -->
  </div>
  <table>
    <!-- Kết quả chi tiết cho từng test case -->
  </table>
</body>
</html>
```

#### Tạo Thống Kê Tóm Tắt
```python
total_tests = len(evaluation_results)
avg_score = mean([result["score"] for result in evaluation_results])
pass_rate = 100 * len([s for s in scores if s >= 7]) / total_tests
```

**Chỉ Số Được Tính:**
- **Tổng Test Case**: Đếm tất cả kịch bản được đánh giá
- **Điểm Trung Bình**: Trung bình của tất cả điểm số (thang điểm 1-10)
- **Tỷ Lệ Đạt**: Phần trăm case đạt điểm ≥7 (được coi là đạt)

#### Yếu Tố Trực Quan
1. **Chấm Điểm Mã Màu**:
   - Xanh lá (8-10): Hiệu suất cao
   - Vàng (6-7): Hiệu suất trung bình
   - Đỏ (1-5): Hiệu suất thấp

2. **Layout Responsive**:
   - Thống kê tóm tắt flexbox
   - Thiết kế bảng thân thiện mobile
   - Ngắt dòng văn bản thích hợp cho output dài

3. **Định Dạng Code**:
   - Làm nổi bật cú pháp cho output code
   - Bảo toàn khoảng trắng và định dạng
   - Font monospace cho nội dung kỹ thuật

#### Xử Lý Dữ Liệu
```python
# Chuyển đổi dữ liệu test case thành hàng bảng HTML
for result in evaluation_results:
    prompt_inputs_html = "<br>".join([
        f"<strong>{key}:</strong> {value}"
        for key, value in result["test_case"]["prompt_inputs"].items()
    ])
    criteria_string = "<br>• ".join(result["test_case"]["solution_criteria"])
    # ... định dạng bổ sung
```

**Cột Bảng:**
- **Kịch Bản**: Mô tả cấp cao của test case
- **Input Prompt**: Các cặp key-value được sử dụng làm input
- **Tiêu Chí Giải Pháp**: Tiêu chí đánh giá dạng bullet point
- **Output**: Phản hồi thực tế của prompt (được định dạng)
- **Điểm**: Điểm số mã màu
- **Lý Luận**: Giải thích chi tiết về điểm

#### Output File
- Lưu dưới dạng `output.html` với encoding UTF-8
- File tự chứa với CSS được nhúng
- Có thể mở trực tiếp trong bất kỳ trình duyệt web nào
- Bao gồm tất cả dữ liệu cần thiết để phân tích và chia sẻ

## Thiết Lập

1. Clone repository
2. Cài đặt dependencies:
   ```bash
   pip install anthropic python-dotenv jupyter
   ```
3. Tạo file `.env` với Anthropic API key của bạn:
   ```
   ANTHROPIC_API_KEY=your_api_key_here
   ```
4. Khởi động Jupyter:
   ```bash
   jupyter notebook
   ```

## Sử Dụng

### Sử Dụng Framework Nâng Cao (`001_prompting.ipynb`)

1. **Định nghĩa tác vụ và input của bạn:**
   ```python
   evaluator = PromptEvaluator()
   dataset = evaluator.generate_dataset(
       task_description="Viết kế hoạch bữa ăn cho vận động viên",
       prompt_inputs_spec={
           "height": "Chiều cao vận động viên tính bằng cm",
           "weight": "Cân nặng vận động viên tính bằng kg",
           "goal": "Mục tiêu tập luyện",
           "restriction": "Hạn chế chế độ ăn"
       },
       num_cases=3
   )
   ```

2. **Tạo hàm prompt của bạn:**
   ```python
   def run_prompt(prompt_inputs):
       prompt = f"Tạo kế hoạch bữa ăn cho: {prompt_inputs}"
       return chat_with_claude(prompt)
   ```

3. **Chạy đánh giá:**
   ```python
   results = evaluator.run_evaluation(
       run_prompt_function=run_prompt,
       dataset_file="dataset.json"
   )
   ```

4. **Xem kết quả trong báo cáo HTML** với điểm số, phản hồi và phân tích chi tiết

### Thêm Test Case Mới

Bạn có thể chỉnh sửa prompt tạo dataset để tạo các loại tác vụ đánh giá khác nhau, hoặc chỉnh sửa thủ công `dataset.json` để thêm test case cụ thể.

## Ví Dụ Output

Framework tạo các kịch bản test đa dạng như:
- Dinh dưỡng vận động viên cường độ cao (cử tạ Olympic)
- Lập kế hoạch bữa ăn vận động viên sức bền (marathon)
- Bữa ăn vận động viên sinh viên tiết kiệm chi phí

Mỗi tác vụ được đánh giá toàn diện với chấm điểm và phản hồi chi tiết.

## Đóng Góp

Hãy thoải mái thêm phương pháp đánh giá mới, mở rộng test dataset, hoặc cải thiện thuật toán chấm điểm.