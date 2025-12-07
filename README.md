# ScamShield - Real-Time Scam Call Detection

**Software Engineering Project - September 2025, Team 8**

**ScamShield** is an AI-powered system that detects scam calls in real-time while the conversation is happening. It uses speech-to-text transcription and AI analysis to identify suspicious patterns and alert users immediately.

---

## Features

- **Real-Time Detection**: Analyzes calls every 10 seconds during the conversation
- **AI-Powered Analysis**: Uses Whisper for speech-to-text and Qwen LLM for scam pattern detection
- **Encrypted Communication**: AES-256-GCM encryption for secure data transmission
- **Live Probability Updates**: Users see scam probability updating in real-time (0-100%)
- **Comprehensive Logging**: All analysis stored in database for audit trails
- **Interactive UI**: Vue.js frontend with mobile-like call interface

---

## How It Works

```
Incoming Call
        ↓
Audio Recording (POST /trigger_processing)
        ↓
Process Every 10 Seconds
        ↓
Whisper AI (Speech → Text)
        ↓
Encrypt → Send → Decrypt
   AES-256-GCM secure transmission
        ↓
AI Analysis (Qwen LLM)
   Scam pattern detection
        ↓
Encrypt ← Receive ← Decrypt
   Secure response transmission
        ↓
Real-Time Probability (0-100%)
   Updates every 2 seconds
        ↓
Alert User if Scam Detected
```

---

## Technology Stack

### Backend
- **FastAPI** - REST API framework
- **Whisper** - OpenAI's speech-to-text model
- **Qwen2-1.5B-Instruct** - Base language model for scam detection
  - Fine-tuned with LoRA adapters using QLoRA for PEFT (Parameter Efficient Fine Tuning)
  - Optimized for low-resource deployment while maintaining accuracy
- **SQLModel** - Database ORM
- **Cryptography** - AES-256-GCM encryption
- **PyTorch** - ML framework

### Frontend
- **Vue.js 3** - Progressive JavaScript framework
- **Vue Router** - Client-side routing
- **Vite** - Build tool

### Testing
- **pytest** - Testing framework
- **pytest-asyncio** - Async test support

---

## Project Structure

```
ScamShield-Final/
├── app/                        # Backend application
│   ├── server.py               # Main API server (port 5000)
│   ├── ai_server.py            # AI analysis server (port 8000)
│   ├── audio_processor.py      # Audio processing & transcription
│   ├── database.py             # Database models (SQLModel)
│   ├── config.py               # Configuration settings
│   └── utils/
│       └── encryption.py       # AES-256-GCM encryption utilities
├── frontend/                   # Vue.js frontend
│   ├── src/
│   │   ├── views/              # Vue components
│   │   ├── router/             # Route definitions
│   │   └── assets/             # Images and static files
│   └── public/
│       └── audio/              # Sample audio recordings
├── tests/                      # Test suite (23 tests)
│   ├── test_api.py             # API endpoint tests
│   ├── test_database.py        # Database tests
│   ├── test_encryption.py      # Encryption tests
│   └── conftest.py             # Pytest fixtures
├── recordings/                 # Sample audio files
├── audio_chunks/               # Processed audio chunks
├── requirements.txt            # Python dependencies
├── pytest.ini                  # Pytest configuration
└── README.md                   # This file
```

---

## Setup Instructions

### Prerequisites

- **Python 3.8+**
- **Node.js 16+** and npm
- **FFmpeg** (for audio processing)

### 1. Clone the Repository

```bash
git clone <repository-url>
cd ScamShield-Final
```

### 2. Backend Setup

#### Create Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate
```

#### Install Dependencies

```bash
pip install -r requirements.txt
```

**Note**: This will install heavy AI models (~4GB). First run may take time.

#### Configure Environment (Optional)

Create a `.env` file:

```env
ENCRYPTION_PASSWORD=scamshield_password
CHUNK_DURATION_SECONDS=10
AI_SERVER_URL=http://localhost:8000/analyze
```

### 3. Frontend Setup

```bash
cd frontend
npm install
```

---

## Running the Application

You need to run **THREE** servers simultaneously:

### Terminal 1: AI Server (Port 8000)

```bash
python -m app.ai_server
```

This runs the Qwen LLM for scam detection.

### Terminal 2: Backend Server (Port 5000)

```bash
python -m app.server
```

This handles API requests and audio processing.

### Terminal 3: Frontend (Port 5173)

```bash
cd frontend
npm run dev
```

Access the application at: **http://localhost:5173**

---

## Running Tests

### Run All Tests

```bash
pytest tests/ -v
```

### Run Specific Test File

```bash
# API tests
pytest tests/test_api.py -v

