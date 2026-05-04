# ReAct Agent

_Tác giả: Amit Shekhar | Ngày đăng: 30 tháng 4, 2026_

---
![[ReAct Agent.png]]

Trong bài này, chúng ta sẽ tìm hiểu về ReAct Agent — nó là gì, được xây dựng như thế nào, cấu trúc bên trong, cách nó suy nghĩ và hành động, và cách xử lý các lỗi phổ biến.

Nghe đến **ReAct Agent** có vẻ phức tạp, nhưng đừng lo. Nếu ta tách nó ra từng phần nhỏ, mỗi phần đều rất đơn giản.

Nội dung bao gồm:

- ReAct Agent là gì
- ReAct Agent vs AI Agent
- Cấu trúc của một ReAct Agent
- ReAct Prompt Template
- Cách ReAct Agent suy nghĩ và hành động
- Ví dụ trace đầy đủ
- Triển khai một ReAct Agent
- Các lỗi phổ biến và cách khắc phục
- Tóm tắt nhanh

---

## ReAct Agent là gì

**ReAct Agent** là một **AI Agent** được xây dựng theo pattern **ReAct (Reasoning + Acting)** — pattern phổ biến nhất để xây dựng AI Agent. Nó không phải là một lần gọi LLM đơn lẻ. Đó là một vòng lặp quanh LLM, trong đó LLM được cung cấp danh sách các tool nó có thể gọi. Ở mỗi bước, LLM suy luận về việc cần làm và đề xuất một tool. Vòng lặp chạy tool đó, thêm kết quả vào bộ nhớ, và LLM suy luận tiếp ở lượt sau.

Phân tích thuật ngữ:

**ReAct Agent = Reasoning (Suy luận) + Acting (Hành động) + Agent**

- **Reasoning** là phần suy nghĩ — LLM tìm ra bước tiếp theo cần làm.
- **Acting** là phần thực hiện — LLM chọn một tool, và vòng lặp gọi nó.
- **Agent** là lớp bọc quanh LLM, chạy vòng lặp này, thực thi tool, và đưa kết quả trở lại.

Nói đơn giản:

> **ReAct Agent = LLM + Tools + Vòng lặp cho phép LLM suy nghĩ, hành động, và quan sát cho đến khi task hoàn thành.**

Hãy nghĩ ReAct Agent như một intern mới vào ngày đầu tiên. Họ không giải quyết mọi vấn đề trong đầu. Họ nghĩ về những gì cần, mở một tool, đọc kết quả, nghĩ lại, và lặp lại. ReAct Agent hoạt động y hệt — nhưng với tốc độ của máy móc.

Một lần gọi LLM đơn thuần cho ta một phản hồi rồi dừng. ReAct Agent cho ta một phản hồi, chạy tool, đọc kết quả, và tiếp tục — cho đến khi có câu trả lời cuối cùng.

---

## ReAct Agent vs AI Agent

Câu hỏi tự nhiên: nếu ReAct Agent là LLM trong vòng lặp với tool, thì đó chẳng phải là AI Agent sao?

**ReAct là một pattern để xây dựng AI Agent. Nó không phải là một danh mục khác.**

- **AI Agent** là danh mục — bất kỳ hệ thống nào có LLM được bọc trong vòng lặp với tool và bộ nhớ. Nó không quy định _cách_ vòng lặp được cấu trúc, _cách_ LLM chọn hành động tiếp theo, hay _liệu_ quá trình suy luận có hiển thị hay không.
    
- **ReAct** là một pattern để xây dựng vòng lặp đó, trong đó LLM theo chu kỳ **Thought (Suy nghĩ) → Action (Hành động) → Observation (Quan sát)**. Ở mỗi bước, LLM trước tiên viết suy luận của mình (Thought), sau đó chọn tool (Action), rồi đọc kết quả (Observation), rồi suy nghĩ lại.
    

Khi ta nói "ReAct Agent", ta có nghĩa là AI Agent được xây dựng theo pattern ReAct. Các pattern khác như Plan-and-Execute, Reflection, hay Agentic RAG cũng xây dựng AI Agent — chúng chỉ định hình vòng lặp theo cách khác.

> **ReAct là một cách để xây dựng AI Agent. Không phải tất cả AI Agent đều được xây dựng theo cách này, nhưng hầu hết là vậy.**

