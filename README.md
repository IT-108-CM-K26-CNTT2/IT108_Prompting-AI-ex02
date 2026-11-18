# Chuỗi Prompt viết bài PR sản phẩm công nghệ (Prompt Chaining)

## 1. Mục tiêu và bối cảnh

- Thành thạo Prompt Chaining để giải quyết tác vụ nhiều công đoạn.
- Thiết lập và truyền ngữ cảnh (Context) liên tiếp giữa các prompt thành phần.
- Xử lý đứt gãy ngữ cảnh, giữ nhất quán khi viết bài dài.

**Bối cảnh:** Phòng Marketing cần một bài PR chuyên sâu (1.000 – 1.500 từ) giới thiệu giải pháp AI Chatbot mới cho doanh nghiệp. Một prompt đơn lẻ thường cho bài hời hợt, lặp ý, thiếu chiều sâu. Giải pháp là tách thành 3 bước, mỗi bước chỉ làm một việc và trao lại kết quả theo cấu trúc cố định cho bước sau.

---

## 2. Tổng quan chuỗi Prompt

```text
[BRIEF + TONE_CARD]
        │
        ▼
  PROMPT 1 ── Nghiên cứu khách hàng & pain points
        │  →  OUTPUT_1  (Persona, Pain points P1..Pn, Từ khóa KW)
        ▼
  PROMPT 2 ── Dàn ý chi tiết
        │  →  OUTPUT_2  (Headline, Dàn ý S0..S_CTA, bảng truy vết P ↔ S)
        ▼
  PROMPT 3 ── Viết bài + tối ưu SEO
        │  →  BÀI PR hoàn chỉnh
        │
        └── nếu bị ngắt giữa chừng ──▶ PROMPT 3C (lệnh tiếp tục chính xác)
```

| Bước | Vai trò | Đầu vào bắt buộc | Đầu ra | Bước sau dùng để làm gì |
|---|---|---|---|---|
| Prompt 1 | Nghiên cứu | BRIEF, TONE_CARD | OUTPUT_1 | Là "nguyên liệu": pain point nào phải được giải quyết, từ khóa nào cần dùng |
| Prompt 2 | Chiến lược nội dung | BRIEF, TONE_CARD, **OUTPUT_1** | OUTPUT_2 | Là "bản thiết kế": thứ tự, độ dài, từ khóa của từng phần |
| Prompt 3 | Viết bài | BRIEF, TONE_CARD, OUTPUT_1, **OUTPUT_2** | Bài PR | Sản phẩm cuối |
| Prompt 3C | Điều khiển tiếp tục | Như Prompt 3 + đoạn đã viết | Phần còn lại của bài | Nối liền mạch với đoạn bị ngắt |

**Cách vận hành:**

1. Điền BRIEF (mục 3.2), gửi Prompt 1.
2. **Điểm kiểm duyệt 1:** đọc OUTPUT_1, xóa hoặc sửa các ý đánh dấu `[CẦN XÁC MINH]`.
3. Dán nguyên văn OUTPUT_1 vào Prompt 2, gửi.
4. **Điểm kiểm duyệt 2:** kiểm tra dàn ý, bảng truy vết và ngân sách từ.
5. Dán nguyên văn OUTPUT_2 vào Prompt 3, gửi.
6. Nếu bài không kết thúc bằng `[[HẾT-BÀI]]`, dùng Prompt 3C (mục 7).
7. Ghép bài, xóa các dấu `[[...]]` và Phần B (báo cáo SEO) trước khi đăng.

Mỗi prompt là một lượt gửi riêng. Chuỗi không gộp được thành một prompt vì cổng kiểm tra ở Prompt 2 và 3 từ chối chạy nếu thiếu đầu ra của bước trước.

---

## 3. Khối dữ liệu dùng chung và quy ước truyền dữ liệu

### 3.1 Cấu trúc dữ liệu truyền giữa các bước (Data Contract)

| Khối | Tạo bởi | Dùng ở | Quy tắc |
|---|---|---|---|
| `<BRIEF>` | Người dùng | P1, P2, P3, 3C | Nguồn sự thật duy nhất về sản phẩm và số liệu |
| `<TONE_CARD>` | Người dùng (mẫu cố định) | P1, P2, P3, 3C | Dán nguyên văn ở **mọi** prompt, không sửa giữa chừng |
| `<OUTPUT_1>` | Prompt 1 | P2, P3, 3C | Gồm mục A–E, dán nguyên văn |
| `<OUTPUT_2>` | Prompt 2 | P3, 3C | Gồm mục A–E, dán nguyên văn |
| `<ANCHOR>` / `<ARTICLE_SO_FAR>` | Người dùng | 3C | Đoạn bài đã viết, dùng để nối tiếp |

**Hệ thống mã định danh (ID) giữ tính nhất quán xuyên chuỗi:**

