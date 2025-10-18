# Overview
Cung cấp nền tảng phỏng vấn mô phỏng có trí tuệ nhân tạo, nơi ứng viên có thể:
- Trả lời câu hỏi phỏng vấn qua chat hoặc giọng nói.
- Bộ câu hỏi được cá nhân hoá dựa trên CV và năng lực hiện tại.
- AI interviewer sinh câu hỏi, phân tích câu trả lời, và chọn câu hỏi tiếp theo dựa vào ngữ cảnh, vector DB, và dữ liệu kỹ năng. 
- Kết thúc phỏng vấn, AI interviewer sẽ đánh giá và đưa ra phản hồi chi tiết.

# Components
| #      | Thành phần                                               | Vai trò & mô tả chính                                                                                                                                                                            | Công nghệ đề xuất                                      |
| ------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| **1**  | **AI Interviewer Engine**                                | Trung tâm điều khiển logic phỏng vấn: sinh câu hỏi, phân tích câu trả lời, chọn câu hỏi tiếp theo dựa ngữ cảnh và vector DB.                                                                     | Python + OpenAI API / LangChain / Hugging Face |
| **2**  | **📄 CV Analyzer Component (Pre-Interview Preparation)** | Phân tích CV ứng viên để chuẩn bị câu hỏi: <br>• Trích xuất kỹ năng, kinh nghiệm.<br>• Sinh embedding CV để chọn câu hỏi khởi đầu phù hợp.<br>• Gợi ý chủ đề cần đánh giá.                       | Python / LangChain / spaCy / OpenAI Embeddings         |
| **3**  | **Vector Database**                                      | Lưu trữ embedding của câu hỏi, kỹ năng, chủ đề và câu trả lời; hỗ trợ semantic search để chọn câu hỏi kế tiếp hoặc xác định năng lực.                                                            | Pinecone                           |
| **4**  | **Chat UI / Frontend**                                   | Giao diện chat realtime giữa ứng viên và AI; hỗ trợ văn bản, emoji, hiển thị kết quả đánh giá.                                                                                                   | React + WebSocket / REST                     |
| **5**  | **🎤 Voice Answer Component (Speech-to-Text)**           | Cho phép ứng viên trả lời bằng giọng nói: <br>• Thu âm & chuyển sang văn bản.<br>• Hỗ trợ đa ngôn ngữ (EN, VI).<br>• Gửi transcript đến AI Interviewer Engine.                                   | Azure Speech-to-Text API                           |
| **6**  | **🗣️ AI Voice Response Component (Text-to-Speech)**     | Cho phép AI interviewer phản hồi hoặc đặt câu hỏi bằng giọng nói: <br>• Chuyển văn bản sang âm thanh tự nhiên.<br>• Hỗ trợ lựa chọn giọng nói & ngôn ngữ.<br>• Phát audio realtime cho ứng viên. | Edge TTS (Microsoft Text-to-Speech)                |
| **7**  | **Analytics & Feedback Service**                         | Thu thập dữ liệu trả lời, đánh giá năng lực, tổng hợp phản hồi & báo cáo cuối buổi phỏng vấn.                                                                                                    | LangChain + OpenAI GPT-4 / Claude / Llama 3 để phân tích & sinh phản hồi. scikit-learn / spaCy để đánh giá ngôn ngữ và cảm xúc.                       |
| **8** | **Question Bank Service**           | Lưu trữ bộ câu hỏi phỏng vấn (technical, behavioral, situational). Cho phép tagging, versioning, fine-tuning mô hình.                                                                            | Python + PostgreSQL                   |

# Main flows
## 1. Giai đoạn Chuẩn bị (Scan CV & sinh chủ đề)
```mermaid
sequenceDiagram
    actor Candidate
    participant ChatUI as Chat UI / Frontend
    participant CVAnalyzer as 📄 CV Analyzer Component
    participant VectorDB as 🧠 Vector Database
    participant AIEngine as 🤖 AI Interviewer Engine

    Candidate->>ChatUI: Upload CV file
    ChatUI->>CVAnalyzer: Send CV for analysis
    CVAnalyzer->>VectorDB: Generate & store CV embeddings
    VectorDB-->>CVAnalyzer: Confirm embeddings stored
    CVAnalyzer-->>ChatUI: Return extracted skills & suggested topics
    ChatUI-->>Candidate: Display preparation summary
    ChatUI->>AIEngine: Notify readiness (skills, topics)
    AIEngine-->>ChatUI: Acknowledged
```

## 2. Giai đoạn Phỏng vấn (Hỏi – Đáp realtime)
```mermaid
sequenceDiagram
    actor Candidate
    participant ChatUI as Chat UI / Frontend
    participant AIEngine as 🤖 AI Interviewer Engine
    participant VectorDB as 🧠 Vector Database
    participant QBank as 📚 Question Bank Service
    participant STT as 🎤 Speech-to-Text
    participant TTS as 🗣️ Text-to-Speech
    participant Analytics as 📊 Analytics & Feedback Service

    %% --- Start Interview ---
    Candidate->>ChatUI: Start interview
    ChatUI->>AIEngine: Request first question
    AIEngine->>VectorDB: Query similar question embeddings (based on CV topics)
    VectorDB-->>AIEngine: Return question candidates
    AIEngine->>QBank: Fetch selected question
    QBank-->>AIEngine: Return question details
    AIEngine-->>ChatUI: Send question text
    ChatUI-->>TTS: Convert question text to speech
    TTS-->>Candidate: Play AI voice question

    %% --- Candidate answers ---
    Candidate->>STT: Speak answer
    STT-->>AIEngine: Send transcript text
    AIEngine->>VectorDB: Compare answer embeddings & evaluate quality
    VectorDB-->>AIEngine: Return similarity & semantic score
    AIEngine->>Analytics: Send answer evaluation (score, sentiment, reasoning)
    Analytics-->>AIEngine: Acknowledged

    alt More questions remain
        AIEngine->>VectorDB: Retrieve next suitable question
        VectorDB-->>AIEngine: Return next question candidate
        AIEngine-->>ChatUI: Send next question
        ChatUI-->>TTS: Convert to speech & play
        TTS-->>Candidate: Play next question
    else Interview finished
        AIEngine-->>ChatUI: Notify interview end
    end

```

## 3. Giai đoạn Kết thúc (Đánh giá & Báo cáo)
```mermaid
sequenceDiagram
    actor Candidate
    participant ChatUI as Chat UI / Frontend
    participant AIEngine as 🤖 AI Interviewer Engine
    participant Analytics as 📊 Analytics & Feedback Service

    AIEngine->>Analytics: Send final interview summary (scores, metrics, transcript)
    Analytics->>Analytics: Aggregate results & generate report
    Analytics-->>AIEngine: Acknowledged

    AIEngine-->>ChatUI: Notify interview completion
    ChatUI->>Analytics: Request final feedback report
    Analytics-->>ChatUI: Return detailed feedback & improvement suggestions
    ChatUI-->>Candidate: Display performance summary & insights

```
