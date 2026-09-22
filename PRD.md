# PRD — @NovaAssistantBot: AI Agent Trợ lý cá nhân trong Chat
### Dự án: CHAT-01 | Công ty giả lập: NovaMedia Digital JSC

---

## 1. Pain Point (Vấn đề)

Nhân sự NovaMedia trao đổi công việc chủ yếu qua các group Telegram, chia theo phòng ban (Sales, Account, Media, Creative, Content, Performance, Data/MarTech, Tech/Product, Finance, HR/Admin) và theo từng chiến dịch/khách hàng. Một chiến dịch quảng cáo điển hình phát sinh:

- Brief và yêu cầu thay đổi từ khách hàng (qua Account) rải rác trong nhiều tin nhắn.
- Deadline booking media, deadline duyệt creative, deadline báo cáo performance nằm ở các group khác nhau.
- Lời hứa miệng ("chiều nay gửi báo giá", "mai duyệt xong banner") không được ghi lại có hệ thống.

Hệ quả: task bị trôi, deadline bị bỏ lỡ, khách hàng không hài lòng, và nhân sự phải tự nhớ/tự note thủ công — tốn thời gian và có rủi ro sai sót cao, đặc biệt với các cam kết liên quan trực tiếp đến khách hàng.

| Pain point cụ thể | Bộ phận ảnh hưởng nhiều nhất |
|---|---|
| Quên task được giao qua chat nội bộ | Creative, Content, Performance |
| Bỏ lỡ deadline vì không ai nhắc chủ động | Media, Account |
| Cam kết miệng với khách hàng không được lưu vết | Account, Sales |
| Phải cuộn lại đọc hội thoại dài để tìm thông tin | Tất cả phòng ban |
| Lịch họp/lịch duyệt nằm rải rác nhiều group | Account, Creative, Media |

---

## 2. Mục tiêu (Goal)

Xây dựng AI Agent (`@NovaAssistantBot`) tích hợp vào Telegram, giúp nhân sự NovaMedia:

- Tóm tắt nhanh các hội thoại dài trong group/chat được cho phép.
- Từ hội thoại đã được cấp quyền, tự động phát hiện task và deadline; bản tóm tắt được dùng làm ngữ cảnh hỗ trợ nhưng **không phải nguồn duy nhất** để trích xuất task.
- Tạo nhắc việc (reminder) sau khi người dùng xác nhận, gửi trực tiếp qua **kênh chat riêng (1-1) giữa người dùng và agent**.
- Giảm tỷ lệ bỏ lỡ deadline và cam kết với khách hàng.

> *Ghi chú (theo góp ý mentor): tính năng lịch hẹn/sự kiện (Meeting/Event Extraction, đồng bộ Calendar) tạm thời **không thuộc MVP** — xem mục 21 Roadmap.*

**Chỉ số mục tiêu (định lượng, tham chiếu mục 16):** Task Extraction Precision ≥ 90%, Recall ≥ 85%, Deadline Accuracy ≥ 90%, Hallucinated Task Rate ≤ 5%, Reminder Delivery Success ≥ 99%, và 100% yêu cầu truy cập sai phạm vi phải bị Guardrail chặn.

---

## 3. Đối tượng người dùng (Target Users)

- **Người dùng thường**: toàn bộ nhân sự NovaMedia (~120 người) tham gia các group Telegram công việc — đặc biệt Account, Media, Creative do khối lượng giao tiếp cao và nhiều deadline theo chiến dịch.
- **Admin**: Tech/Product hoặc HR/Admin — quản lý cấu hình bot, quyền truy cập group, giám sát chi phí LLM.

---

## 4. Feature Overview (Tổng quan tính năng)

| Feature | Giải quyết pain point | Requirement chi tiết |
|---|---|---|
| Tóm tắt hội thoại theo yêu cầu | Phải cuộn đọc hội thoại dài | FR-01 |
| Trích xuất task từ hội thoại có đối chiếu nguồn | Quên task giao qua chat | FR-02 |
| Trích xuất deadline | Bỏ lỡ deadline | FR-03 |
| Hỏi lại khi thông tin mơ hồ | Giảm false reminder | FR-04 |
| Xác nhận trước khi tạo (Human-in-the-loop) | Cam kết khách hàng không lưu vết chính xác | FR-05 |
| Tạo reminder, gửi qua kênh chat riêng với agent | Bỏ lỡ deadline | FR-06 |
| Hiển thị danh sách task cá nhân | Task trôi, không tổng hợp được | FR-07 |
| Truy vấn dữ liệu qua lớp bảo vệ (guardrail + tool schema) | Ngăn agent đọc/ghi sai phạm vi dữ liệu | FR-08 |
| Lưu dữ liệu qua hàm xác nhận (không ghi thẳng DB) | Ngăn agent tự ý ghi sai/thiếu kiểm soát | FR-09 |