- `PE1, PE2…`: persona. `P1, P2…`: pain point. Cả hai do Prompt 1 tạo.
- `KW-A`: từ khóa chính. `KW-1, KW-2…`: từ khóa phụ. `KW-L1…`: từ khóa dài (long-tail). Do Prompt 1 tạo.
- `S0`: Headline và Lead. `S1, S2…`: các H2. `S2.1, S2.2…`: các H3. `S_CTA`: lời kêu gọi hành động. Do Prompt 2 tạo.
- **Truy vết hai chiều:** mỗi phần S phải trỏ tới ít nhất một P, và mỗi P phải xuất hiện ở ít nhất một phần S.

**Bốn nguyên tắc bất biến:** (1) dán nguyên văn, không tóm tắt; (2) ID không đổi; (3) số liệu chỉ lấy từ BRIEF; (4) mỗi bước chỉ làm việc của bước đó.

### 3.2 Mẫu BRIEF (người dùng điền)

```text
<BRIEF>
- Tên sản phẩm: [...]
- Mô tả 1–2 câu: [...]
- Tính năng chính (đánh số): [1) ... 2) ... 3) ...]
- Số liệu/bằng chứng ĐƯỢC PHÉP dùng (kèm nguồn): [...]
- Đối tượng mục tiêu sơ bộ: [ngành, quy mô doanh nghiệp, vai trò]
- Mục tiêu bài PR: [nhận biết thương hiệu / thu hút dùng thử / ...]
- CTA mong muốn: [hành động + đường dẫn hoặc liên hệ]
- Từ khóa gợi ý (nếu có): [...]
</BRIEF>
```

### 3.3 TONE_CARD (dùng chung, giữ nguyên ở mọi bước)

```text
<TONE_CARD>
GIỌNG ĐIỆU: chuyên nghiệp, tin cậy, điềm tĩnh, dựa trên bằng chứng. Thuyết phục bằng sự rõ ràng và dữ kiện, không bằng cường điệu.
NGÔI KỂ: ngôi thứ ba. Gọi sản phẩm bằng tên "[Tên sản phẩm]", gọi người đọc là "doanh nghiệp". Không dùng "bạn", "mình", "chúng tôi", "chúng ta".
CÂU VĂN: câu chủ động, trung bình 15–25 từ mỗi câu, mỗi đoạn 2–4 câu.
THUẬT NGỮ CHỐT (dùng đúng một cách gọi xuyên suốt): "AI Chatbot" (sản phẩm), "khách hàng" (người nhắn tin), "doanh nghiệp" (người mua), "nhân viên chăm sóc khách hàng". Chỉ dùng "chatbot theo kịch bản" khi nói về loại cũ để đối chiếu. Không đổi sang "bot", "trợ lý ảo", "robot".
CẤM: dấu "!", emoji, từ cường điệu ("đột phá", "cách mạng", "số 1", "tốt nhất", "hoàn hảo", "duy nhất"), câu hỏi tu từ liên tiếp, số liệu hoặc cam kết ngoài BRIEF.
TRÌNH BÀY: số dùng dấu chấm ngăn hàng nghìn (1.000); phần trăm viết liền (40%); thuật ngữ kỹ thuật được giải thích ngắn ở lần xuất hiện đầu tiên.
</TONE_CARD>
```

---

## 4. PROMPT 1: Nghiên cứu đối tượng và pain points

