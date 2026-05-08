# Context Engineering

## Context Engineering là gì?
![[Context Engineering.png]]
**Context Engineering** là thực hành thiết kế, tổ chức và quản lý mọi thứ đi vào _context window_ của LLM để mô hình có thể thực hiện task một cách ổn định và đáng tin cậy.

Nói đơn giản:

> **Context Engineering = đưa đúng thông tin, đúng cấu trúc, đúng thời điểm vào LLM.**

LLM bản chất chỉ là một hàm dự đoán token tiếp theo dựa trên context hiện tại.  
Nếu context tốt → output tốt.  
Nếu context tệ → output tệ.

---

# Bức tranh lớn (The Big Picture)

Hãy tưởng tượng bạn đang briefing cho một đồng nghiệp cực kỳ thông minh nhưng:

- không nhớ bất kỳ cuộc trò chuyện nào trước đó
- mỗi task đều bắt đầu từ con số 0

Để họ làm việc tốt, bạn phải cung cấp:

- Vai trò của họ là gì
- Task cần làm
- Tài liệu nền
- Ví dụ mẫu
- Tool được phép dùng
- Format output mong muốn

LLM hoạt động y hệt như vậy.

- **LLM** = đồng nghiệp thông minh
- **Context window** = bản briefing
- **Context Engineering** = nghệ thuật viết briefing đó

---

# Vì sao Context Engineering quan trọng?

LLM là **stateless** — không nhớ gì giữa các lần gọi.

Điều này dẫn đến:

1. Output phụ thuộc hoàn toàn vào input context.
2. Model trung bình + context tốt có thể mạnh hơn model mạnh + context tệ.
3. Khi chuyển sang:
   - RAG
   - AI Agents
   - multi-step workflows

   → phần khó nhất không còn là model nữa, mà là context.

Ngày trước:

- trọng tâm là Prompt Engineering

Ngày nay:

- trọng tâm là Context Engineering

---

# Prompt Engineering vs Context Engineering

## Prompt Engineering

Tập trung vào:

- wording
- phrasing
- viết 1 prompt tốt

Ví dụ:

- “Summarize this article”  
   vs
- “Summarize this article in 3 bullet points for non-technical readers”

---

## Context Engineering

Thiết kế **toàn bộ môi trường thông tin** mà model nhìn thấy:

- system prompt
- examples
- RAG docs
- tools
- memory
- chat history
- tool outputs
- output format

---

## Khác biệt cốt lõi

| Aspect        | Prompt Engineering | Context Engineering               |
| ------------- | ------------------ | --------------------------------- |
| Scope         | Một prompt         | Toàn bộ context window            |
| Tập trung vào | Wording            | Toàn bộ thông tin model nhìn thấy |
| Phù hợp       | One-shot tasks     | Agents, RAG, workflows            |
| Failure mode  | Prompt wording tệ  | Thiếu/sai context                 |
| Ví dụ fix     | Rewrite prompt     | Inject thêm docs/memory           |

---

# Hình ảnh tổng quan

```text
+--------------------------------------------------+
| CONTEXT WINDOW                                   |
|--------------------------------------------------|
| System Prompt                                    |
| Few-shot Examples                                |
| Retrieved Docs (RAG)                             |
| Tools + Tool Descriptions                        |
| Memory                                            |
| Conversation History                             |
| Tool Results                                     |
| Current User Message                             |
|                                                  |
|   +------------------------------------------+   |
|   | PROMPT                                  |   |
|   | "Summarize in 3 bullet points..."       |   |
|   +------------------------------------------+   |
+--------------------------------------------------+
```

> Prompt Engineering chỉ là “hộp nhỏ bên trong”.  
> Context Engineering là toàn bộ “hộp lớn”.

---

# Các thành phần của Context

## 1. System Prompt

Định nghĩa:

- role
- rules
- persona
- format

Ví dụ:

- “You are a helpful coding assistant…”

---