> *Lịch hẹn/sự kiện (Meeting/Event Extraction) đã được rút khỏi MVP theo góp ý mentor — xem mục 21 Roadmap.*

---

## 5. Luồng chính (Main Flow)

```
Người dùng thêm @NovaAssistantBot vào group / chat riêng
        → Cấp quyền đọc hội thoại (opt-in theo group)
        → Người dùng gọi lệnh (VD: /tomtat, /task); proactive detection để sau MVP
        → [Guardrail layer] Kiểm tra quyền trước mọi thao tác đọc dữ liệu
        → Agent gọi tool lấy các message được phép đọc + metadata (message_id, sender, timestamp)
        → Nhánh A: FR-01 tạo bản tóm tắt để người dùng đọc nhanh
        → Nhánh B: FR-02/FR-03 trích task + deadline từ source messages, dùng summary làm context hỗ trợ
        → Source verification: mỗi task candidate phải tham chiếu được message nguồn
        → Nếu thiếu/mơ hồ/mâu thuẫn → Agent hỏi lại (FR-04)
        → Agent tạo TaskCandidate + đề xuất reminder draft; chưa tạo Task chính thức
        → Người dùng Confirm / Edit / Reject qua DM 1-1 (FR-05)
        → Nếu Confirm/Edit: gọi save_confirmed_task(...) và save_confirmed_reminder(...) (FR-09)
        → Scheduler gửi reminder qua DM theo thời điểm đã xác nhận (FR-06)
        → Cập nhật Task Memory / Audit Log; không tự lưu thêm dữ liệu ngoài schema cho phép
        → Người dùng xem danh sách task qua Telegram hoặc dashboard (FR-07)
```

---

## 6. Scope / Non-goals

### Trong phạm vi MVP
- Đăng nhập/xác thực, ≥2 vai trò (người dùng thường, admin).
- Agent tóm tắt hội thoại **khi được yêu cầu** (reactive, qua lệnh Telegram).
- Trích xuất task/deadline từ **source messages đã được cấp quyền**, có thể dùng bản tóm tắt làm ngữ cảnh hỗ trợ; mọi task candidate phải giữ tham chiếu tới message nguồn để truy vết.
- Tạo reminder có xác nhận (Human-in-the-loop), **gửi qua kênh chat riêng (1-1) giữa người dùng và agent** — không gửi trong group gốc.
- Hiển thị danh sách task cá nhân (trong app web hoặc qua lệnh bot).
- Memory cơ bản theo 3 lớp: **session context** (ngắn hạn), **task memory** (task đã xác nhận), và **semantic memory** (thông tin đã được phép giữ để tìm lại); không mặc định lưu toàn bộ hội thoại thô.
- **Lớp bảo vệ (guardrail)** kiểm soát quyền truy cập trước khi agent đọc dữ liệu (group/DB) — agent không được tự do truy cập ngoài phạm vi cho phép.
- **Hàm truy vấn DB có mô tả rõ (tool schema)** để agent lựa chọn đúng hàm khi cần lấy dữ liệu, thay vì agent tự sinh câu truy vấn.
- **Hàm lưu dữ liệu đã xác nhận** — mọi thao tác ghi (task, reminder) phải đi qua hàm này; agent tuyệt đối không ghi trực tiếp vào DB.
- Xử lý lỗi cơ bản (tool gọi thất bại, LLM timeout).