|Thuộc tính|AI Agent (danh mục)|ReAct Agent (theo pattern ReAct)|
|---|---|---|
|Định nghĩa|Bất kỳ LLM nào trong vòng lặp với tool|LLM lặp qua Thought, Action, Observation|
|Suy luận|Có thể hiển thị hoặc không|Được hiển thị rõ ràng ở bước Thought|
|Pattern vòng lặp|Không quy định|Thought → Action → Observation → lặp lại|
|Ví dụ|ReAct, Plan-and-Execute, Reflection, Agentic RAG|Chỉ pattern ReAct|
|Khi nào dùng thuật ngữ|Khi nói chung về agent|Khi muốn chỉ vòng lặp Thought-Action-Observation cụ thể|

---

## Cấu trúc của một ReAct Agent

Một ReAct Agent có năm phần:

![[Anatomy of ReAct Agent.png]]

**1. LLM.** Đây là bộ não. Nó thực hiện việc suy luận. Nó đọc lịch sử cuộc trò chuyện và quyết định bước tiếp theo — hoặc chọn tool để gọi, hoặc đưa ra câu trả lời cuối cùng.

**2. System Prompt.** Đây cho LLM biết cách hoạt động. Nó giải thích pattern ReAct, liệt kê các tool có sẵn, và đặt ra các quy tắc. Không có system prompt tốt, LLM có thể không suy nghĩ, hành động, hay dừng đúng cách.

**3. Tools.** Đây là các hành động agent có thể thực hiện — tìm kiếm web, truy vấn database, chạy máy tính, gửi email, đọc file, v.v. Mỗi tool có tên, mô tả, và input schema.

**4. Memory.** Đây là lịch sử chạy của cuộc trò chuyện — câu hỏi của người dùng, mọi thought, mọi action, mọi observation. LLM đọc bộ nhớ này ở mỗi bước để quyết định làm gì tiếp theo.

**5. Loop Controller.** Đây là code chạy vòng lặp. Nó gửi bộ nhớ đến LLM, thực thi các tool call, thêm kết quả trở lại bộ nhớ, và kiểm tra xem agent đã xong chưa. Nó cũng xử lý điều kiện dừng — như số bước tối đa.

Cả năm phần hoạt động cùng nhau. Xóa bất kỳ phần nào, nó không còn là ReAct Agent nữa.

---

## ReAct Prompt Template

System prompt tốt là thứ khiến LLM thuần túy hoạt động đáng tin cậy như một ReAct Agent. Không có nó, LLM có thể trả lời quá sớm, bỏ qua suy luận, hoặc dùng tool kém.

Đây là một template ReAct prompt đơn giản:

```
Bạn là một trợ lý hữu ích giải quyết vấn đề bằng cách suy nghĩ từng bước và đề xuất tool khi cần.

Bạn có quyền truy cập vào các tool sau:
- search(query): Tìm kiếm thông tin trên web.
- calculator(expression): Tính toán một biểu thức toán học.

Ở mỗi bước, hãy trả lời theo một trong hai format sau:

Format 1 - Khi bạn cần đề xuất tool:
Thought: <suy luận của bạn về việc cần làm tiếp theo>
Action: <tên_tool>(<input_tool>)

Format 2 - Khi bạn có câu trả lời cuối cùng:
Thought: <suy luận cuối cùng của bạn>
Final Answer: <câu trả lời cho câu hỏi của người dùng>

Quy tắc:
- Luôn bắt đầu bằng Thought.
- Đề xuất một tool mỗi lượt, trừ khi nhiều tool độc lập rõ ràng có thể giúp song song.
- Chờ Observation trước khi tiếp tục.
- Dừng ngay khi bạn có câu trả lời cuối cùng.
```

Prompt này làm ba việc:

- Giải thích pattern ReAct (Thought, Action, Observation, Final Answer).
- Liệt kê các tool có sẵn với input của chúng.
- Đặt ra quy tắc để giữ agent không đi chệch hướng.

> **Lưu ý:** Nếu dùng Claude, GPT, hay Gemini, không nên copy format string `Action: tool(input)` ở trên — hãy định nghĩa tool bằng tool/function-calling schema của API. API xử lý format action cho ta, validate input, và parse tool call. Ý tưởng cốt lõi vẫn như nhau — prompt cho LLM biết khi nào suy nghĩ, khi nào hành động, khi nào dừng — nhưng cách kết nối sạch hơn.

