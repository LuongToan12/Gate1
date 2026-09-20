# CHAT-01 — AI Agent Trợ lý cá nhân trong Chat

**1-Page Project Brief**

---

## 1. Vấn đề

Công ty phần mềm có nhiều nhóm chat theo chức năng: DEV 1, DEV 2, BA, Tester, Technical, Support, System Admin, BI. Mỗi nhân viên tham gia nhiều nhóm, nhận rất nhiều tin nhắn mỗi ngày → dễ bỏ lỡ task, deadline, cuộc họp.

## 2. Đối tượng dùng

Bất kỳ nhân viên nào trong công ty. Mỗi người có **1 AI Agent cá nhân riêng**, chỉ đọc các nhóm chat họ tự cấp quyền.

Ví dụ task theo nhóm:

| Nhóm | Task ví dụ |
|---|---|
| DEV 1 / DEV 2 | Fix bug, merge code trước deadline |
| BA | Chốt spec tính năng |
| Tester | Hoàn thành test case |
| Technical | Review kiến trúc hệ thống |
| Support | Phản hồi lỗi production |
| System Admin | Deploy server |
| BI Engineer/Analyst | Gửi báo cáo dashboard |

## 3. Giải pháp

Một AI Agent xuất hiện như "user" trong app chat. Agent:
- Đọc các nhóm được cấp quyền
- Tóm tắt hội thoại dài
- Trích task/lịch hẹn từ tin nhắn
- Nhắc việc, tạo lịch — **luôn xin xác nhận trước khi hành động**
- Hỏi lại khi thông tin chưa rõ
- Ghi nhớ ngữ cảnh qua nhiều lượt chat

## 4. Tính năng chính

- Tóm tắt hội thoại theo yêu cầu
- Trích task/lịch hẹn + tạo reminder có confirm
- Hiển thị task/lịch cá nhân
- Memory ngữ cảnh
- Hỏi lại khi thông tin mơ hồ

## 5. Ràng buộc quan trọng

- **Không tự hành động:** phải confirm trước khi tạo/gửi
- **Bảo mật:** chỉ đọc nhóm được cấp quyền, tôn trọng E2E, không lưu nội dung thô
- **Chính xác:** hạn chế trích task/lịch sai
- **Hiệu năng:** tối ưu tốc độ & chi phí (cache, batch call)

## 6. Công nghệ dùng

LLM (GPT-4o-mini/Claude Haiku) + LangGraph · Qdrant/pgvector · WebSocket · Google Calendar API · FastAPI/NestJS · React/Next.js · Docker, Postgres, Redis

## 7. Tiêu chí hoàn thành

- App deploy online, có login, ≥2 vai trò
- Agent tóm tắt + trích task + nhắc việc có xác nhận
- Hiển thị lịch cá nhân
- Có memory + xử lý lỗi cơ bản

## 8. Deadline

**23:59, 20/09/2026** — nộp Brief, PRD, Wireframe/UI Flow, GitHub Repo + AI Log
