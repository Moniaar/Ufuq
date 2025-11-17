# Ufuq AI Tutoring Platform

An offline-first AI tutoring system using Model Context Protocol (MCP) for refugee and underserved students. Works locally via Wi-Fi hotspot with optional cloud integration.

## 🌟 Features

### Core Capabilities
- **Offline-First**: Works without internet using local AI models
- **Multi-Agent System**: Specialized agents for tutoring, translation, quizzes, and content retrieval
- **Bilingual Support**: Arabic and English with real-time translation
- **Progressive Web App**: Works on phones, tablets, and computers
- **Local Network Access**: Students connect via Wi-Fi hotspot or LAN
- **Cloud Sync**: Syncs progress when internet is available

### MCP Agents
1. **Tutor Agent** - Explains concepts, answers questions
2. **Translator Agent** - Arabic ↔ English translation
3. **Quiz Agent** - Generates and grades assessments
4. **Content Agent** - Retrieves from stored educational materials (RAG)
5. **Sync Agent** - Syncs data to cloud when online

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- 4GB+ RAM (for local models)
- Wi-Fi hotspot capability or local network

### Installation

```bash
# Clone the repository
git clone <your-repo-url>
cd ufuq

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Download models (first time only)
python download_models.py
```

### Configuration

Create a `.env` file:

```env
# Optional: Add API keys for online mode
OPENAI_API_KEY=your_key_here
SAMBANOVA_API_KEY=your_key_here
MODAL_API_KEY=your_key_here

# Debug mode
DEBUG=True
```

### Running the Server

```bash
# Start the Flask backend
python app.py
```

The server will run on `http://0.0.0.0:5000` (accessible on local network)

### Connecting Students

1. **Enable Wi-Fi Hotspot** on the host device
2. Students connect to the hotspot
3. Students open browser and navigate to: `http://<host-ip>:5000`
4. Start learning!

## 📁 Project Structure

```
ufuq/
├── app.py                 # Main Flask application
├── mcp_runtime.py         # MCP agent orchestration hub
├── agents.py              # All 5 MCP agents
├── storage.py             # Database and caching
├── config.py              # Configuration management
├── requirements.txt       # Python dependencies
├── download_models.py     # Model download script
├── data/
│   ├── ufuq.db           # SQLite database
│   ├── cache/            # Offline cache
│   ├── vectordb/         # ChromaDB vector storage
│   └── content/          # Educational PDFs/materials
└── frontend/             # React PWA (separate)
    └── ...
```

## 🔧 API Endpoints

### Chat (Tutor Agent)
```bash
POST /api/chat
{
  "student_id": "student_123",
  "message": "Explain Newton's laws",
  "language": "en"
}
```

### Translation
```bash
POST /api/translate
{
  "text": "Hello, how are you?",
  "source_lang": "en",
  "target_lang": "ar"
}
```

### Generate Quiz
```bash
POST /api/quiz/generate
{
  "student_id": "student_123",
  "topic": "Physics - Newton's Laws",
  "difficulty": "medium",
  "num_questions": 5,
  "language": "en"
}
```

### Submit Quiz
```bash
POST /api/quiz/submit
{
  "student_id": "student_123",
  "quiz_id": 1,
  "answers": ["A", "B", "C", "D", "A"]
}
```

### Retrieve Content (RAG)
```bash
POST /api/content/retrieve
{
  "query": "What is photosynthesis?",
  "subject": "biology",
  "language": "en"
}
```

### Student Progress
```bash
GET /api/student/progress?student_id=student_123
```

### Sync Data
```bash
POST /api/sync
{
  "student_id": "student_123"
}
```

### Check Mode
```bash
GET /api/mode
```

## 🤖 MCP Architecture

### How It Works

1. **Student Request** → Flask receives HTTP request
2. **MCP Router** → Routes to appropriate agent
3. **Agent Execution** → Agent processes with local/cloud models
4. **Mode Detection** → Automatically switches between offline/online
5. **Response** → Returns result to student
6. **Storage** → Saves interaction locally, syncs when online

