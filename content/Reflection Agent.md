# Reflection Agent

_Tác giả: Amit Shekhar | Ngày đăng: 7 tháng 5, 2026_

---
![[Reflection Agent.png]]

Trong bài viết này, chúng ta sẽ tìm hiểu về **Reflection Agent** — nó là gì, được xây dựng như thế nào, cấu trúc bên trong, cách nó tự tạo, tự đánh giá, và tự chỉnh sửa kết quả của mình, cũng như cách xử lý các lỗi phổ biến.

Nội dung bao gồm:

- Reflection Agent là gì
- Reflection Agent vs AI Agent
- Cấu trúc của một Reflection Agent
- Cách Reflection Agent hoạt động
- Ví dụ trace đầy đủ
- Reflection Agent vs ReAct Agent
- Các lỗi phổ biến và cách khắc phục
- Tóm tắt nhanh

---

# Reflection Agent là gì

**Reflection Agent** là một **AI Agent** được xây dựng theo pattern **Reflection**. Trong pattern này, agent trước tiên viết ra một bản nháp, sau đó tự đọc lại bản nháp của mình và đánh giá nó, rồi viết phiên bản tốt hơn dựa trên phần đánh giá đó. Vòng lặp này tiếp tục cho đến khi kết quả đủ tốt.

Phân tích thuật ngữ:

**Reflection Agent = Reflection + Agent**

- **Reflection** là phần tự đánh giá — agent đọc lại output của chính nó, xác định điểm sai hoặc chưa tốt, rồi viết feedback cho vòng tiếp theo.
- **Agent** là lớp bao quanh LLM, chạy vòng lặp này, theo dõi các bản nháp và critique, và quyết định khi nào dừng.

Nói đơn giản:

> **Reflection Agent = Một Generator tạo bản nháp + Một Critic đánh giá bản nháp + Một vòng lặp chỉnh sửa cho đến khi kết quả đủ tốt.**

Hãy tưởng tượng Reflection Agent như một người viết bài làm việc cùng một editor.

- Người viết tạo bản nháp đầu tiên.
- Editor đọc và chỉ ra điểm yếu, thiếu sót, hoặc chỗ chưa rõ ràng.
- Người viết đọc feedback và viết lại phiên bản tốt hơn.
- Editor tiếp tục đánh giá phiên bản mới.
- Vòng lặp tiếp tục cho đến khi editor không còn gì để sửa nữa.

Reflection Agent hoạt động y hệt như vậy — một phần viết, một phần đánh giá, và cả hai cùng cải thiện chất lượng đầu ra.

Một lần gọi LLM thông thường chỉ tạo ra **một bản nháp rồi dừng**. Reflection Agent thì:

- tạo bản nháp,
- tự đánh giá,
- tự sửa,
- và tiếp tục cải thiện cho đến khi đạt ngưỡng chất lượng mong muốn.

Vì vậy, Reflection Agent đặc biệt hữu ích cho các task mà **bản nháp đầu tiên thường chưa đủ tốt**, và ta muốn agent tự nâng cấp chất lượng trước khi trả kết quả cho người dùng.

---

# Reflection Agent vs AI Agent

Một câu hỏi tự nhiên xuất hiện:

> Nếu Reflection Agent cũng chỉ là LLM chạy trong một vòng lặp, vậy chẳng phải nó chỉ là AI Agent sao?

Câu trả lời là:

> **Reflection là một pattern để xây dựng AI Agent. Nó không phải là một danh mục khác.**

- **AI Agent** là khái niệm rộng — bất kỳ hệ thống nào có LLM được đặt trong vòng lặp cùng tool và memory.
- **Reflection** là một cách cụ thể để tổ chức vòng lặp đó:
  - tạo output,
  - đánh giá output,
  - dùng feedback để tạo phiên bản tốt hơn.

Các pattern khác như:

- ReAct
- Plan-and-Execute
- Agentic RAG

… cũng là các cách khác nhau để xây dựng AI Agent.

Nói ngắn gọn:

