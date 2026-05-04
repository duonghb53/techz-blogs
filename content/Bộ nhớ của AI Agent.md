---
title: AI Agents Memory
---
# Bộ nhớ của AI Agent

_Tác giả: Amit Shekhar | Ngày đăng: 28 tháng 4, 2026_

![[AI Agent Memory ChatGPT.png]]

---

Trong bài này, chúng ta sẽ tìm hiểu về Bộ nhớ của AI Agent — tại sao agent cần bộ nhớ, memory stack, bốn thao tác cốt lõi (ghi, đọc, cập nhật, xóa), cách bộ nhớ hoạt động tại runtime, và các lỗi phổ biến.

Nội dung bao gồm:

- Bức tranh tổng thể
- Tại sao AI Agent cần bộ nhớ
- Memory Stack
- Bốn thao tác cốt lõi
- Cách bộ nhớ hoạt động tại Runtime
- Nên lưu gì và không nên lưu gì
- Các lỗi phổ biến và cách khắc phục
- Tóm tắt nhanh

---

## Bức tranh tổng thể

Trước khi đi vào chi tiết, hãy hiểu bức tranh tổng thể.

Một LLM tự thân là **stateless** (không có trạng thái). Mỗi lần gọi nó, giống như nói chuyện với người chưa bao giờ gặp chúng ta. Nó không nhớ tên, cuộc trò chuyện trước, hay task nó đã giúp hôm qua. **Bộ nhớ AI Agent** là mọi thứ chúng ta xây dựng xung quanh LLM để tạo cảm giác ghi nhớ. LLM thực sự không nhớ — agent nhớ thay cho nó.

Nói đơn giản:

> **Bộ nhớ AI Agent = Hệ thống cho phép một LLM stateless hoạt động như thể nó nhớ qua các lượt, phiên, và người dùng.**

Hãy nghĩ AI agent như một trợ lý thông minh nhưng quên hết mọi thứ ngay khi cuộc họp kết thúc. Không có bộ nhớ, họ không thể học sở thích của bạn, nhớ các quyết định cũ, hay tiếp nối công việc hôm qua. Bộ nhớ là cuốn sổ tay, tủ hồ sơ, và nhật ký cá nhân giúp họ hữu ích từ ngày này sang ngày khác.

---

## Tại sao AI Agent cần bộ nhớ

Không có bộ nhớ, agent gặp bốn vấn đề lớn:

**Không có sự liên tục.** Agent không thể nhớ những gì người dùng nói 30 giây trước khi cuộc gọi hiện tại kết thúc — cuộc gọi tiếp theo bắt đầu từ đầu, không có gì được mang theo trừ khi chúng ta truyền vào một cách tường minh.

**Không có cá nhân hóa.** Agent không thể nhớ rằng người dùng thích câu trả lời ngắn, viết Python, hay sống ở Hà Nội. Nó đối xử với mọi người dùng như nhau.

**Không có học hỏi.** Agent không thể nhớ cái gì hiệu quả và cái gì không. Nó sẽ mắc cùng lỗi lặp đi lặp lại.

**Không có task dài.** Các task nhiều bước kéo dài hàng giờ hay hàng ngày trở nên bất khả thi. Agent quên kế hoạch giữa chừng và phải bắt đầu lại.

Đó là lúc **Bộ nhớ AI Agent** ra đời để giải cứu. Nó giải quyết cả bốn vấn đề bằng cách cho agent một nơi để lưu thông tin và một cách để lấy lại khi cần.

---

## Memory Stack

Bộ nhớ AI Agent không phải là một thứ đơn lẻ. Đó là một **stack gồm nhiều tầng**, mỗi tầng có vòng đời và mục đích khác nhau.

```
+--------------------------------------------+
|           Bộ nhớ AI Agent                  |
|                                            |
|  Tầng 1: Context Window (trong prompt)     |
|     - Trong prompt hiện tại                |
|     - Mất khi context tràn                 |
|                                            |
|  Tầng 2: Short-Term Memory (phiên làm)     |
|     - Scratchpad, trạng thái task hiện tại |
|     - Mất khi phiên kết thúc               |
|                                            |
|  Tầng 3: Long-Term Memory (lâu dài)        |
|     - Thông tin về người dùng              |
|     - Các cuộc trò chuyện cũ              |
|     - Sở thích đã học                      |
|                                            |
|  Tầng 4: External Knowledge (tools/RAG)    |
|     - Documents, databases, APIs           |
|     - Truy vấn, không do agent quản lý     |
+--------------------------------------------+
```

**Tầng 1 — Context Window.** Đây là văn bản LLM thấy trong một lần gọi. Đây là bộ nhớ nhanh nhất vì LLM đọc trực tiếp. Nhưng cũng nhỏ nhất. Khi đầy, các tin nhắn cũ phải bị xóa hoặc tóm tắt.

**Tầng 2 — Short-Term Memory.** Đây là scratchpad agent dùng trong một task hoặc phiên. Nó lưu kế hoạch, bước hiện tại, và kết quả trung gian. Khi phiên kết thúc, bộ nhớ này thường biến mất.