### Agent Selection Logic

```python
# MCP Runtime automatically routes based on request type
mcp_runtime.route_request(
    agent_type="tutor",  # or "translator", "quiz", "content", "sync"
    payload={...}
)
```

### Offline ↔ Online Switching

The system automatically detects internet connectivity:
- **Offline**: Uses Hugging Face models (FLAN-T5, Helsinki-NLP)
- **Online**: Uses OpenAI GPT-4 for better quality
- **Hybrid**: Falls back to offline if online fails

## 📚 Adding Educational Content

### Ingest PDFs for RAG

```python
# Create a content ingestion script
from content_ingestion import ingest_pdf

# Add textbooks
ingest_pdf('path/to/physics_textbook.pdf', subject='physics')
ingest_pdf('path/to/math_textbook.pdf', subject='mathematics')
```

### Manual Content Addition

```python
from agents import ContentAgent
from config import Config

config = Config()
content_agent = ContentAgent(config)

# Add content manually
content_agent.collection.add(
    documents=["Newton's First Law: Objects at rest stay at rest..."],
    metadatas=[{"subject": "physics", "source": "textbook_ch1"}],
    ids=["physics_newton_1"]
)
```

## 🌐 Cloud Integration (Optional)

### OpenAI Setup
```python
# Set API key in .env
OPENAI_API_KEY=sk-...
```

### Modal Setup (for serverless functions)
```bash
pip install modal
modal token new
```

### SambaNova Setup
```python
# Add SambaNova API key
SAMBANOVA_API_KEY=your_key
```

## 🧪 Testing

```bash
# Run tests
pytest tests/

# Test specific agent
pytest tests/test_tutor_agent.py

# Test offline mode
FORCE_OFFLINE=True pytest tests/
```

## 📱 Frontend Integration

The React PWA connects to these endpoints. Example:

```javascript
// Chat with tutor
const response = await fetch('http://<host-ip>:5000/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    student_id: 'student_123',
    message: 'Explain gravity',
    language: 'en'
  })
});

const data = await response.json();
console.log(data.response);  // AI tutor's explanation
```

## 🔐 Security Considerations

- **Local Network Only**: By default, only accessible on LAN
- **No Authentication**: Add authentication for production
- **Data Privacy**: All data stored locally, synced optionally
- **API Keys**: Keep in `.env`, never commit to git

## 🐛 Troubleshooting

### Models Not Loading
```bash
# Re-download models
python download_models.py --force
```

### Port Already in Use
```bash
# Change port in app.py
app.run(host='0.0.0.0', port=5001)
```

### Out of Memory
- Reduce model sizes in `config.py`
- Use smaller models (e.g., `flan-t5-small` instead of `base`)
- Disable local model loading and use online-only mode

### Connection Issues
- Check firewall settings
- Ensure all devices on same network
- Use IP address instead of localhost

## 📊 Performance Optimization

### For Low-Resource Devices
```python
# In config.py, use smaller models
'offline': {
    'tutor': 'google/flan-t5-small',  # Instead of base
    'translator': 'Helsinki-NLP/opus-mt-tc-big-en-ar'  # Smaller variant
}
```

### Batch Processing
```python
# Process multiple quiz questions at once
quiz_agent.execute({
    'action': 'batch_generate',
    'topics': ['physics', 'math', 'chemistry']
})
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test offline and online modes
5. Submit a pull request

## 📄 License

MIT License - Feel free to use for educational purposes

## 🆘 Support

For issues during the hackathon:
- Check logs: `tail -f logs/ufuq.log`
- Test mode: `curl http://localhost:5000/health`

## 🚀 Next Steps

- [ ] Add voice input/output
- [ ] Implement peer-to-peer quiz sharing
- [ ] Add gamification (points, badges)
- [ ] Multi-user collaboration features
- [ ] Teacher dashboard for monitoring
- [ ] More languages (French, Spanish)
- [ ] Offline video lessons

---

Built with ❤️ for underserved students worldwide