---

## Cách ReAct Agent suy nghĩ và hành động

Đây là hình dạng của vòng lặp:

```
        +-----------------+
        |  Câu hỏi User   |
        +-----------------+
                 |
                 v
        +-----------------+
        |     Thought     |  <-- LLM suy luận về bước tiếp theo
        +-----------------+
                 |
        +--------+--------+
        |                 |
        v                 v
  +-----------+    +---------------+
  |  Action   |    | Final Answer  | ---> trả về người dùng
  +-----------+    +---------------+
        |
        v
  +-----------+
  |   Tool    |
  +-----------+
        |
        v
  +-------------+
  | Observation | --+
  +-------------+   |
                    |
                    +---> quay lại Thought (vòng lặp tiếp tục)
```

Các bước agent thực hiện:

**Bước 1:** Người dùng gửi câu hỏi. Loop Controller thêm vào Memory.

**Bước 2:** Loop Controller gửi Memory (system prompt + câu hỏi + lịch sử) đến LLM.

**Bước 3:** LLM trả về Thought + Action, hoặc Final Answer.

**Bước 4:** Nếu là Final Answer, Loop Controller trả về cho người dùng. Agent xong.

**Bước 5:** Nếu là Thought + Action, Loop Controller thực thi tool và lấy kết quả — đây là Observation.

**Bước 6:** Loop Controller thêm Thought, Action, và Observation vào Memory.

**Bước 7:** Quay lại Bước 2.

Vòng lặp tiếp tục cho đến khi LLM đưa ra Final Answer hoặc đạt giới hạn bước tối đa.

---

## Ví dụ trace đầy đủ

Hãy trace qua một ví dụ thực tế:

**User:** "15% dân số Tokyo là bao nhiêu người?"

```
Thought 1: Tôi cần tìm dân số Tokyo trước. Tôi không biết con số chính xác hiện tại, vì vậy tôi sẽ tìm kiếm.
Action 1: search("dân số hiện tại của Tokyo")
Observation 1: Dân số Tokyo xấp xỉ 14 triệu người.

Thought 2: Bây giờ tôi cần tính 15% của 14 triệu. Hãy dùng máy tính để chính xác.
Action 2: calculator("14000000 * 0.15")
Observation 2: 2100000

Thought 3: Tôi đã có câu trả lời. 15% dân số Tokyo (14 triệu) là 2.1 triệu.
Final Answer: 15% dân số Tokyo là khoảng 2.1 triệu người.
```

Ta thấy agent đã:

- Nhận ra nó không có dữ liệu dân số và tìm đến tool.
- Kết nối output của search vào input của calculator.
- Dừng ngay khi có câu trả lời cuối cùng.

Đó là vẻ đẹp của ReAct Agent. Mỗi bước nhỏ và đơn giản, nhưng vòng lặp cho phép nó giải quyết các vấn đề mà một lần gọi LLM đơn lẻ không thể.

---

## Triển khai một ReAct Agent

Đây là nhận thức quan trọng:

**Nếu ta viết code lấy câu hỏi người dùng, gửi đến LLM, chạy các tool LLM chọn, đưa observation trở lại, và lặp lại vòng lặp này cho đến khi LLM có câu trả lời cuối cùng — đoạn code đó là một ReAct Agent.**

Đây là toàn bộ skeleton của một ReAct Agent trong khoảng 20 dòng Python:

```python
REACT_SYSTEM_PROMPT = "..."  # ReAct prompt template từ phần trước

async def run_react_agent(user_question, tools, max_steps=10):
    messages = [
        {"role": "system", "content": REACT_SYSTEM_PROMPT},
        {"role": "user", "content": user_question},
    ]
    step = 0

    while True:
        # Dừng an toàn: thoát nếu đạt giới hạn bước
        if step >= max_steps:
            return "Đã đạt giới hạn bước mà không có câu trả lời cuối cùng."
        step += 1

        # Yêu cầu LLM suy nghĩ, sau đó hành động hoặc đưa ra câu trả lời
        response = await call_llm(messages, tools)

        # Nếu LLM có câu trả lời cuối cùng, dừng vòng lặp và trả về
        if response.is_done:
            return response.final_answer

        # Nếu không, chạy từng tool LLM chọn và đưa observation trở lại
        for tool_call in response.tool_calls:
            observation = await call_tool(tool_call.name, tool_call.arguments)
            messages.append({
                "role": "tool",
                "name": tool_call.name,
                "content": str(observation),
            })
```