### Ngoài phạm vi MVP (Non-goals)
- Agent tự động gửi tin nhắn thay người dùng.
- Agent tự tạo lịch/reminder mà **không** cần xác nhận.
- Agent tự động, liên tục quét toàn bộ lịch sử chat của mọi group mà chưa được cấp quyền rõ ràng.
- Multi-agent orchestration phức tạp (nhiều agent con chuyên biệt theo phòng ban).
- **Trích xuất lịch hẹn/sự kiện (Meeting/Event Extraction) và tích hợp Calendar** — theo góp ý mentor, tạm chưa cần ở giai đoạn này, chuyển xuống Roadmap (mục 21).
- Planning nhiều bước phức tạp kiểu multi-agent/multi-tool dài. MVP vẫn phải có **agent decision loop tối thiểu**: đọc state → chọn tool/clarification/confirmation → quan sát kết quả → quyết định bước tiếp theo.
- Phân biệt tự động "cam kết nội bộ" vs "cam kết với khách hàng" bằng NLU nâng cao (để ở roadmap sau MVP).

### Điểm cộng (mở rộng sau MVP)
- Agent chủ động (proactive) phát hiện cam kết/lịch hẹn ngay khi tin nhắn tới, gợi ý tạo reminder.
- Trích xuất lịch hẹn/sự kiện + đồng bộ Google Calendar 2 chiều.
- Dashboard "inbox nhiệm vụ" ưu tiên theo mức khẩn cấp/khách hàng.
- Cảnh báo khi vượt hạn mức token/chi phí LLM.
- Đánh giá độ chính xác trích task trên bộ test (benchmark nội bộ).

---

## 8–9. Functional Requirements & Acceptance Criteria

### FR-01 — Conversation Summarization
- **Input**: Source messages theo khoảng thời gian hoặc số lượng tin nhắn chỉ định trong group/chat đã được cấp quyền, kèm `message_id`, `sender_id`, `timestamp`.
- **Expected behavior**: Agent tóm tắt nội dung chính, ưu tiên quyết định, thay đổi yêu cầu, deadline và người phụ trách. Summary phục vụ người dùng đọc nhanh và làm context hỗ trợ cho agent, nhưng **không được dùng làm nguồn duy nhất để FR-02/FR-03 trích xuất task/deadline**.
- **Output**: Bản tóm tắt ngắn gọn; các ý quan trọng liên quan task/deadline nên giữ reference tới message nguồn khi có thể.
- **Điều kiện lỗi**: Nếu đoạn hội thoại quá ngắn hoặc không có nội dung đáng tóm tắt → agent phản hồi rõ "không có nội dung cần tóm tắt" thay vì tạo tóm tắt rỗng/vô nghĩa.

**Acceptance Criteria:**
- Với đoạn hội thoại ≥ 20 tin nhắn trong group "Chiến dịch ABC – Account", agent trả về summary ngắn hơn dữ liệu gốc và không làm mất các quyết định/deadline được dùng để tạo task candidate.
- Một task/deadline **không được tạo chỉ vì xuất hiện trong summary** nếu không xác minh được bằng source message hợp lệ.
- Agent chỉ tóm tắt group mà người dùng đã cấp quyền; nếu gọi lệnh ở group chưa cấp quyền → Guardrail từ chối trước khi dữ liệu được đưa vào LLM (xem FR-08).

---

### FR-02 — Task Extraction & Source Verification
- **Input**: Source messages đã qua Guardrail + summary từ FR-01 (summary chỉ là context hỗ trợ), kèm metadata `message_id`, `sender_id`, `timestamp`, `chat_id`.
- **Expected behavior**: System SHALL identify actionable task candidates và trích xuất (nếu có):
  - `action`
  - `assignee_id | null`
  - `deadline | null`
  - `confidence`
  - `source_message_ids[]`
  - `source_context`
- **Source verification**: mỗi task candidate phải có ít nhất một source message hợp lệ chứng minh task thực sự xuất hiện trong hội thoại; agent không được tạo task chỉ từ suy diễn của summary/memory.
- **Output**: Danh sách `TaskCandidate` chưa phải Task chính thức và chưa có side effect.
- **Điều kiện lỗi**: Nếu không đủ bằng chứng để xác định actionable task → không tạo candidate; nếu action rõ nhưng assignee/deadline mơ hồ → giữ `null`/confidence thấp và chuyển FR-04.

**Acceptance Criteria:**
- Với source message `"Mai 8h tối gửi slide cho anh nhé."` tại timestamp T:
  - `action = gửi slide`
  - `deadline = 20:00 ngày kế tiếp so với T` theo timezone user
  - `source_message_ids` chứa ID của tin nhắn trên.