# Database tests
pytest tests/test_database.py -v

# Encryption tests
pytest tests/test_encryption.py -v
```

### Test Coverage

| Module | Tests | Status |
|--------|-------|--------|
| API Endpoints | 5 | ✅ Passing |
| Encryption | 10 | ✅ Passing |
| Database | 8 | ✅ Passing |
| **Total** | **23** | ✅ **100%** |

**Execution Time**: ~2.5 seconds

---

## API Endpoints

### 1. Start Call Processing

```http
POST /trigger_processing
```

**Request Body**:
```json
{
  "audio_id": 1
}
```

**Response**:
```json
{
  "message": "Processing started",
  "session_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "file": "recording0.mp3"
}
```

### 2. Get Status

```http
GET /status/{session_id}
```

**Response**:
```json
{
  "status": "SCAM",
  "probability": 0.95,
  "transcription": "This is IRS calling...",
  "reason": "IRS impersonation detected"
}
```

### 3. Shutdown Server

```http
POST /shutdown
```

Gracefully shuts down the backend server.

---

## Security Features

### 1. End-to-End Encryption

- **Algorithm**: AES-256-GCM (Galois/Counter Mode)
- **Key Derivation**: PBKDF2-HMAC-SHA256 with 100,000 iterations
- **Purpose**: Encrypts transcripts before sending to AI server

### 2. Authenticated Encryption

- **GCM Mode**: Provides both confidentiality and integrity
- **Tamper Detection**: Rejects any modified or corrupted data
- **Authentication Tags**: 16-byte tags verify data authenticity

### 3. Secure Key Management

- **Random Salt**: 16-byte random salt per encryption
- **Random Nonce**: 12-byte random nonce per message
- **Key Size**: 256-bit encryption keys

### 4. Data Protection

- **In-Transit Encryption**: All API communications encrypted
- **Database Logging**: Audit trail for all analysis
- **No Password Storage**: Uses configuration-based shared secrets

---

## Scam Detection Criteria

The AI analyzes transcripts for high-risk indicators:

### Critical Triggers (High Severity)

1. **Financial Immediacy**: Demands for immediate money transfer via gift cards, cryptocurrency, wire transfers
2. **False Authority**: Impersonation of IRS, police, banks, or tech support
3. **Confidential Data Requests**: Requests for OTPs, PINs, passwords, security answers
4. **External Access**: Requests to download software or grant remote access
5. **Identity Avoidance**: Refusal to verify identity through standard protocols

### Severity Scoring

- **0.0 - 0.6**: Normal conversation, monitoring
- **0.6 - 0.8**: Suspicious indicators detected
- **0.8 - 1.0**: High confidence scam, alert user

---

## Sample Recordings

Three sample recordings are included in `recordings/`:

- **recording0.mp3**: banking KYC scam call
- **recording1.mp3**: Credit Card rewards scam call
- **recording2.mp3**: Normal call between friends

---

## Testing Strategy

### Unit Tests

- **API Tests**: Mock heavy AudioProcessor to avoid loading AI models
- **Encryption Tests**: Verify AES-256-GCM security implementation
- **Database Tests**: Test CRUD operations and query performance

### Mocking Approach

Tests use module-level mocking to avoid loading 6GB+ AI models:

```python
# Mock heavy imports BEFORE loading server
sys.modules['app.audio_processor'] = mock_audio_processor
```

**Benefits**:
- Tests run on any laptop (< 100MB RAM)
- Fast execution (~2.5 seconds)
- Core logic validated without AI dependencies

---


## Troubleshooting

### Issue: "No module named 'torch'"

**Solution**: Install PyTorch separately if needed:
```bash
pip install torch torchvision torchaudio
```

### Issue: FFmpeg not found

**Solution**: Install FFmpeg:

**Windows**: Download from https://ffmpeg.org/download.html
**macOS**: `brew install ffmpeg`
**Linux**: `sudo apt-get install ffmpeg`

### Issue: Tests failing with import errors

**Solution**: Ensure you're in the project root directory:
```bash
cd ScamShield-Final
pytest tests/ -v
```

### Issue: Frontend not connecting to backend

**Solution**: Verify both servers are running:
- AI Server: http://localhost:8000
- Backend Server: http://localhost:5000

---



## Acknowledgments

- **OpenAI Whisper** - Speech-to-text model
- **Qwen Team** - Language model for scam detection
- **FastAPI** - Modern Python web framework
- **Vue.js** - Progressive JavaScript framework