Giải thích các phần quan trọng:

- **Messages ban đầu:** Seed bộ nhớ với ReAct system prompt và câu hỏi người dùng.
- **Vòng lặp `while True`:** Đây là Loop Controller. Nó tiếp tục cho đến khi LLM có câu trả lời hoặc đạt giới hạn bước.
- **Giới hạn bước:** Thoát sau `max_steps` vòng lặp — bảo vệ khỏi infinite loop.
- **Lần gọi LLM:** Gửi toàn bộ bộ nhớ cùng danh sách tool. LLM thực hiện bước Thought ở đây.
- **Kiểm tra dừng:** Nếu `response.is_done` là `True`, LLM đang nói **"Tôi xong rồi, đây là câu trả lời."**
- **Nhánh tool call:** Nếu LLM chọn tool, ta chạy nó, thêm observation vào bộ nhớ, và vòng lặp tiếp tục.

---

## Các lỗi phổ biến và cách khắc phục

**1. Infinite loop.** Agent liên tục gọi cùng một tool với cùng input và không bao giờ đưa ra câu trả lời.

_Khắc phục:_ Đặt giới hạn `max_steps` cứng. Phát hiện các action lặp lại giống hệt nhau và dừng lại hoặc inject tin nhắn yêu cầu agent thử cách tiếp cận khác.

**2. Chọn sai tool.** Agent gọi tool sai — như dùng search cho bài toán toán học.

_Khắc phục:_ Viết mô tả tool thật rõ ràng. Mô tả là thứ LLM dùng để chọn tool, nên mỗi từ đều quan trọng.

**3. Hallucinated tool call.** Agent gọi tool không tồn tại hoặc truyền argument không hợp lệ.

_Khắc phục:_ Dùng LLM có hỗ trợ native tool-use (như Claude, GPT, hoặc Gemini) để schema được enforce. Luôn validate tool input trước khi thực thi.

**4. Context explosion.** Sau nhiều bước, lịch sử cuộc trò chuyện quá dài cho context window của LLM.

_Khắc phục:_ Tóm tắt các bước cũ, xóa observation lỗi thời, hoặc dùng bộ nhớ riêng. Với agent chạy lâu, ta phải chủ động quản lý context.

**5. Dừng quá sớm.** Agent đưa ra câu trả lời trước khi thu thập đủ thông tin.

_Khắc phục:_ Thêm bước critic review câu trả lời cuối trước khi trả về, hoặc yêu cầu agent kiểm tra câu trả lời theo checklist trước khi emit Final Answer.

**6. Bị kẹt sau lỗi tool.** Tool thất bại và agent không biết cách phục hồi.

_Khắc phục:_ Bắt lỗi tool, chuyển thành Observation ngôn ngữ tự nhiên (ví dụ: "Lỗi: search API timeout. Thử lại hoặc dùng query khác."), và để LLM tự tìm cách thoát.

---

## Tóm tắt nhanh

- **ReAct Agent** là AI Agent lặp qua Thought, Action, và Observation cho đến khi task hoàn thành. Đó là LLM được bọc trong vòng lặp với tool và bộ nhớ.
- **Cấu trúc** có năm phần: LLM, System Prompt, Tools, Memory, và Loop Controller. Xóa bất kỳ phần nào, nó không còn là ReAct Agent.
- **System prompt** là thứ khiến LLM thuần túy hoạt động đáng tin cậy như ReAct Agent. Nó giải thích pattern, liệt kê tool, và đặt ra quy tắc.
- **Vòng lặp** đơn giản: gửi memory đến LLM → nếu tool call, thực thi và thêm observation → nếu final answer, trả về.
- **Các lỗi phổ biến** bao gồm infinite loop, chọn sai tool, hallucinated tool call, context explosion, dừng quá sớm, và lỗi sau khi tool thất bại.
- ReAct hoặc vòng lặp kiểu ReAct là pattern phổ biến nhất bên dưới các AI agent hiện đại — từ coding assistant đến customer support bot đến research assistant.

---

_Nguồn: [ReAct Agent - Outcome School](https://outcomeschool.com/blog/react-agent) by Amit Shekhar_