- Nếu summary ghi sai/thiếu so với source message, source message là căn cứ ưu tiên.
- Nếu không xác định được assignee → `assignee_id = null`; **không tự gán cho người gọi lệnh**.
- Agent không tạo Task/Reminder chính thức trước khi người dùng xác nhận (FR-05).

---

### FR-03 — Deadline Extraction & Normalization
- **Input**: Task candidate + source message(s) liên quan + `message_timestamp` + timezone của user. Đây là input chuẩn; không parse deadline chỉ từ summary nếu source message còn khả dụng.
- **Expected behavior**: Agent chuẩn hoá mốc thời gian tương đối/tuyệt đối (VD: "mai", "cuối tuần", "17h thứ 5", "trước 25/10") thành timestamp hoặc time range theo timezone user (mặc định Asia/Bangkok / GMT+7 cho NovaMedia).
- **Output**: `normalized_deadline`, `confidence`, `requires_clarification`, `deadline_source_message_id`.
- **Điều kiện lỗi**: Nếu mốc thời gian mơ hồ → không ép thành một timestamp giả; đặt `requires_clarification = true` và kích hoạt FR-04.

**Acceptance Criteria:**
- `"chiều thứ 5"` → trả về time range của thứ 5 phù hợp ngữ cảnh (VD: 13:00–18:00) + `requires_clarification = true` trước khi lên reminder chính xác.
- `"mai 8h tối"` phải được tính tương đối từ `message_timestamp`, không từ thời điểm LLM xử lý nếu hai thời điểm khác ngày.
- Nếu deadline bị thay đổi ở message mới hơn trong cùng thread/context → candidate dùng deadline mới nhất nhưng lưu reference tới cả message cũ và mới để audit.

---

> **Lưu ý:** FR "Meeting/Event Extraction" (trích xuất lịch hẹn/sự kiện) đã được **rút khỏi MVP** theo góp ý mentor — Calendar tạm thời chưa cần ở giai đoạn này. Xem mục 21 (Roadmap) để biết đặc tả dự kiến khi triển khai giai đoạn mở rộng.

---

### FR-04 — Clarification (Hỏi lại)
- **Input**: Kết quả trích xuất có confidence dưới ngưỡng quy định hoặc thiếu `assignee/deadline`, có mâu thuẫn, hoặc source verification chưa đủ rõ.
- **Expected behavior**: Agent chủ động hỏi lại người dùng bằng câu hỏi cụ thể, đóng (dễ trả lời), không hỏi chung chung.
- **Output**: Câu hỏi làm rõ, chờ phản hồi trước khi tiếp tục pipeline.

**Acceptance Criteria:**
- Agent không tạo bất kỳ task/reminder nào ở trạng thái confidence thấp mà chưa hỏi lại.
- Câu hỏi làm rõ phải nêu rõ phần thông tin còn thiếu (VD: "Bạn muốn deadline là thứ 7 hay chủ nhật tuần này?"), không hỏi chung chung kiểu "Bạn có thể nói rõ hơn không?".

---

### FR-05 — User Confirmation (Human-in-the-loop)
- **Input**: `TaskCandidate` đã đủ rõ + reminder draft (nếu có deadline). Reminder draft được đề xuất theo policy ở FR-06 trước khi hiển thị confirmation.
- **Expected behavior**: Agent hiển thị một bản nháp đầy đủ gồm action, assignee, deadline, source reference và thời điểm nhắc đề xuất; chờ người dùng **Confirm / Edit / Reject** qua Telegram DM 1-1.
- **Output**: `Confirmation` với decision `accepted / edited / rejected`. Khi accepted/edited → agent mới được gọi FR-09 để tạo Task/Reminder chính thức.

**Acceptance Criteria:**
- Không có Task/Reminder chính thức nào được tạo hoặc lên lịch nếu chưa có `Confirmation` accepted/edited.
- Người dùng từ chối → candidate chuyển `rejected`; không tạo Task/Reminder và không tự đề xuất lại cùng candidate.
- Người dùng chỉnh sửa action/deadline/reminder time → `confirmed_payload` phải phản ánh đúng dữ liệu đã chỉnh sửa; save function không được dùng payload cũ.
- Confirmation UI phải hiển thị ít nhất: action, deadline (nếu có), reminder time (nếu có), và nguồn chat/message để user kiểm tra.

