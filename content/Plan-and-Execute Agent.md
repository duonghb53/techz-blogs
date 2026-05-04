# Plan-and-Execute Agent

_Tác giả: Amit Shekhar | Ngày đăng: 4 tháng 5, 2026_

---
![[Pland and Execute.png]]

Trong bài này, chúng ta sẽ tìm hiểu về Plan-and-Execute Agent — nó là gì, cấu trúc bên trong, cách nó lập kế hoạch và thực thi từng bước, điểm khác biệt so với ReAct Agent, và cách xử lý các lỗi phổ biến.

Nội dung bao gồm:

- Plan-and-Execute Agent là gì
- Plan-and-Execute Agent vs AI Agent
- Cấu trúc của một Plan-and-Execute Agent
- Cách Plan-and-Execute Agent hoạt động
- Ví dụ trace đầy đủ
- Plan-and-Execute Agent vs ReAct Agent
- Các lỗi phổ biến và cách khắc phục
- Tóm tắt nhanh

---

## Plan-and-Execute Agent là gì

**Plan-and-Execute Agent** là một **AI Agent** được xây dựng theo pattern **Plan-and-Execute**. Trong pattern này, agent trước tiên viết toàn bộ kế hoạch ngay từ đầu, sau đó thực thi kế hoạch từng bước một. Nó không tự tìm ra bước tiếp theo mỗi lượt. Nó suy nghĩ một lần lúc bắt đầu, đi qua kế hoạch, và chỉ lập kế hoạch lại khi thực tế không khớp với kế hoạch.

Phân tích thuật ngữ:

**Plan-and-Execute Agent = Plan (Lập kế hoạch) + Execute (Thực thi) + Agent**

- **Plan** là phần suy nghĩ — một LLM mạnh đọc task của người dùng và viết kế hoạch đầy đủ từng bước để giải quyết.
- **Execute** là phần thực hiện — thường là một LLM nhỏ hơn, rẻ hơn chọn tool cho mỗi bước, và runtime gọi tool đó.
- **Agent** là lớp bọc kết nối Planner và Executor, theo dõi tiến độ, và quyết định khi nào task hoàn thành.

Nói đơn giản:

> **Plan-and-Execute Agent = Planner viết các bước + Executor chạy các bước + Vòng lặp kết nối chúng.**

Hãy nghĩ Plan-and-Execute Agent như một project manager và một worker trong một team. Project manager đọc yêu cầu, viết danh sách task rõ ràng, và bàn giao. Worker nhận từng task, hoàn thành, và báo cáo lại. Nếu mọi thứ lệch hướng, project manager cập nhật kế hoạch. Plan-and-Execute Agent hoạt động y hệt — một phần lập kế hoạch, phần còn lại thực thi.

---

## Plan-and-Execute Agent vs AI Agent

**Plan-and-Execute là một pattern để xây dựng AI Agent. Nó không phải là một danh mục khác.**

- **AI Agent** là danh mục — bất kỳ hệ thống nào có LLM được bọc trong vòng lặp với tool và bộ nhớ. Nó không quy định _cách_ vòng lặp được cấu trúc, _liệu_ agent có lập kế hoạch trước hay không, hay _liệu_ việc lập kế hoạch và thực thi có được tách ra thành các LLM call khác nhau hay không.
    
- **Plan-and-Execute** là một pattern để xây dựng vòng lặp đó, trong đó agent trước tiên tạo ra kế hoạch đầy đủ trong một LLM call, sau đó chạy từng bước của kế hoạch bằng một LLM call khác (hoặc một loạt call). Việc lập kế hoạch được thực hiện một lần lúc bắt đầu. Việc thực thi diễn ra từng bước sau đó.
    

Khi ta nói "Plan-and-Execute Agent", ta có nghĩa là AI Agent được xây dựng theo pattern Plan-and-Execute. Các pattern khác như ReAct, Reflection, và Agentic RAG cũng xây dựng AI Agent — chúng chỉ định hình vòng lặp theo cách khác.

> **Plan-and-Execute là một cách để xây dựng AI Agent. Đây là một trong những pattern phổ biến, đặc biệt cho các task nhiều bước.**

---

## Cấu trúc của một Plan-and-Execute Agent

Một Plan-and-Execute Agent có năm phần:

```
+--------------------------------------------------------+
|                     Vòng lặp Agent                     |
|                                                        |
|   Task của User                                        |
|       |                                                |
|       v                                                |
|   +---------+                                          |
|   | Planner |  --->  Kế hoạch: [Bước 1, Bước 2, Bước 3]|
|   +---------+                                          |
|       |                                                |
|       v                                                |
|   +----------+         +-------+                       |
|   | Executor |  <--->  | Tools |                       |
|   +----------+         +-------+                       |
|       |     ^                                          |
|       v     | (kế hoạch cập nhật, nếu cần)             |
|   +------------+                                       |
|   | Re-Planner |                                       |
|   +------------+                                       |
|       |                                                |
|       v                                                |
|   Final Answer (khi task hoàn thành)                   |
+--------------------------------------------------------+
```

**1. Planner.** Đây là LLM viết kế hoạch. Nó đọc task của người dùng và tạo ra danh sách các bước để giải quyết. Kế hoạch thường là danh sách đánh số, mỗi bước là một hướng dẫn nhỏ, rõ ràng. Planner tốt chia task thành các bước mà Executor có thể chạy từng cái một.

**2. Executor.** Đây là phần chạy từng bước của kế hoạch. LLM bên trong Executor thường nhỏ và rẻ hơn Planner, vì mỗi bước hẹp hơn nhiều so với task đầy đủ. LLM chọn tool cho bước hiện tại, runtime gọi tool, và kết quả được lưu vào bộ nhớ. Trong một số thiết lập, Executor tự nó là một sub-agent nhỏ như vòng lặp ReAct. Executor không lập kế hoạch trước — nó chỉ xử lý bước hiện tại rồi tiếp tục.

**3. Tools.** Đây là các hành động Executor có thể thực hiện — tìm kiếm web, truy vấn database, chạy máy tính, gửi email, đọc file, v.v. Mỗi tool có tên, mô tả, và input schema.

**4. Memory.** Đây là lịch sử chạy của agent — task gốc, kế hoạch, mọi bước đã thực thi, và mọi kết quả. Planner đọc nó khi sửa đổi kế hoạch. Executor đọc nó để biết những gì đã được làm.

**5. Re-Planner.** Đây là LLM quyết định làm gì sau khi mỗi bước được thực thi. Nó kiểm tra kết quả, so sánh với kế hoạch, và quyết định một trong ba việc — tiếp tục với bước tiếp theo, cập nhật kế hoạch nếu có gì đó thay đổi, hoặc dừng vì task đã hoàn thành. Trong một số thiết lập, Re-Planner là cùng LLM với Planner, chỉ được gọi lại với trạng thái mới nhất.

Cả năm phần hoạt động cùng nhau. Bốn phần đầu — Planner, Executor, Tools, và Memory — là các phần bắt buộc. Re-Planner là tùy chọn trong các thiết lập đơn giản nhất, nhưng hầu hết agent production đều có nó để xử lý các trường hợp thực tế không khớp với kế hoạch.

---

## Cách Plan-and-Execute Agent hoạt động

Luồng gồm ba giai đoạn — Plan, Execute, và Re-Plan.

**Giai đoạn 1: Plan.** Người dùng giao task cho agent. Planner đọc task và viết kế hoạch từng bước. Kế hoạch được lưu vào bộ nhớ.

**Giai đoạn 2: Execute.** Executor chọn bước đầu tiên của kế hoạch. LLM đề xuất tool phù hợp, runtime gọi nó, đọc kết quả, và lưu kết quả vào bộ nhớ. Sau đó chuyển sang bước tiếp theo.

**Giai đoạn 3: Re-Plan.** Sau mỗi bước (hoặc sau vài bước), Re-Planner đọc trạng thái hiện tại. Nếu mọi thứ đúng hướng, Executor tiếp tục. Nếu một bước thất bại, trả về kết quả không mong đợi, hoặc làm phần còn lại của kế hoạch không còn hợp lệ, Re-Planner cập nhật kế hoạch. Nếu task xong, agent dừng và trả về câu trả lời cuối cùng.

Nói đơn giản:

> **Lập kế hoạch một lần. Thực thi từng bước. Lập kế hoạch lại khi thực tế không khớp với kế hoạch.**

---

## Ví dụ trace đầy đủ

Giả sử người dùng giao cho agent task:

> "Tìm dân số hiện tại của Ấn Độ và Nhật Bản, và cho tôi biết nước nào có nhiều người hơn và hơn bao nhiêu."

**Giai đoạn 1: Plan.**

Planner viết kế hoạch:

```
1. Tìm kiếm dân số hiện tại của Ấn Độ.
2. Tìm kiếm dân số hiện tại của Nhật Bản.
3. So sánh hai con số.
4. Trả về nước có dân số lớn hơn và chênh lệch.
```