```text
# VAI TRÒ
Bạn là chuyên gia nghiên cứu thị trường B2B công nghệ, am hiểu doanh nghiệp Việt Nam.

# DỮ LIỆU ĐẦU VÀO
<BRIEF>
[dán BRIEF]
</BRIEF>
<TONE_CARD>
[dán TONE_CARD]
</TONE_CARD>

# NHIỆM VỤ (BƯỚC 1/3)
Chỉ nghiên cứu khách hàng mục tiêu và nỗi đau mà sản phẩm giải quyết.
KHÔNG viết dàn ý. KHÔNG viết bài. Kết quả của bạn sẽ được dán nguyên văn vào Prompt 2, nên phải đúng định dạng bên dưới.

# QUY TRÌNH
1. Persona: xác định 2–3 persona (người ra quyết định và người ảnh hưởng quyết định mua). Mỗi persona gồm vai trò, mục tiêu, mối bận tâm khi mua.
2. Pain points: liệt kê 5–7 pain point. Mỗi pain point gắn một persona, nêu biểu hiện thực tế, hậu quả kinh doanh, tính năng của sản phẩm giải quyết được nó, mức độ nghiêm trọng (1–5) và nguồn chứng cứ.
3. Chọn Top 3 pain point sẽ mở đầu bài viết (Lead), kèm lý do ngắn.
4. Từ khóa SEO sơ bộ: 1 từ khóa chính (KW-A), 3–5 từ khóa phụ (KW-1…), 3–5 từ khóa dài (KW-L1…). Ghi ý định tìm kiếm và pain point liên quan.
5. Tự kiểm tra theo checklist ở mục E.

# QUY TẮC CHỐNG BỊA
- Chỉ gắn vào pain point những tính năng có trong BRIEF.
- Không đưa số liệu thị trường, tỷ lệ, tên nghiên cứu ngoài BRIEF. Nếu một ý cần số liệu mà BRIEF không có, ghi "[CẦN XÁC MINH]" thay vì tự nghĩ ra.
- Mỗi pain point phải cụ thể đến mức người đọc nhận ra tình huống của chính mình.

# ĐỊNH DẠNG ĐẦU RA (bắt buộc, đúng thứ tự)
<OUTPUT_1>
## A. PERSONA
| ID | Persona | Vai trò trong quyết định mua | Mục tiêu | Mối bận tâm khi mua |
|---|---|---|---|---|

## B. PAIN POINTS
| ID | Pain point (1 câu) | Persona | Biểu hiện thực tế | Hậu quả kinh doanh | Tính năng giải quyết | Mức độ (1–5) | Nguồn chứng cứ (BRIEF / CẦN XÁC MINH) |
|---|---|---|---|---|---|---|---|

## C. TOP 3 CHO LEAD
- P?, P?, P? và lý do chọn

## D. TỪ KHÓA SEO
| ID | Từ khóa | Loại (chính/phụ/dài) | Ý định tìm kiếm | Gắn với P-ID |
|---|---|---|---|---|

## E. KIỂM TRA
- [ ] Có 2–3 persona
- [ ] Có 5–7 pain point, mỗi cái đều có tính năng giải quyết lấy từ BRIEF
- [ ] Không có số liệu ngoài BRIEF (hoặc đã gắn CẦN XÁC MINH)
- [ ] Có đủ KW-A, ≥3 KW phụ, ≥3 KW dài
</OUTPUT_1>

Kết thúc bằng đúng một dòng: >>> SẴN SÀNG CHUYỂN SANG PROMPT 2
```

---

## 5. PROMPT 2: Dàn ý chi tiết

```text
# VAI TRÒ
Bạn là biên tập viên trưởng kiêm chiến lược gia nội dung SEO cho báo chí và PR công nghệ.

# DỮ LIỆU ĐẦU VÀO BẮT BUỘC
<BRIEF>
[dán BRIEF]
</BRIEF>
<TONE_CARD>
[dán TONE_CARD]
</TONE_CARD>
<OUTPUT_1>
[dán NGUYÊN VĂN kết quả Prompt 1]
</OUTPUT_1>

# CỔNG KIỂM TRA ĐẦU VÀO
Nếu <OUTPUT_1> trống, thiếu mục A–D, hoặc không có ID dạng P1/KW-A: DỪNG, chỉ trả lời
"THIẾU DỮ LIỆU BƯỚC 1: [liệt kê mục thiếu]" và không làm gì thêm.

# NHIỆM VỤ (BƯỚC 2/3)
Lập dàn ý chi tiết cho bài PR dựa HOÀN TOÀN trên OUTPUT_1. KHÔNG viết nội dung bài.
Đơn vị đếm từ: như Microsoft Word (mỗi cụm ký tự cách nhau bởi khoảng trắng là một từ).

# RÀNG BUỘC CẤU TRÚC
1. Tổng ngân sách từ: 1.200–1.400 (nằm trong khung 1.000–1.500).
2. Headline H1: 1 phương án chính + 2 phương án thay thế, mỗi cái ≤ 65 ký tự, chứa KW-A.
3. Lead (S0): 80–120 từ, mở bằng đúng 3 pain point ở OUTPUT_1 mục C, theo logic: tình huống thực tế → hậu quả → hứa hẹn giải pháp.
4. Thân bài: 4–6 H2, mỗi H2 có 0–3 H3. Tiêu đề H2/H3 chứa từ khóa được gán, đọc tự nhiên.
5. Mạch bài: Vấn đề → Hệ quả → Giải pháp → Cách hoạt động → Bằng chứng → Triển khai/Lợi ích → CTA.
6. Cuối bài: S_CTA 60–100 từ, đúng CTA trong BRIEF.
7. Truy vết hai chiều: mỗi phần gắn ≥ 1 P-ID; mỗi P-ID xuất hiện ở ≥ 1 phần.
8. Mỗi ý chính chỉ được giao cho MỘT phần để tránh lặp ý khi viết.

# ĐỊNH DẠNG ĐẦU RA (bắt buộc, đúng thứ tự)
<OUTPUT_2>
## A. HEADLINE VÀ META
- H1 chính / H1 thay thế 1 / H1 thay thế 2 (kèm số ký tự)
- Meta description (≤ 155 ký tự, chứa KW-A, có lời mời hành động)
- Slug đề xuất

## B. DÀN Ý CHI TIẾT
Với từng phần, theo mẫu:
### [S-ID] [H1/Lead/H2/H3/CTA]: Tiêu đề
- Gắn pain point: P?
- Mục tiêu của phần:
- Ý chính (3–4 gạch đầu dòng, không trùng với phần khác):
- Bằng chứng được phép dùng (trích từ BRIEF):
- Từ khóa gán: KW-?
- Ngân sách từ:
- Câu dẫn sang phần kế tiếp (1 câu, để nối mạch):

## C. BẢNG TRUY VẾT
| P-ID | Được giải quyết ở phần |
|---|---|

## D. NGÂN SÁCH TỪ
Cộng dồn từng cặp: [a] + [b] = ...; ... = TỔNG (phải nằm trong 1.200–1.400)

## E. KIỂM TRA
- [ ] Mọi P-ID đều có phần phụ trách
- [ ] Mọi KW đều được gán ít nhất một phần
- [ ] Không có hai phần cùng giữ một ý chính
- [ ] Tổng ngân sách từ hợp lệ
</OUTPUT_2>

Kết thúc bằng đúng một dòng: >>> SẴN SÀNG CHUYỂN SANG PROMPT 3
```