---

### FR-06 — Reminder Proposal, Scheduling & Delivery
- **Input**: `TaskCandidate` trước confirmation để tạo reminder draft; sau FR-05, chỉ payload đã accepted/edited mới được đưa vào save/scheduler.
- **Expected behavior**:
  1. Nếu candidate có deadline, agent/policy tạo **reminder proposal** trước khi FR-05 hiển thị confirmation.
  2. Policy Engine kiểm tra reminder proposal không ở sau deadline, không ở quá khứ và tuân thủ giới hạn hệ thống.
  3. User có thể sửa reminder time trong FR-05; thời điểm sau chỉnh sửa phải được validate lại.
  4. Sau Confirmation, FR-09 tạo Reminder chính thức; Scheduler chỉ xử lý record này.
  5. Reminder chỉ gửi vào DM 1-1 giữa user và bot, không gửi vào group gốc.
- **Default policy của MVP** (có thể cấu hình):
  - Deadline > 24 giờ: đề xuất nhắc trước 24 giờ.
  - Deadline từ 4–24 giờ: đề xuất nhắc trước 2 giờ.
  - Deadline ≤ 4 giờ: đề xuất nhắc trước 30 phút.
  - Không có deadline: mặc định chỉ lưu task; user có thể chủ động chọn reminder time.
  - Nếu proposal ở quá khứ/không hợp lệ → yêu cầu user chọn lại, không tự schedule.
- **Output**: Reminder chính thức đã xác nhận, được lưu qua FR-09 và đưa vào scheduler.
- **Điều kiện lỗi**: Nếu gửi thất bại → status `failed`, retry theo policy hệ thống và ghi AuditLog; không silent-fail.

**Acceptance Criteria:**
- Scheduler chỉ nhận Reminder có `confirmation_id` tham chiếu tới Confirmation accepted/edited.
- Với task có deadline: `scheduled_at < deadline`; với task không deadline: `scheduled_at` chỉ tồn tại khi user đã chọn/xác nhận thời điểm nhắc.
- Reminder được gửi đúng DM của user; sai user/sai group là lỗi nghiêm trọng.
- Reminder delivery success trên bộ test tích hợp mục tiêu ≥ 99% (không tính outage bên thứ ba đã xác nhận).
- Nếu gửi thất bại sau retry, hệ thống ghi lỗi và hiển thị trạng thái failed trong lần tương tác/dashboard kế tiếp.

---

### FR-07 — Task List Display
- **Input**: Yêu cầu xem danh sách task (qua lệnh Telegram hoặc giao diện web).
- **Expected behavior**: Hiển thị "Task Inbox" gồm `TaskCandidate` đang chờ xác nhận và `Task` chính thức; sắp xếp theo deadline gần nhất, phân biệt rõ candidate vs confirmed/completed.
- **Output**: Danh sách có thể lọc theo `pending confirmation / confirmed / completed / cancelled` và khoảng thời gian; rejected candidate không xuất hiện mặc định.

**Acceptance Criteria:**
- Task đã bị từ chối (FR-05) không xuất hiện trong danh sách "đã xác nhận".
- Task đã xác nhận phải xuất hiện trên dashboard/Telegram task list trong **≤ 2 giây** sau khi `save_confirmed_task` trả về thành công (trong điều kiện hệ thống bình thường).

---

### FR-08 — Data Access Guardrail & Tool Calling
- **Input**: Yêu cầu của agent muốn đọc dữ liệu (hội thoại, task, memory) để phục vụ tóm tắt/trích xuất/trả lời người dùng.
- **Expected behavior**: Mọi thao tác đọc dữ liệu của agent phải đi qua **lớp bảo vệ (guardrail)** kiểm tra quyền truy cập (group đã opt-in hay chưa, user có quyền xem dữ liệu này không) **trước khi** agent được phép gọi hàm truy vấn. Agent không tự sinh câu truy vấn (query) tự do vào DB — chỉ được chọn và gọi một trong các **hàm truy vấn đã định nghĩa sẵn**, mỗi hàm có mô tả (docstring/schema) rõ ràng để agent (qua tool calling/function calling) chọn đúng hàm phù hợp với nhu cầu.
- **Output**: Kết quả truy vấn đã được lọc theo đúng phạm vi quyền của người dùng gọi lệnh.
- **Điều kiện lỗi**: Nếu agent cố gọi hàm/truy cập ngoài phạm vi quyền → guardrail chặn lại, trả về lỗi permission, agent không được thử cách khác để lách qua.