**Tầng 3 — Long-Term Memory.** Đây là bộ nhớ lâu dài tồn tại qua các cuộc trò chuyện và phiên. Nó lưu thông tin người dùng, sở thích, tóm tắt cuộc trò chuyện cũ, và hành vi đã học. Đây là thứ khiến agent cảm giác như "biết" người dùng.

**Tầng 4 — External Knowledge.** Đây là mọi thứ agent có thể truy vấn nhưng không tự quản lý — documents, databases, APIs, web. Agent đọc từ các nguồn này nhưng không ghi lại như bộ nhớ của chính mình.

Bốn tầng hoạt động cùng nhau: Context window là bàn làm việc. Short-term memory là tờ ghi chú cạnh bàn. Long-term memory là nhật ký trong ngăn kéo. External knowledge là thư viện ở phố bên cạnh.

> **Lưu ý:** Trong Long-Term Memory còn có các sub-type: **episodic** (kinh nghiệm quá khứ), **semantic** (sự kiện và khái niệm), và **procedural** (cách làm gì đó). Sẽ được đề cập trong các bài tiếp theo.

---

## Bốn thao tác cốt lõi

Dù có bao nhiêu tầng, mọi hệ thống bộ nhớ đều chỉ làm bốn việc:

**1. Ghi (Write).** Lưu thông tin mới. Khi có điều quan trọng xảy ra — sở thích người dùng, kết quả task, sự kiện quan trọng — ta ghi vào bộ nhớ. _Ví dụ: sau khi người dùng nói "tôi thích code bằng Go", ghi "ngôn ngữ ưa thích: Go" vào long-term memory._

**2. Đọc (Read).** Lấy thông tin liên quan khi cần. Đây thường là phần khó nhất. Ta không muốn mọi thứ, chỉ cần thứ quan trọng lúc này. _Ví dụ: nếu người dùng hỏi "viết hàm đảo ngược chuỗi", ta lấy "ngôn ngữ ưa thích: Go" trước khi gọi LLM._

**3. Cập nhật (Update).** Sửa đổi bộ nhớ khi có thông tin mới mâu thuẫn. Nếu ngôn ngữ yêu thích của người dùng từng là Python và giờ là Go, bộ nhớ phải được cập nhật.

**4. Xóa (Forget).** Xóa bộ nhớ cũ, sai, hoặc không còn liên quan. Không có việc xóa, bộ nhớ tăng mãi và truy xuất trở nên tệ hơn. _Ví dụ: ghi chú "người dùng đang debug login" phải bị xóa khi phiên kết thúc._

> **Hệ thống bộ nhớ = Ghi + Đọc + Cập nhật + Xóa.**

Nếu bất kỳ thao tác nào bị hỏng, cả hệ thống hỏng theo.

---

## Cách bộ nhớ hoạt động tại Runtime

Đây là luồng hoạt động khi agent chạy:

```
                   Tin nhắn người dùng
                          |
                          v
          +----------------------------+
          |         ĐỌC bộ nhớ        |
          |  - Long-Term (sự kiện,     |
          |    sở thích, tóm tắt)      |
          |  - Short-Term (kế hoạch,   |
          |    các bước gần đây)       |
          +-------------+--------------+
                        |
                        v
          +----------------------------+
          |       Xây dựng Prompt      |
          |  (system prompt +          |
          |   bộ nhớ đã lấy +          |
          |   tin nhắn người dùng)     |
          +-------------+--------------+
                        |
                        v
          +----------------------------+    +---------------------+
          |            LLM             |<-->| External Knowledge  |
          |    (suy luận, trả lời)     |    | (tools, RAG, APIs)  |
          +-------------+--------------+    +---------------------+
                        |
                        v
          +----------------------------+
          |         GHI bộ nhớ        |
          |  - Short-Term (bước mới)   |
          |  - Long-Term (nếu đáng     |
          |    lưu lại)                |
          +-------------+--------------+
                        |
                        v
                      Phản hồi
```

Mỗi lượt người dùng, luồng là: **Đọc → Xây dựng prompt → Phản hồi → Ghi.**

```python
while user_active:
    user_message = receive_message()

    # ĐỌC
    long_term_context = search_long_term_memory(user_id, user_message)
    short_term_context = read_short_term_memory(session_id)

    # XÂY DỰNG PROMPT
    messages = [system_prompt_with(long_term_context, short_term_context), user_message]

    # VÒNG LẶP LLM
    while True:
        response = call_llm(messages, tools)
        if response.is_tool_call:
            tool_result = call_tool(response.tool_name, response.tool_args)
            messages.append(response)
            messages.append(tool_result)
        else:
            break

    # GHI
    write_short_term_memory(session_id, user_message, response)
    if is_worth_keeping(user_message, response):
        write_long_term_memory(user_id, user_message, response)

    send_to_user(response)
```

**Ví dụ minh họa:**

**Lượt 1:** "Tên tôi là Priya và tôi code bằng Go."

- Long-term memory trống. Không có gì để lấy.
- LLM trả lời: "Rất vui được gặp bạn, Priya."
- Agent ghi vào long-term memory: "Tên người dùng là Priya. Ngôn ngữ ưa thích: Go."