> **Reflection là một cách xây dựng AI Agent — đặc biệt phù hợp cho các task mà chất lượng quan trọng hơn tốc độ.**

---

# Cấu trúc của một Reflection Agent

Một Reflection Agent có 5 thành phần chính.

```text
+--------------------------------------------------------+
|                    The Agent Loop                      |
|                                                        |
|   User Task                                            |
|       |                                                |
|       v                                                |
|   +-----------+         +--------+                     |
|   | Generator |  ---->  | Critic |                     |
|   +-----------+         +--------+                     |
|       ^                     |                          |
|       |                     v                          |
|       |               +------------+                   |
|       +-- (revise) -- | Stop Check | -> Final Output   |
|                       +------------+                   |
+--------------------------------------------------------+
```

---

## 1. Generator

Đây là LLM tạo ra bản nháp.

- Nó đọc task của người dùng.
- Tạo phiên bản đầu tiên của output.
- Ở các vòng sau, nó đọc:
  - bản nháp trước,
  - critique,
  - rồi viết phiên bản mới tốt hơn.

---

## 2. Critic

Đây là LLM đánh giá bản nháp.

- Nó đọc draft.
- Chỉ ra:
  - cái gì sai,
  - cái gì thiếu,
  - cái gì có thể tốt hơn.

Critique chính là tín hiệu để Generator cải thiện output.

Trong đa số hệ thống:

- Critic và Generator thực chất là cùng một model,
- chỉ khác prompt.

Tuy nhiên cũng có thể dùng hai model riêng biệt.

---

## 3. Tools

Đây là các công cụ hỗ trợ Critic kiểm chứng output.

Ví dụ:

- web search để kiểm tra fact,
- chạy unit test cho code,
- query database,
- xác minh citation.

Nếu không có tool:

- Critic chỉ dựa vào hiểu biết nội tại của model.

Critic có khả năng:

- chạy code,
- xác minh fact,
- kiểm tra citation

… sẽ mạnh hơn rất nhiều so với Critic chỉ “đoán”.

---

## 4. Memory

Đây là lịch sử hoạt động của agent:

- task gốc,
- mọi draft,
- mọi critique,
- số vòng revision.

Generator đọc memory để biết cần sửa gì.

Critic đọc memory để tránh lặp lại feedback cũ.

Trong thực tế:

- giữ toàn bộ history sẽ làm nổ context window,
- nên thường chỉ giữ:
  - draft mới nhất,
  - critique mới nhất,
  - hoặc summary của các vòng cũ.

---

## 5. Stop Check

Đây là thành phần quyết định khi nào dừng vòng lặp.

Nó có thể dừng khi:

- Critic nói draft đã đủ tốt,
- đạt số vòng tối đa,
- hoặc hai critique liên tiếp giống hệt nhau.

Nếu không có Stop Check:

> agent có thể chạy mãi mãi.

---

Tất cả 5 thành phần phối hợp với nhau để tạo thành Reflection Agent.

---

# Cách Reflection Agent hoạt động

Luồng hoạt động rất đơn giản.

Agent chạy qua 3 giai đoạn:

1. Generate
2. Critique
3. Revise

Sau mỗi Critique sẽ có một bước Stop Check để quyết định:

- dừng,
- hay tiếp tục revise.

```text
+----------+      +----------+      +-------+
| Generate | ---> | Critique | ---> | Stop? | ---> [yes] ---> Final Output
+----------+      +----------+      +-------+
                       ^                |
                       |                v [no]
                       |           +--------+
                       +---<-------| Revise |
                                   +--------+
```

---

## Giai đoạn 1: Generate

- User đưa task.
- Generator tạo bản nháp đầu tiên.
- Draft được lưu vào memory.

---

## Giai đoạn 2: Critique

Critic đọc draft và viết critique:

- sai ở đâu,
- thiếu gì,
- có thể cải thiện thế nào.

Critique được lưu vào memory.

Sau đó Stop Check quyết định:

- draft đủ tốt → dừng,
- đạt max rounds → dừng,
- chưa đủ tốt → chuyển sang Revise.

---

## Giai đoạn 3: Revise

Generator đọc:

- draft mới nhất,
- critique mới nhất,

… rồi tạo phiên bản mới đã sửa theo feedback.

Draft mới lại được lưu vào memory.

Sau đó vòng lặp quay lại Critique.

---

Nói ngắn gọn:

> Tạo draft → đánh giá → nếu chưa tốt thì sửa → lặp lại.

---

# Ví dụ trace đầy đủ

Giả sử user yêu cầu:

> “Viết mô tả sản phẩm một đoạn cho chuột không dây dành cho dân văn phòng.”

---

## Phase 1: Generate

Generator tạo draft đầu tiên:

```text
This wireless mouse is good for office work. It connects to your computer
and lets you click and scroll. The battery lasts a long time. It is
comfortable to hold.
```

Draft được lưu vào memory.

---

## Phase 2: Critique

Critic đánh giá:

```text
- Draft quá chung chung.
- Không đề cập feature cụ thể mà dân văn phòng quan tâm.
- Không có số liệu pin cụ thể.
- Cụm "good for office work" quá yếu.
- Không có opening hook rõ ràng.
```

Critique được lưu vào memory.

---

## Phase 3: Revise

Generator đọc critique và viết lại:

```text
Stay focused through long meetings and quiet workdays with our silent-click
wireless mouse, designed for office workers who care about a clean,
distraction-free desk. The contoured grip keeps your hand relaxed for
hours, and the precision scroll wheel makes long documents easy to skim.
A single AA battery powers the mouse for up to 18 months, so you can
forget about charging cables for good.
```

Draft mới được lưu vào memory.

---

### Lưu ý quan trọng

Agent đã tự bịa ra:

> “18 months battery life”

… để đáp ứng yêu cầu “có số liệu cụ thể” của Critic.

Đây là rủi ro rất thật:

> Khi Critic yêu cầu chi tiết mà model không thực sự có dữ liệu, Generator có thể hallucinate.

Ta sẽ xử lý điều này ở phần failure modes.

---

## Stop Check

Critic đọc draft mới và không còn gì đáng sửa.

Stop Check kích hoạt → agent dừng.

Final Output là draft thứ hai.

---

Ở đây ta thấy:

- Agent không trả lời tốt ngay lần đầu.
- Nó tự phát hiện vấn đề.
- Tự sửa.
- Và cải thiện output hoàn toàn tự động.

---

# Reflection Agent vs ReAct Agent

Reflection Agent và ReAct Agent đều là AI Agent, nhưng mục tiêu của vòng lặp khác nhau.

---

## ReAct Agent

Vòng lặp dùng để:

- suy nghĩ,
- gọi tool,
- đọc observation,
- tìm bước tiếp theo.

Mục tiêu:

> khám phá thông tin và hành động.

---

## Reflection Agent

Vòng lặp dùng để:

- viết draft,
- tự review,
- viết draft tốt hơn.

Mục tiêu:

> cải thiện chất lượng output.

---

## So sánh trực quan

### ReAct Agent

```text
+---------+   +--------+   +-------------+
| Thought |-->| Action |-->| Observation |
+---------+   +--------+   +-------------+
```

---

### Reflection Agent

```text
+-----------+   +--------+   +--------+
| Generator |-->| Critic |-->| Revise |
+-----------+   +--------+   +--------+
```

---

## Khác biệt cốt lõi

| Thuộc tính        | ReAct Agent              | Reflection Agent         |
| ----------------- | ------------------------ | ------------------------ |
| Mục đích vòng lặp | Khám phá thế giới        | Cải thiện output         |
| Thay đổi mỗi lượt | Action + Observation     | Draft                    |
| Tool              | Trung tâm                | Tùy chọn                 |
| Điều kiện dừng    | Đủ thông tin để trả lời  | Draft đủ tốt             |
| Tốt nhất cho      | Task cần thông tin ngoài | Task cần chất lượng cao  |
| Cost              | Tăng theo tool calls     | Tăng theo số vòng revise |