**Ví dụ tool schema (mô tả sơ bộ để agent lựa chọn):**
```
get_recent_messages(chat_id, since_timestamp, limit)
  → Lấy tin nhắn gần đây trong 1 chat/group MÀ user gọi lệnh có quyền truy cập.
  → Guardrail kiểm tra: chat_id có nằm trong danh sách group đã opt-in bởi user không.

get_user_task_history(user_id, status?)
  → Lấy lịch sử task của chính user đang gọi lệnh (không được lấy của user khác).

search_past_context(user_id, query, top_k)
  → Semantic search trong memory/embedding của chính user đó.
```

**Acceptance Criteria:**
- Agent không thể truy vấn dữ liệu của group/user mà mình không có quyền, kể cả khi được yêu cầu trực tiếp trong hội thoại (VD: người dùng cố tình hỏi "cho tôi xem task của group khác").
- Mỗi lần agent gọi hàm truy vấn, hệ thống ghi AuditLog: tool name, user_id, resource scope, tham số đã redacted khi cần, kết quả allowed/denied, timestamp.
- Bộ test security phải đạt **100% deny** với các test case cố truy cập user/group không có quyền.

---

### FR-09 — Confirmed-Data Save Function
- **Input**: Payload đã được người dùng xác nhận (FR-05) từ `TaskCandidate`/reminder draft.
- **Expected behavior**: Agent **không được phép ghi trực tiếp vào database**. Mọi thao tác lưu phải gọi qua một **hàm lưu đã định nghĩa sẵn** (VD: `save_confirmed_task(...)`, `save_confirmed_reminder(...)`), hàm này chịu trách nhiệm validate dữ liệu đầu vào, áp dụng đúng schema, và thực hiện ghi DB thay cho agent.
- **Output**: Bản ghi `Task`/`Reminder` chính thức được lưu thành công hoặc trả về lỗi validate rõ ràng. `TaskCandidate` chưa xác nhận không được coi là Task chính thức.
- **Điều kiện lỗi**: Nếu dữ liệu không hợp lệ (thiếu field bắt buộc, sai định dạng) → hàm lưu từ chối và trả lỗi cụ thể cho agent, agent thông báo lại cho người dùng, không tự "sửa" dữ liệu để cố ghi cho bằng được.

**Acceptance Criteria:**
- Không tồn tại đường dẫn code nào cho phép agent (hoặc LLM output) ghi thẳng vào DB mà bỏ qua hàm lưu này.
- Hàm lưu từ chối payload không có `confirmation_id` hợp lệ ở trạng thái accepted/edited — đảm bảo Human-in-the-loop được enforce ở tầng dữ liệu, không chỉ ở tầng logic agent.

---

## 13. Privacy & Security Requirements

- Agent **chỉ xử lý** các group/chat mà người dùng (hoặc admin group) đã cấp quyền rõ ràng (opt-in), không tự động quét mọi group bot được thêm vào.
- Lưu ý đặc thù Telegram: bot không có khả năng truy cập "Secret Chat" (chat E2E thật của Telegram); các group/chat thường mà bot hoạt động vốn không phải E2E ở tầng client — do đó cam kết bảo mật tập trung vào: **hạn chế tối đa dữ liệu lưu trữ, mã hoá dữ liệu lưu trữ (at-rest), giới hạn quyền truy cập nội bộ, không log nội dung thô không cần thiết**.
- Áp dụng nguyên tắc data minimization: chỉ lưu phần trích xuất cần thiết (task, deadline, tóm tắt), hạn chế lưu nguyên văn tin nhắn gốc trừ khi cần cho việc truy vết (kèm thời gian retention rõ ràng).
- **Retention MVP**: raw message cache chỉ giữ khi cần cho source verification và có TTL cấu hình (đề xuất mặc định 7 ngày trong môi trường demo); Task/Confirmation/Audit giữ theo vòng đời project/pilot. Khi group bị revoke, agent mất quyền truy vấn ngay; dữ liệu derived từ group đó phải được đánh dấu inaccessible và đưa vào quy trình xoá theo retention policy.
- Phạm vi permission của bot phải tương ứng đúng với nhu cầu (chỉ đọc tin nhắn nhóm đã bật quyền, không truy cập chat riêng người dùng trừ khi được yêu cầu trực tiếp).
- Người dùng/admin group phải có khả năng thu hồi quyền với từng group bất kỳ lúc nào; revoke phải có hiệu lực authorization ngay lập tức, kể cả với semantic search/memory.
- **Guardrail layer là bắt buộc** — mọi lượt đọc dữ liệu của agent (qua tool calling) phải được lớp này kiểm tra quyền trước khi thực thi (chi tiết FR-08); agent không có đường nào để bỏ qua lớp này.
- **Không ghi trực tiếp vào DB** — agent chỉ được lưu dữ liệu thông qua hàm lưu đã xác nhận (FR-09), đảm bảo mọi bản ghi đều đã qua validate và đúng trạng thái "đã xác nhận" trước khi tồn tại trong hệ thống.