---

## 6. PROMPT 3: Viết bài chi tiết và tối ưu SEO

```text
# VAI TRÒ
Bạn là chuyên viên PR công nghệ kiêm SEO copywriter, viết cho tạp chí kinh doanh và công nghệ.

# DỮ LIỆU ĐẦU VÀO BẮT BUỘC
<BRIEF>
[dán BRIEF]
</BRIEF>
<TONE_CARD>
[dán TONE_CARD]
</TONE_CARD>
<OUTPUT_1>
[dán NGUYÊN VĂN kết quả Prompt 1]
</OUTPUT_1>
<OUTPUT_2>
[dán NGUYÊN VĂN kết quả Prompt 2]
</OUTPUT_2>
PHẠM_VI_VIẾT: [TOÀN BỘ | hoặc ví dụ "S0–S2"]

# CỔNG KIỂM TRA ĐẦU VÀO
Nếu thiếu <OUTPUT_2> hoặc OUTPUT_2 không có mục B (dàn ý): DỪNG, chỉ trả lời
"THIẾU DỮ LIỆU BƯỚC 2: [liệt kê mục thiếu]".

# NHIỆM VỤ (BƯỚC 3/3)
Viết bài PR hoàn chỉnh theo ĐÚNG OUTPUT_2: không thêm, không bỏ, không đổi thứ tự, không đổi tên H2/H3. Chỉ viết trong PHẠM_VI_VIẾT.

# QUY TẮC VIẾT
1. Giọng điệu: tuân thủ TONE_CARD ở mọi phần. Trước khi viết mỗi phần, đối chiếu lại TONE_CARD để giữ nguyên giọng, ngôi kể và thuật ngữ. Phần cuối phải cùng giọng với phần đầu.
2. Nội dung: mỗi phần chỉ triển khai các ý chính được giao trong OUTPUT_2. Ý đã dùng ở phần trước không được nhắc lại, chỉ được dẫn chiếu ngắn.
3. Mối nối: câu mở đầu mỗi phần phải tiếp nối "Câu dẫn" của phần trước.
4. Bằng chứng: chỉ dùng số liệu, tên, cam kết có trong BRIEF. Không bịa trích dẫn, không bịa khách hàng.
5. Độ dài: đúng ngân sách từ của từng phần (sai lệch ≤ ±10%); tổng bài 1.000–1.500 từ.
6. SEO:
   - KW-A có mặt trong H1, trong 100 từ đầu của Lead, trong ít nhất 1 H2 và trong S_CTA; tổng 8–14 lần trong cả bài (mật độ khoảng 0,7–1,2%), không nhồi nhét.
   - Mỗi từ khóa phụ (KW-1…) xuất hiện 2–4 lần ở các phần được gán; từ khóa dài xuất hiện tự nhiên ở ít nhất 1 chỗ.
   - Đoạn ≤ 4 câu; có ít nhất 1 danh sách gạch đầu dòng hoặc bảng để dễ đọc lướt.
   - Không in đậm tràn lan; chỉ in đậm tối đa 1 cụm mỗi phần.
7. Cấu trúc đầu ra theo Markdown: # H1, đoạn Lead (không có tiêu đề), ## H2, ### H3, phần CTA cuối bài.

# CƠ CHẾ CHỐNG ĐỨT GẪY (bắt buộc)
- Sau khi hoàn tất mỗi phần, in một dòng riêng: [[S?-HOÀN-TẤT | ~số từ]]
- Khi viết xong toàn bộ nội dung bài, in một dòng riêng: [[HẾT-BÀI]]
- Không viết vội hoặc nén nội dung để cho kịp. Nếu sắp hết dung lượng, hoàn thành câu hiện tại cho trọn ý rồi dừng.
- Không in lời chào, lời giải thích, hay tóm tắt xen giữa bài.

# ĐỊNH DẠNG ĐẦU RA
PHẦN A: BÀI VIẾT (theo mục 7 ở trên).
PHẦN B (chỉ in sau [[HẾT-BÀI]]): BÁO CÁO SEO VÀ NHẤT QUÁN
| Từ khóa | Số lần xuất hiện | Mục tiêu | Đạt? |
|---|---|---|---|
Tổng số từ: cộng dồn từng phần.
Checklist: [ ] có Headline [ ] Lead giải quyết pain point [ ] H2/H3 khớp dàn ý [ ] có CTA cuối bài [ ] giọng điệu nhất quán [ ] không có số liệu ngoài BRIEF
```