**Giai đoạn 2: Execute.**

**Bước 1.** Executor gọi search tool với query `"dân số hiện tại của Ấn Độ"`. Tool trả về `1.43 tỷ`. Kết quả được lưu vào bộ nhớ.

**Bước 2.** Executor gọi search tool với query `"dân số hiện tại của Nhật Bản"`. Tool trả về `124 triệu`. Kết quả được lưu vào bộ nhớ.

**Bước 3.** Executor gọi calculator tool với `1.430.000.000 - 124.000.000`. Tool trả về `1.306.000.000`. Kết quả được lưu vào bộ nhớ.

**Bước 4.** Executor đọc bộ nhớ và tạo ra câu trả lời cuối cùng.

**Giai đoạn 3: Re-Plan.**

Sau mỗi bước, Re-Planner kiểm tra trạng thái. Mọi thứ đúng hướng nên không thay đổi kế hoạch. Sau Bước 4, Re-Planner thấy task đã xong. Agent dừng.

**Final Answer:**

> "Ấn Độ có nhiều người hơn Nhật Bản khoảng 1.31 tỷ người. Ấn Độ có 1.43 tỷ người, và Nhật Bản có 124 triệu người."

Ta thấy agent không tìm ra từng bước một lần lượt. Nó lập kế hoạch toàn bộ hành trình trước, rồi đi qua từng bước.

---

## Plan-and-Execute Agent vs ReAct Agent

**ReAct Agent** và Plan-and-Execute Agent là hai pattern khác nhau để xây dựng AI Agent. Cả hai đều phổ biến. Điểm khác biệt là _khi nào_ việc suy nghĩ xảy ra.

- **ReAct Agent** suy nghĩ từng bước một. Ở mỗi lượt, nó viết Thought, chọn Action, đọc Observation, và suy nghĩ lại. Nó không lập kế hoạch trước. Nó quyết định bước tiếp theo chỉ sau khi thấy kết quả mới nhất.
    
- **Plan-and-Execute Agent** suy nghĩ một lần lúc bắt đầu. Nó viết toàn bộ kế hoạch trước, sau đó chạy từng bước. Nó chỉ lập kế hoạch lại nếu có gì đó sai.
    

```
ReAct Agent - LLM suy nghĩ ở mỗi bước:

  +---------+   +--------+   +-------------+
  | Thought |-->| Action |-->| Observation |--+
  +---------+   +--------+   +-------------+  |
       ^                                      |
       +-- (vòng lặp đến final answer) -------+


Plan-and-Execute Agent - LLM suy nghĩ một lần, sau đó chạy:

  +------+   +--------+   +--------+   +--------+
  | Plan |-->| Bước 1 |-->| Bước 2 |-->| Bước 3 |--> Final Answer
  +------+   +--------+   +--------+   +--------+
                              ^
                              +-- (re-plan chỉ khi một bước thất bại)
```

ReAct Agent quay lại LLM ở mỗi bước, trong khi Plan-and-Execute Agent gọi LLM một lần để lập kế hoạch rồi chạy từng bước tuần tự. ReAct Agent trả chi phí suy nghĩ ở mỗi lượt. Plan-and-Execute Agent trả chi phí suy nghĩ một lần lúc đầu.

|Thuộc tính|ReAct Agent|Plan-and-Execute Agent|
|---|---|---|
|Khi nào LLM lập kế hoạch|Từng bước một|Toàn bộ kế hoạch trước, re-plan chỉ khi cần|
|LLM call mỗi task|Một LLM call mỗi bước|Một call lập kế hoạch lớn + call nhỏ hơn mỗi bước|
|Chi phí|Cao hơn khi cần nhiều bước|Thấp hơn cho task có nhiều bước có thể đoán trước|
|Độ trễ|Chậm hơn cho task dài (nhiều call lớn)|Nhanh hơn cho task dài (Executor call có thể nhỏ hơn)|
|Khả năng thích ứng|Rất cao — thích ứng ở mỗi lượt|Thấp hơn — thích ứng chỉ khi re-planning|
|Phù hợp nhất cho|Task mở khi bước tiếp theo chưa rõ|Task nhiều bước với con đường rõ ràng đến câu trả lời|
|Khả năng quan sát|Suy luận hiển thị ở mỗi lượt|Toàn bộ kế hoạch hiển thị ngay từ đầu|