---

## 16. Success Metrics / Evaluation

### 16.1 Quality metrics cho MVP

| Metric | Mục tiêu MVP | Cách đo |
|---|---:|---|
| Task Extraction Precision | ≥ 90% | Candidate đúng / tổng candidate agent tạo |
| Task Extraction Recall | ≥ 85% | Task đúng agent phát hiện / tổng task thật trong bộ test |
| Deadline Accuracy | ≥ 90% | Deadline normalize đúng / tổng case có deadline rõ |
| Hallucinated Task Rate | ≤ 5% | Candidate không có bằng chứng source / tổng candidate |
| Missed Task Rate | ≤ 15% | Task thật bị bỏ sót / tổng task thật |
| Reminder Delivery Success | ≥ 99% | Reminder sent đúng user/đúng thời điểm cửa sổ cho phép |
| Unauthorized Access Block Rate | 100% | Test case sai quyền bị deny / tổng test case sai quyền |
| P95 Response Latency | ≤ 5 giây | Với request ≤ 100 messages |
| Dashboard Sync Latency | ≤ 2 giây | Từ save thành công đến khi task hiển thị |

### 16.2 Product metrics (pilot)

- **Acceptance rate** của Task/Reminder proposal: theo dõi để đánh giá usefulness; mục tiêu pilot ban đầu ≥ 60%.
- **Clarification rate**: theo dõi theo loại ambiguity; không dùng một ngưỡng cứng duy nhất vì hỏi lại đúng lúc tốt hơn tự đoán sai.
- **Cost/user/day** và **tokens/request**: đo thực tế trong pilot để chốt ngân sách scale.
- **Duplicate task rate**: theo dõi task bị tạo trùng sau retry/re-processing; mục tiêu ≤ 2%.

### 16.3 Benchmark dataset

Bộ test MVP tối thiểu nên có các nhóm case: task rõ ràng, không có task, nhiều task trong một message, deadline tương đối, deadline bị sửa ở message sau, không rõ assignee, task bị hủy, mâu thuẫn, prompt injection/yêu cầu đọc sai quyền, và summary bỏ sót chi tiết nhưng source message vẫn chứa thông tin.
---

## 21. Roadmap sau MVP

- **Meeting/Event Extraction** — trích xuất lịch hẹn/sự kiện từ hội thoại (đặc tả FR chi tiết sẽ bổ sung khi bắt đầu giai đoạn này; tái sử dụng logic clarification/confirmation đã có ở MVP).
- **Đồng bộ Google Calendar 2 chiều** — gắn liền với Meeting/Event Extraction ở trên.
- Agent chủ động (proactive) phát hiện cam kết ngay khi tin nhắn tới.
- Phân loại tự động "cam kết nội bộ" vs "cam kết khách hàng" bằng NLU nâng cao.
- Dashboard "inbox nhiệm vụ" ưu tiên theo khách hàng/mức khẩn cấp.
- Cảnh báo khi vượt hạn mức token/chi phí LLM (nếu chưa kịp đưa vào MVP).
- Mở rộng multi-agent theo phòng ban (nếu thực sự cần thiết, hiện đang là non-goal).
- Planning nhiều bước phức tạp hơn (multi-step tool orchestration thực sự) nếu nhu cầu sản phẩm mở rộng vượt ngoài phạm vi "lên lịch nhắc việc".

---