---

## 7. PROMPT 3C: Xử lý khi bài bị ngắt giữa chừng (đạt max token)

### 7.1 Nhận biết bài bị ngắt

Bài chưa kết thúc bằng `[[HẾT-BÀI]]` nghĩa là chưa hoàn tất. Có 3 kiểu ngắt:

| Kiểu ngắt | Dấu hiệu | Điểm nối tiếp |
|---|---|---|
| Ngắt giữa từ/giữa câu | Câu cuối dở dang | Viết phần còn lại của đúng từ/câu đó |
| Ngắt giữa phần (sau câu trọn vẹn) | Câu cuối đủ ý, chưa có dòng `[[S?-HOÀN-TẤT]]` | Viết câu kế tiếp trong cùng phần |
| Ngắt ở ranh giới phần | Dòng cuối là `[[S?-HOÀN-TẤT]]` | Bắt đầu bằng heading của phần kế tiếp theo OUTPUT_2 |

**Không** yêu cầu AI viết lại từ đầu và **không** xóa đoạn đã có. Sao chép nguyên văn khoảng 80–120 từ cuối cùng làm `<ANCHOR>`.

### 7.2 Lệnh tiếp tục A (cùng cuộc trò chuyện, ngữ cảnh còn)

```text
Bài viết bị ngắt do giới hạn độ dài. Hãy TIẾP TỤC CHÍNH XÁC từ điểm bị ngắt.

Đoạn cuối cùng đã in (chỉ để định vị, KHÔNG in lại):
<ANCHOR>
[dán nguyên văn 80–120 từ cuối]
</ANCHOR>

Tiến độ: đã hoàn tất [S0–S?]; đang viết dở [S?]; còn lại [S?–S_CTA].

QUY TẮC NỐI TIẾP:
1. Ký tự đầu tiên của câu trả lời phải là ký tự đứng ngay sau ký tự cuối cùng của <ANCHOR>.
   - ANCHOR dừng giữa từ hoặc giữa câu: viết nốt phần còn lại của từ/câu đó, tuyệt đối không lặp phần đã có.
   - ANCHOR dừng ở câu trọn vẹn: bắt đầu câu kế tiếp trong cùng phần.
   - ANCHOR dừng ở dòng [[S?-HOÀN-TẤT]]: bắt đầu bằng heading phần kế tiếp đúng như OUTPUT_2.
2. KHÔNG mở đầu bằng lời chào, "Tiếp tục:", "Dưới đây là...", KHÔNG tóm tắt phần đã viết, KHÔNG nhắc lại heading đã in, KHÔNG lặp ý đã triển khai.
3. Nếu ANCHOR dừng giữa bảng hoặc danh sách: viết tiếp từ hàng/mục kế tiếp, không lặp dòng tiêu đề bảng.
4. Giữ nguyên TONE_CARD, thuật ngữ chốt, ngôi kể và từ khóa gán cho các phần còn lại.
5. Tiếp tục đúng OUTPUT_2 đến hết S_CTA, in [[HẾT-BÀI]], rồi in PHẦN B (báo cáo SEO tính trên TOÀN BỘ bài).
6. Nếu lại sắp hết dung lượng: hoàn thành câu hiện tại cho trọn ý rồi dừng, chờ lệnh tiếp.
```

**Lệnh rút gọn khẩn cấp** (chỉ dùng khi lượt trả lời trước là tin nhắn cuối và bị ngắt gọn):

```text
Tiếp tục chính xác từ ký tự cuối cùng của câu trả lời trước. Không lặp lại, không chào hỏi, không giải thích.
```

### 7.3 Lệnh tiếp tục B (cuộc trò chuyện mới hoặc AI đã "quên" ngữ cảnh)

Khi cửa sổ ngữ cảnh bị đứt, nạp lại toàn bộ khối dữ liệu cùng phần bài đã viết:

```text
Bạn đang hoàn thiện một bài PR đang viết dở. Hãy nạp toàn bộ dữ liệu dưới đây, rồi chỉ viết PHẦN CÒN THIẾU.

<BRIEF>[dán BRIEF]</BRIEF>
<TONE_CARD>[dán TONE_CARD]</TONE_CARD>
<OUTPUT_1>[dán OUTPUT_1]</OUTPUT_1>
<OUTPUT_2>[dán OUTPUT_2]</OUTPUT_2>
<ARTICLE_SO_FAR>
[dán TOÀN BỘ phần bài đã có, giữ nguyên các dòng [[...]]]
</ARTICLE_SO_FAR>

NHIỆM VỤ:
1. Đọc toàn bộ ARTICLE_SO_FAR để nắm giọng điệu, ngôi kể, thuật ngữ và các ý ĐÃ viết (không được lặp).
2. Xác định điểm ngắt: ký tự cuối cùng của ARTICLE_SO_FAR.
3. Viết tiếp theo ĐÚNG các quy tắc nối tiếp 1–6 của Lệnh A. Ký tự đầu tiên của câu trả lời nối liền ký tự cuối của ARTICLE_SO_FAR.
4. Chỉ in phần mới. Không in lại bất kỳ ký tự nào của ARTICLE_SO_FAR.
```

### 7.4 Kiểm tra mối nối và xử lý sự cố

| Sự cố | Cách xử lý |
|---|---|
| AI lặp lại câu/cụm từ cuối của ANCHOR | Xóa phần lặp khi ghép; hoặc gửi lại và thêm: "Không in lại bất kỳ ký tự nào của ANCHOR" |
| AI mở đầu bằng "Tiếp tục:", lời chào | Tạo lại với dòng: "Ký tự đầu tiên phải thuộc nội dung bài viết" |
| Giọng điệu đổi (thân mật hơn, cường điệu hơn) | Dán lại TONE_CARD, yêu cầu viết lại riêng đoạn mới theo TONE_CARD |
| Ngắt giữa bảng hoặc danh sách | Đưa cả bảng/danh sách đang dở vào ANCHOR, yêu cầu viết tiếp từ hàng kế tiếp |
| Bài vẫn bị ngắt nhiều lần | Chia lô bằng PHẠM_VI_VIẾT: "S0–S2", rồi "S3–S5", rồi "S6–S_CTA" |
| Từ khóa dùng quá hoặc thiếu so với mục tiêu | Chỉnh trực tiếp khi ghép; số lần đếm ở PHẦN B giúp xác định |

**Ba bước kiểm tra mối nối trước khi ghép:** (1) đọc liền hai câu quanh điểm nối, không được thừa hoặc thiếu chữ; (2) tìm ý đã xuất hiện trước điểm nối, không được lặp; (3) so lại thuật ngữ chốt và ngôi kể với TONE_CARD.

---

## 8. Chạy thử minh họa

*Sản phẩm và số liệu dưới đây là giả định, chỉ để minh họa luồng dữ liệu.*

### 8.1 BRIEF mẫu

```text
<BRIEF>
- Tên sản phẩm: BizChat AI
- Mô tả: nền tảng AI Chatbot cho doanh nghiệp, tự động trả lời khách hàng trên website, Facebook Messenger và Zalo OA.
- Tính năng chính: 1) Hiểu tiếng Việt, ghi nhớ ngữ cảnh nhiều lượt hội thoại; 2) Học từ tài liệu của doanh nghiệp (FAQ, catalog, chính sách) và trả lời dựa trên nguồn đó; 3) Chuyển cuộc trò chuyện sang nhân viên kèm toàn bộ lịch sử khi cần; 4) Tích hợp CRM và dữ liệu đơn hàng; 5) Bảng phân tích câu hỏi phổ biến và tỷ lệ giải quyết; 6) Triển khai trong 7 ngày.
- Số liệu được phép dùng (thử nghiệm nội bộ, 3 tháng): thời gian phản hồi đầu tiên giảm từ trung bình 8 phút xuống dưới 30 giây; 65% yêu cầu thường gặp được xử lý không cần nhân viên; hoạt động 24/7.
- Đối tượng sơ bộ: doanh nghiệp vừa và nhỏ (10–200 nhân viên) trong bán lẻ, thương mại điện tử, dịch vụ.
- Mục tiêu PR: thu hút đăng ký dùng thử.
- CTA: Đăng ký dùng thử miễn phí 14 ngày tại bizchat.example.vn
- Từ khóa gợi ý: AI Chatbot cho doanh nghiệp
</BRIEF>
```

### 8.2 OUTPUT_1 mẫu (rút gọn)