**Lượt 2 (một tuần sau):** "Giúp tôi viết hàm đảo ngược chuỗi."

- Agent tìm kiếm long-term memory, lấy ra: "Priya, thích Go."
- Prompt = System prompt + "Người dùng là Priya, thích Go" + "Giúp tôi viết hàm đảo ngược chuỗi."
- LLM trả lời bằng hàm Go, gọi tên Priya.

Đó là vẻ đẹp của Bộ nhớ AI Agent. Lượt thứ hai cảm giác cá nhân hóa dù xảy ra một tuần sau.

---

## Nên lưu gì và không nên lưu gì

**Nên lưu:**

- Thông tin người dùng ổn định (tên, vai trò, ngôn ngữ, múi giờ)
- Sở thích mạnh (thích câu trả lời ngắn, muốn code bằng Python)
- Kết quả của các task cũ (cái gì hiệu quả, cái gì thất bại)
- Quyết định ảnh hưởng đến công việc tương lai (lựa chọn kiến trúc, chính sách, ràng buộc)

**Không nên lưu:**

- Mọi tin nhắn (context window đã có cuộc trò chuyện trực tiếp)
- Chit-chat ít giá trị ("hi", "cảm ơn", "ok")
- Trạng thái tạm thời đã có trong short-term memory
- Dữ liệu nhạy cảm không có sự đồng ý lưu trữ

> **Lưu những gì sẽ quan trọng tuần sau. Bỏ qua những gì chỉ quan trọng trong phút tới.**

---

## Các lỗi phổ biến và cách khắc phục

**1. Context window tràn.** Prompt quá dài, bị cắt hoặc lỗi. _Khắc phục:_ Tóm tắt các lượt cũ. Chỉ đưa bộ nhớ liên quan nhất vào prompt.

**2. Retrieval bỏ sót bộ nhớ quan trọng.** Agent có bộ nhớ nhưng không tìm thấy khi cần. _Khắc phục:_ Dùng semantic search (embeddings) thay vì keyword match. Lưu bộ nhớ với tiêu đề và mô tả tốt.

**3. Bộ nhớ lỗi thời.** Agent tiếp tục hành động dựa trên thông tin cũ. _Khắc phục:_ Mỗi lần ghi, kiểm tra xem có bộ nhớ nào mâu thuẫn không. Nếu có, cập nhật hoặc thay thế, không chỉ append.

**4. Memory bloat.** Bộ nhớ tăng mãi, retrieval chậm và nhiễu. _Khắc phục:_ Thêm chính sách xóa. Xóa bộ nhớ không được dùng lâu, hoặc đã bị thay thế bởi bộ nhớ mới hơn.

**5. Rò rỉ thông tin cá nhân.** Agent nhớ thông tin nhạy cảm và tiết lộ sai ngữ cảnh. _Khắc phục:_ Gán mỗi bộ nhớ với user ID. Không bao giờ trộn bộ nhớ giữa các người dùng. Mã hóa dữ liệu nhạy cảm và yêu cầu đồng ý trước khi ghi.

**6. Nhầm lẫn short-term và long-term.** Agent ghi trạng thái tạm thời vào long-term memory, hoặc ngược lại. _Khắc phục:_ Cố ý chọn tầng cho mỗi lần ghi. Nếu chỉ hữu ích cho task này, giữ ở short-term. Nếu sẽ quan trọng sau, đẩy lên long-term.

> **Quan trọng:** Hầu hết các lỗi này im lặng. Agent không crash — nó chỉ đưa ra câu trả lời sai một cách tinh tế. Đó là lý do phải thêm logging, monitoring, và kiểm tra định kỳ bộ nhớ.

---

## Tóm tắt nhanh

- **Bộ nhớ AI Agent** là hệ thống cho phép LLM stateless hoạt động như thể nó nhớ qua các lượt, phiên, và người dùng.
- **Memory stack** có bốn tầng: context window, short-term memory, long-term memory, và external knowledge.
- **Bốn thao tác cốt lõi** là Ghi, Đọc, Cập nhật, và Xóa. Hỏng bất kỳ thao tác nào, cả hệ thống hỏng.
- **Luồng runtime** mỗi lượt: lấy bộ nhớ liên quan → xây dựng prompt → gọi LLM → ghi bộ nhớ mới → phản hồi.
- **Lưu những gì sẽ quan trọng tuần sau.** Hầu hết hệ thống bộ nhớ thất bại vì lưu quá nhiều, không phải quá ít.
- **Các lỗi phổ biến** bao gồm context overflow, retrieval bỏ sót, bộ nhớ lỗi thời, memory bloat, rò rỉ thông tin cá nhân, và nhầm lẫn các tầng bộ nhớ.

Bộ nhớ là thứ biến LLM từ một cỗ máy trả lời thông minh nhưng hay quên thành một agent thực sự biết chúng ta và cải thiện theo thời gian.

---

_Nguồn: [AI Agent Memory - Outcome School](https://outcomeschool.com/blog/ai-agent-memory) by Amit Shekhar_