---

Nói đơn giản:

> Dùng ReAct khi cần lấy thông tin từ thế giới bên ngoài.  
> Dùng Reflection khi cần output được polish tốt hơn.

---

## Khi KHÔNG nên dùng Reflection

Không nên dùng Reflection nếu:

- Draft đầu tiên thường đã đúng:
  - Q&A đơn giản,
  - classification,
  - lookup.
- Latency quan trọng hơn chất lượng:
  - real-time chat,
  - voice agent.

---

# Các lỗi phổ biến và cách khắc phục

## 1. Soft Critic

Critic luôn khen và không chỉ ra lỗi thật.

Kết quả:

- agent dừng quá sớm,
- output yếu.

**Khắc phục:**

- yêu cầu Critic luôn chỉ ra ít nhất 3 vấn đề cụ thể,
- chỉ được nói “không có vấn đề” nếu thật sự đạt tiêu chuẩn rõ ràng.

---

## 2. Useless Critic

Critique quá mơ hồ:

> “make it better”

Generator không biết sửa thế nào.

**Khắc phục:**

- critique phải actionable,
- ví dụ:
  - “câu thứ hai quá dài, hãy tách ra”
  - thay vì “improve flow”.

---

## 3. Worse Revisions

Revision mới tệ hơn bản cũ.

**Khắc phục:**

- giữ draft cũ trong memory,
- Stop Check phải so sánh bản mới với bản cũ,
- nếu tệ hơn → rollback.

---

## 4. Endless Loop

Critic luôn tìm được thứ để sửa.

Agent không bao giờ dừng.

**Khắc phục:**

- đặt max rounds,
- dừng nếu critique lặp lại.

---

## 5. Cost Explosion

Mỗi vòng Reflection cần:

- 1 Critic call
- 1 Generator call

Ví dụ:

- 5 rounds ≈ 11 LLM calls.

**Khắc phục:**

- giới hạn số vòng,
- dùng model nhỏ hơn cho Critic,
- bỏ Reflection cho task đơn giản.

---

## 6. Drift From the Task

Critic liên tục yêu cầu:

- dài hơn,
- fancy hơn,
- chi tiết hơn,

… nhưng lệch khỏi yêu cầu user.

**Khắc phục:**

- luôn đưa original task vào prompt,
- Critic phải kiểm tra:
  - “draft có trả lời đúng task không?”  
     trước khi đề xuất cải thiện khác.

---

## 7. Hallucination via Critique

Critic yêu cầu:

- benchmark,
- citation,
- số liệu cụ thể

… mà model không có.

Generator bịa ra để đáp ứng critique.

**Khắc phục:**

- cho Critic tool để verify fact,
- hoặc cấm Critic yêu cầu thông tin không có trong task gốc.

---

# Tóm tắt nhanh

- **Reflection Agent** là AI Agent dùng pattern Reflection:
  - generate,
  - critique,
  - revise,
  - lặp lại cho đến khi output đủ tốt.
- Reflection Agent = Generator + Critic + Agent Loop.
- Cấu trúc gồm:
  - Generator,
  - Critic,
  - Tools,
  - Memory,
  - Stop Check.
- Flow gồm:
  - Generate,
  - Critique,
  - Revise.
- Reflection vs ReAct:
  - ReAct dùng vòng lặp để khám phá thế giới.
  - Reflection dùng vòng lặp để cải thiện output.
- Các failure modes phổ biến:
  - Soft Critic,
  - Useless Critic,
  - Worse Revisions,
  - Endless Loop,
  - Cost Explosion,
  - Drift From the Task,
  - Hallucination via Critique.

Reflection là một trong những pattern phổ biến nhất để xây dựng AI Agent tạo ra output chất lượng cao và được polish kỹ.

---

_Nguồn: [Outcome School - Reflection Agent](https://outcomeschool.com/blog/reflection-agent?utm_source=chatgpt.com)_
