# PRD — AI Agent Trợ lý cá nhân trong Chat

## Vấn đề

Người dùng nhận nhiều tin nhắn mỗi ngày nên dễ bỏ sót task, lịch hẹn hoặc các việc đã hứa trong hội thoại.

## Mục tiêu

Xây dựng AI Agent có thể đọc các đoạn chat được cho phép, tóm tắt nội dung chính, phát hiện việc cần làm/lịch hẹn và hỗ trợ tạo nhắc việc.

## Người dùng chính

Người thường xuyên trao đổi công việc qua chat, tham gia nhiều nhóm và có nhiều task phát sinh từ tin nhắn.

## Luồng chính

Tin nhắn → Người dùng cho phép agent truy cập đoạn chat → Agent phân tích → Tóm tắt → Trích task/lịch → Hỏi lại nếu chưa rõ → Người dùng xác nhận → Tạo reminder.

## MVP

* Tóm tắt hội thoại.
* Phát hiện task và deadline.
* Phát hiện lịch hẹn.
* Hỏi lại khi thông tin mơ hồ.
* Tạo reminder sau khi người dùng xác nhận.
* Hiển thị danh sách việc cần làm.

## Nguyên tắc

* Không tự tạo lịch hoặc reminder khi chưa được xác nhận.
* Chỉ đọc những hội thoại người dùng cho phép.
* Hạn chế lưu nội dung chat không cần thiết.

## Ví dụ

Tin nhắn: "Mai 8h tối gửi slide cho anh nhé."

Agent hiểu:

* Task: Gửi slide.
* Deadline: 20:00 ngày mai.
* Sau đó hỏi người dùng có muốn tạo reminder hay không.

## Kết quả mong muốn

Giúp người dùng không bỏ sót việc quan trọng và giảm thời gian phải đọc lại, ghi nhớ hoặc tự tạo nhắc việc.