> **Dùng ReAct khi bước tiếp theo phụ thuộc vào kết quả cuối cùng. Dùng Plan-and-Execute khi các bước có thể được lập kế hoạch trước.**

---

## Các lỗi phổ biến và cách khắc phục

**1. Kế hoạch tệ.** Planner viết kế hoạch không đầy đủ, sai thứ tự, hoặc bỏ qua các bước quan trọng. Toàn bộ quá trình chạy lệch hướng từ đầu.

_Khắc phục:_ Cho Planner một system prompt rõ ràng với ví dụ về kế hoạch tốt, liệt kê các tool có sẵn, và yêu cầu nó viết các bước nhỏ, cụ thể thay vì mơ hồ.

**2. Step Drift (Trôi bước).** Executor đọc bước như "tìm kiếm tài liệu về retry policy" và chạy tìm kiếm web công khai thay vào đó. Kết quả kỹ thuật là một tìm kiếm, nhưng không phù hợp với ý định của Planner.

_Khắc phục:_ Làm mỗi bước trong kế hoạch cụ thể — bao gồm tên tool chính xác, input chính xác, và output mong đợi. Bước càng nhỏ và rõ ràng, Executor càng ít có thể drift.

**3. Không Re-Planning.** Một bước thất bại hoặc trả về điều bất ngờ, nhưng agent tiếp tục chạy phần còn lại của kế hoạch một cách mù quáng. Câu trả lời cuối cùng sai.

_Khắc phục:_ Luôn chạy Re-Planner sau mỗi bước, hoặc ít nhất sau bất kỳ bước nào trả về lỗi hoặc kết quả không mong đợi.

**4. Over-Planning (Lập kế hoạch quá mức).** Planner viết kế hoạch khổng lồ với 30 bước cho task chỉ cần 3. Mỗi bước thừa tốn một LLM call và một tool call.

_Khắc phục:_ Nói với Planner viết kế hoạch nhỏ nhất có thể giải quyết task và gộp các bước có thể làm cùng nhau.

**5. Kế hoạch trở nên lỗi thời.** Kế hoạch đúng lúc đầu, nhưng sau vài bước, tình huống đã thay đổi — một tìm kiếm trả về thông tin mới, một tool trả về dữ liệu ở định dạng khác, hoặc một bước tiết lộ rằng một giả định sai. Phần còn lại của kế hoạch không còn phù hợp.

_Khắc phục:_ Re-Planner phải viết lại các bước còn lại dựa trên trạng thái mới nhất, không chỉ tiếp tục với kế hoạch gốc.

**6. Infinite Re-Planning (Lập kế hoạch lại vô hạn).** Re-Planner liên tục viết lại kế hoạch mà không tiến triển. Agent lặp mãi.

_Khắc phục:_ Đặt số lần re-plan tối đa, số bước tổng tối đa, và điều kiện dừng kích hoạt khi kế hoạch không thay đổi trong hai lần re-plan liên tiếp.

---

## Tóm tắt nhanh

- **Plan-and-Execute Agent** là AI Agent được xây dựng theo pattern Plan-and-Execute — lập kế hoạch toàn bộ task trước, sau đó thực thi kế hoạch từng bước.
- **Plan-and-Execute Agent = Plan + Execute + Agent** — Planner viết kế hoạch, Executor chạy từng bước, và vòng lặp Agent kết nối chúng.
- **Cấu trúc** có năm phần — Planner, Executor, Tools, Memory, và Re-Planner.
- **Luồng** có ba giai đoạn — Lập kế hoạch một lần, Thực thi từng bước, Re-Plan khi thực tế không khớp với kế hoạch.
- **Plan-and-Execute vs ReAct** — ReAct suy nghĩ từng bước một, Plan-and-Execute suy nghĩ một lần trước. Dùng ReAct cho task mở. Dùng Plan-and-Execute cho task nhiều bước với con đường rõ ràng.
- **Các lỗi phổ biến** là Kế hoạch tệ, Step Drift, Không Re-Planning, Over-Planning, Kế hoạch lỗi thời, và Infinite Re-Planning — tất cả có thể xử lý với system prompt rõ ràng, bước nhỏ, và Re-Planner nghiêm ngặt.
- **Plan-and-Execute** là một trong những pattern phổ biến nhất để xây dựng AI Agent cần xử lý các task dài, nhiều bước một cách đáng tin cậy.

---

_Nguồn: [Plan-and-Execute Agent - Outcome School](https://outcomeschool.com/blog/plan-and-execute-agent) by Amit Shekhar_