## 2. Few-shot Examples

Các cặp:

- input → output

giúp model học pattern mong muốn.

---

## 3. Retrieved Documents (RAG)

Inject tài liệu liên quan vào context thay vì nhét toàn bộ knowledge base.

---

## 4. Tools & Tool Descriptions

Cho model biết:

- có tool nào
- dùng thế nào
- schema ra sao

---

## 5. Memory

Thông tin dài hạn:

- user preferences
- facts
- previous sessions

---

## 6. Conversation History

Lịch sử chat hiện tại.

---

## 7. Tool Results

Kết quả trả về sau khi tool chạy.

---

## 8. Current User Message

Message mới nhất của user.

---

# Hình ảnh đầy đủ của Context Window

```text
+------------------------------------------------+
|                CONTEXT WINDOW                  |
+------------------------------------------------+
| 1. System Prompt                               |
| 2. Few-shot Examples                           |
| 3. Retrieved Docs (RAG)                        |
| 4. Tools & Tool Descriptions                   |
| 5. Memory                                      |
| 6. Conversation History                        |
| 7. Tool Results                                |
| 8. Current User Message                        |
+------------------------------------------------+
                    |
                    v
               LLM Output
```

---

# Context được build như thế nào?

Context KHÔNG được lưu nguyên vẹn.

Mỗi turn:

- rebuild lại từ đầu

```text
+---------+  +--------+  +------+  +----------+
| System  |  | Memory |  | RAG  |  | Examples |
+---------+  +--------+  +------+  +----------+

+---------+  +--------+  +------+  +----------+
| Tools   |  | Tool   |  | Chat |  | Current  |
|         |  | Result |  | Hist |  | Message  |
+---------+  +--------+  +------+  +----------+

               ↓ combine

       +----------------------+
       |  Final Context       |
       +----------------------+
                 ↓
               LLM
```

---

# Các pattern phổ biến trong Context Engineering

## 1. RAG

Retrieve:

- đúng chunk
- đúng tài liệu

thay vì nhét toàn bộ dữ liệu.

---

## 2. Few-shot Examples

Cho model vài ví dụ mẫu để học pattern.

---

## 3. Tool Calling

Model:

- chọn tool
- runtime chạy tool
- output quay lại context

Đây là nền tảng của AI Agents.

---

## 4. Memory Injection

Chỉ inject phần memory liên quan tới task hiện tại.

Không đưa toàn bộ memory vào mọi lần gọi.

---

## 5. Conversation History Management

Kỹ thuật:

- summarization
- sliding window
- selective inclusion

để giữ context nhỏ.

---

## 6. Output Formatting

Ép model output theo:

- JSON
- schema
- markdown table
- structured form

---

## 7. Context Compression

Tóm tắt context để giảm token:

- summarization
- key fact extraction
- structured rewriting

---

## 8. Per-step Context Rebuilding (Agents)

Mỗi step của agent:

- rebuild context mới
- chỉ giữ thông tin liên quan hiện tại

Agent tốt:

- context gọn
- tập trung

Agent tệ:

- append mọi thứ mãi mãi
- context rot

---

# Các lỗi phổ biến

## 1. Stuffing Everything

Nhét quá nhiều thông tin.

Hậu quả:

- model bị nhiễu
- chậm
- đắt
- output tệ hơn

Hiện tượng này gọi là:

> **Context Rot / Context Degradation**

---

## 2. Không đủ context

Model không thể đoán thứ nó chưa thấy.

Nếu muốn model biết policy refund:

- policy phải nằm trong context.

---

## 3. Bad Ordering

Thông tin quan trọng nằm giữa context dài.

Hiện tượng:

> **Lost in the Middle**

Model chú ý:

- đầu context
- cuối context

hơn phần giữa.

---

## 4. Stale Context

Context chứa:

- docs cũ
- tool results cũ
- chat history không liên quan

→ gây nhiễu.

---

## 5. No Structure

Wall of text không có:

- headers
- separators
- labels

→ model khó parse.

---

## 6. Conflicting Instructions

System prompt nói A  
User prompt nói B

→ model không biết theo cái nào.

---

# Best Practices

## 1. Treat Context as a Budget

Mỗi token:

- tốn tiền
- tốn attention

Phân bổ budget rõ ràng.

---

## 2. Focus on Current Task

Chỉ đưa:

> “những gì model cần cho task hiện tại”

---

## 3. Order từ Stable → Fresh

Thứ tự tốt:

1. System Prompt
2. Examples
3. Retrieved Docs
4. Memory
5. History
6. Current Message

---

## 4. Use Clear Structure

Ví dụ:

```text
### SYSTEM
...

### EXAMPLES
...

### DOCS
...

### HISTORY
...

### CURRENT TASK
...
```

---

## 5. Debug the Context

Khi AI fail:

- đừng hỏi model sai chưa

- hãy hỏi context có đúng chưa

---

## 6. Version & Evaluate Context

Treat context như software:

- version
- diff
- A/B test
- eval

---

## 7. Compress Carefully

Compression giúp giảm token nhưng:

- có thể mất thông tin quan trọng

---

## 8. Build for Worst Case

Production luôn có:

- chat dài
- docs lớn
- nhiều tool calls

Context system phải chịu được worst-case.

# Quick Summary

## Let's recap what we have learned

- **Context Engineering** là thực hành thiết kế, tổ chức và quản lý mọi thứ đi vào _context window_ của LLM để model có thể thực hiện task một cách ổn định và đáng tin cậy.
- **Ý tưởng cốt lõi:**  
   LLM giống như một đồng nghiệp cực kỳ thông minh nhưng không có trí nhớ.  
   Context Engineering chính là bản briefing mà ta đưa cho họ mỗi lần làm task.
- **Vì sao nó quan trọng:**  
   LLM là stateless. Output phụ thuộc hoàn toàn vào context. Khi ứng dụng trở nên phức tạp hơn (RAG, agents, multi-step workflows), phần quan trọng nhất không còn là model nữa mà là context.
- **Prompt Engineering vs Context Engineering:**  
   Prompt Engineering là viết tốt một instruction.  
   Context Engineering là thiết kế toàn bộ môi trường thông tin mà model nhìn thấy.  
   Prompt Engineering chỉ là một phần nhỏ bên trong Context Engineering.
- **Các thành phần của context:**
  - System prompt
  - Few-shot examples
  - Retrieved documents (RAG)
  - Tools & tool descriptions
  - Memory
  - Conversation history
  - Tool results
  - Current user message
- **Các pattern phổ biến:**
  - RAG
  - Few-shot examples
  - Tool calling
  - Memory injection
  - Conversation history management
  - Output formatting
  - Context compression
  - Per-step context rebuilding cho agents
- **Các lỗi phổ biến:**
  - Nhét quá nhiều thông tin
  - Không đủ context
  - Sắp xếp context tệ
  - Context cũ/stale
  - Không có cấu trúc
  - Instructions mâu thuẫn nhau
- **Best practices:**
  - Xem context như một budget
  - Chỉ giữ thông tin liên quan task hiện tại
  - Sắp xếp từ stable → fresh
  - Dùng headers & section markers rõ ràng
  - Log & test context
  - Version và iterate context như software
  - Compress cẩn thận
  - Thiết kế cho worst-case

---

# In Simple Words

> **Context Engineering = Briefing the LLM with the right information, in the right shape, at the right time.**

---

# Insight quan trọng nhất

> **Model là engine. Context là fuel.**

Một model trung bình với context cực tốt thường mạnh hơn một model cực mạnh nhưng context tệ.

---

# Final Takeaway

Lần tới khi debug một AI application hoạt động không đúng:

❌ Đừng hỏi:

> “Model có ngu không?”

✅ Hãy hỏi:

> “Context đã đúng chưa?”