```text
<OUTPUT_1>
## A. PERSONA
| ID | Persona | Vai trò trong quyết định mua | Mục tiêu | Mối bận tâm khi mua |
|---|---|---|---|---|
| PE1 | Chủ doanh nghiệp vừa và nhỏ | Người quyết định | Tăng doanh thu, kiểm soát chi phí | Hiệu quả đầu tư, mức độ phức tạp triển khai |
| PE2 | Trưởng bộ phận chăm sóc khách hàng | Ảnh hưởng mạnh | Giữ thời gian phản hồi, giảm quá tải | Chatbot trả lời sai làm mất uy tín |
| PE3 | Quản lý bán hàng/thương mại điện tử | Ảnh hưởng | Không bỏ sót khách hàng hỏi mua | Kết nối với CRM và đơn hàng |

## B. PAIN POINTS
| ID | Pain point | Persona | Biểu hiện thực tế | Hậu quả kinh doanh | Tính năng giải quyết | Mức độ | Nguồn |
|---|---|---|---|---|---|---|---|
| P1 | Khách hàng chờ phản hồi quá lâu | PE3, PE1 | Tin nhắn ngoài giờ nằm chờ nhiều phút | Khách hàng chuyển sang nơi khác | Trả lời tự động 24/7 | 5 | BRIEF (8 phút → dưới 30 giây) |
| P2 | Nhân viên quá tải vì câu hỏi lặp lại | PE2 | Cùng nhóm câu hỏi giá, đổi trả, đơn hàng | Chất lượng tư vấn giảm vào cao điểm | Học từ tài liệu, xử lý yêu cầu thường gặp | 4 | BRIEF (65% không cần nhân viên) |
| P3 | Chatbot cũ trả lời cứng nhắc hoặc sai | PE2, PE1 | Trả lời theo kịch bản, không hiểu ngữ cảnh | Mất uy tín thương hiệu | Hiểu tiếng Việt, trả lời dựa trên nguồn tài liệu | 4 | BRIEF |
| P4 | Khách hàng gặp tình huống phức tạp không tìm được người hỗ trợ | PE2 | Khách bị "kẹt" ở chatbot | Trải nghiệm tiêu cực | Chuyển sang nhân viên kèm lịch sử | 3 | BRIEF |
| P5 | Dữ liệu hội thoại rời rạc, khó cải tiến | PE1 | Không biết khách hàng hỏi gì nhiều nhất | Bỏ lỡ cơ hội cải thiện sản phẩm | Bảng phân tích, tích hợp CRM | 3 | BRIEF |

## C. TOP 3 CHO LEAD
- P1, P2, P3: ba pain point có mức độ cao nhất, đều gắn trực tiếp với số liệu có trong BRIEF.

## D. TỪ KHÓA SEO
| ID | Từ khóa | Loại | Ý định tìm kiếm | Gắn với |
|---|---|---|---|---|
| KW-A | AI Chatbot cho doanh nghiệp | Chính | Tìm giải pháp | P1–P5 |
| KW-1 | chatbot chăm sóc khách hàng | Phụ | Tìm giải pháp | P2, P4 |
| KW-2 | chatbot tiếng Việt | Phụ | So sánh công nghệ | P3 |
| KW-3 | tự động hóa chăm sóc khách hàng | Phụ | Tìm hiểu | P2, P5 |
| KW-L1 | cách giảm thời gian phản hồi khách hàng | Dài | Tìm cách giải quyết | P1 |
| KW-L2 | chatbot Zalo OA cho doanh nghiệp nhỏ | Dài | Tìm giải pháp | P1, P5 |

## E. KIỂM TRA
- [x] 3 persona  - [x] 5 pain point, tính năng lấy từ BRIEF  - [x] không có số liệu ngoài BRIEF  - [x] đủ từ khóa
</OUTPUT_1>
>>> SẴN SÀNG CHUYỂN SANG PROMPT 2
```

### 8.3 OUTPUT_2 mẫu (rút gọn)

```text
<OUTPUT_2>
## A. HEADLINE VÀ META
- H1 chính: AI Chatbot cho doanh nghiệp: Phản hồi khách hàng dưới 30 giây (≈ 61 ký tự)
- Meta (≈ 130 ký tự): BizChat AI giúp doanh nghiệp phản hồi khách hàng 24/7, giảm tải cho nhân viên và triển khai trong 7 ngày. Dùng thử miễn phí 14 ngày.

## B. DÀN Ý CHI TIẾT (tóm lược)
| Phần | Cấp | Tiêu đề | Gắn P | Từ khóa | Từ |
|---|---|---|---|---|---|
| S0 | Lead | (không tiêu đề) | P1, P2, P3 | KW-A | 100 |
| S1 | H2 | Thời gian phản hồi chậm làm doanh nghiệp mất khách như thế nào | P1, P2 | KW-L1, KW-3 | 200 |
| S2 | H2 | AI Chatbot cho doanh nghiệp hoạt động ra sao | P3 | KW-A, KW-2 | 120 |
| S2.1 | H3 | Hiểu tiếng Việt và trả lời dựa trên tài liệu của doanh nghiệp | P3 | KW-2 | 140 |
| S2.2 | H3 | Chuyển sang nhân viên đúng thời điểm | P4 | KW-1 | 130 |
| S3 | H2 | Dữ liệu hội thoại: từ nhật ký rời rạc thành thông tin hữu ích | P5 | KW-3, KW-L2 | 150 |
| S4 | H2 | Triển khai trong 7 ngày trên website, Messenger và Zalo OA | P1, P5 | KW-L2 | 170 |
| S5 | H2 | Kết quả từ giai đoạn thử nghiệm | P1, P2 | KW-A | 140 |
| S_CTA | CTA | Đăng ký dùng thử miễn phí 14 ngày | P1–P5 | KW-A | 70 |

(Mỗi phần còn có đủ: mục tiêu, ý chính, bằng chứng được phép dùng, câu dẫn sang phần sau.)

## C. BẢNG TRUY VẾT
P1 → S0, S1, S4, S5 | P2 → S0, S1, S5 | P3 → S0, S2, S2.1 | P4 → S2.2 | P5 → S3, S4

## D. NGÂN SÁCH TỪ
100 + 200 = 300; 300 + 120 = 420; 420 + 140 = 560; 560 + 130 = 690; 690 + 150 = 840; 840 + 170 = 1.010; 1.010 + 140 = 1.150; 1.150 + 70 = 1.220 → **TỔNG 1.220** (trong khung 1.200–1.400) ✓

## E. KIỂM TRA
- [x] P1–P5 đều có phần phụ trách  - [x] mọi KW đều được gán  - [x] không trùng ý chính  - [x] ngân sách từ hợp lệ
</OUTPUT_2>
>>> SẴN SÀNG CHUYỂN SANG PROMPT 3
```

### 8.4 Kết quả Prompt 3 (trích đoạn): Headline, Lead và S1 bị ngắt

```markdown
# AI Chatbot cho doanh nghiệp: Phản hồi khách hàng dưới 30 giây

Một khách hàng nhắn tin hỏi về đơn hàng lúc 21 giờ và chờ tám phút mà chưa có ai trả lời. Với doanh nghiệp vừa và nhỏ, tình huống này không hiếm: nhân viên chăm sóc khách hàng quá tải vì câu hỏi lặp lại, các chatbot theo kịch bản trả lời cứng nhắc, còn khách hàng thì không chờ lâu. BizChat AI, vừa ra mắt, là nền tảng AI Chatbot cho doanh nghiệp, được xây dựng để giải quyết đúng ba vấn đề này: phản hồi tức thì, trả lời dựa trên tài liệu của chính doanh nghiệp và chuyển giao cho nhân viên khi cần.

[[S0-HOÀN-TẤT | ~108 từ]]

## Thời gian phản hồi chậm làm doanh nghiệp mất khách như thế nào

Trong kênh nhắn tin, kỳ vọng của khách hàng được tính bằng phút chứ không phải bằng giờ. Khi một câu hỏi về giá, tồn kho hay thời gian giao hàng nằm chờ quá lâu, cuộc trò chuyện thường kết thúc trước khi nhân viên kịp mở tin nhắn. Đây là vấn đề đầu tiên của các doanh nghiệp bán lẻ, thương mại điện tử và dịch vụ khi tìm cách giảm thời gian phản hồi khách hàng.

Vấn đề thứ hai nằm ở chính đội ngũ nhân viên chăm sóc khách hàng. Phần lớn tin nhắn xoay quanh một nhóm nhỏ câu hỏi quen thuộc như giá, chính sách đổi trả và trạng thái đơn hàng. Nhân viên phải trả lời đi trả lại những nội dung giống nhau, trong khi những trường hợp cần sự thấu hiểu và
```

*(Phản hồi bị ngắt ở đây: chưa có `[[HẾT-BÀI]]`, câu cuối dở dang.)*

### 8.5 Áp dụng Lệnh A và kết quả nối tiếp

**Lệnh gửi (rút gọn):** dán Lệnh A với

```text
<ANCHOR>
Vấn đề thứ hai nằm ở chính đội ngũ nhân viên chăm sóc khách hàng. Phần lớn tin nhắn xoay quanh một nhóm nhỏ câu hỏi quen thuộc như giá, chính sách đổi trả và trạng thái đơn hàng. Nhân viên phải trả lời đi trả lại những nội dung giống nhau, trong khi những trường hợp cần sự thấu hiểu và
</ANCHOR>
Tiến độ: đã hoàn tất S0; đang viết dở S1; còn lại S1–S_CTA.
```

**Phản hồi đúng** (bắt đầu bằng từ ngay sau chữ "và", không lặp, không chào hỏi):

```markdown
kinh nghiệm lại phải chờ đến lượt. Khi lượng tin nhắn tăng nhanh vào các đợt khuyến mãi, tình trạng quá tải gần như không thể tránh khỏi, và chất lượng tư vấn là thứ bị ảnh hưởng đầu tiên.

[[S1-HOÀN-TẤT | ~196 từ]]

## AI Chatbot cho doanh nghiệp hoạt động ra sao
...
```

**Đối chiếu sau khi ghép:** câu hoàn chỉnh là "...những trường hợp cần sự thấu hiểu và kinh nghiệm lại phải chờ đến lượt." Giọng điệu, ngôi kể và thuật ngữ ("khách hàng", "nhân viên chăm sóc khách hàng", "doanh nghiệp") không đổi so với phần trước.

**Ví dụ phản hồi SAI cần loại bỏ:** "Tiếp tục bài viết: và kinh nghiệm lại phải chờ..." (thêm lời dẫn, lặp chữ "và").

---


