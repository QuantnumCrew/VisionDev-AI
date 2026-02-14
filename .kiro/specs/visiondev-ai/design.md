# Design Document: VisionDev AI

## Overview

VisionDev AI is a context-aware visual AI assistant that monitors a developer's screen in real-time, automatically detects errors, extracts relevant information using OCR, and provides AI-powered explanations and debugging assistance. The system is designed to help developers, especially students and beginners, learn faster and debug more efficiently.

The architecture follows a modular design with clear separation between screen capture, computer vision processing, OCR text extraction, AI analysis, and user interface components. The system uses Python with FastAPI for the backend, OpenCV for computer vision, Tesseract for OCR, OpenAI API or local models for AI processing, and React.js for the frontend.

Key design principles:
- **Modularity**: Each component (screen capture, vision processing, OCR, AI analysis) is independent and replaceable
- **Performance**: Minimal impact on developer's system through efficient processing and resource management
- **Privacy**: User control over data capture and option for local AI processing
- **Extensibility**: Support for multiple AI models and configurable processing pipelines
- **User-Centric**: Beginner-friendly explanations and intuitive interface

## Architecture

### System Components

The system consists of the following major components:

1. **Screen Capture Module**: Captures screenshots at configurable intervals using platform-specific APIs
2. **Vision Processor**: Analyzes screenshots using OpenCV to detect error patterns and regions of interest
3. **OCR Engine**: Extracts text from screenshots using Tesseract OCR
4. **AI Analyzer**: Sends context to AI models and processes responses
5. **Voice Command Handler**: Processes voice input for hands-free interaction
6. **Backend API**: FastAPI server coordinating all components
7. **Frontend Interface**: React.js web application for user interaction

### Data Flow

```
Developer Screen
    ↓
Screen Capture Module (periodic capture)
    ↓
Vision Processor (error detection)
    ↓
OCR Engine (text extraction)
    ↓
AI Analyzer (context analysis)
    ↓
Backend API (coordination)
    ↓
Frontend Interface (display results)
```

### Technology Stack

**Backend:**
- Python 3.9+
- FastAPI for REST API
- OpenCV for computer vision
- Tesseract OCR for text extraction
- OpenAI API or local models (e.g., Ollama) for AI processing
- Pillow for image manipulation
- SpeechRecognition for voice commands

**Frontend:**
- React.js 18+
- Tailwind CSS for styling
- Axios for API communication
- Web Speech API for voice input

**Development:**
- Git for version control
- VS Code as primary IDE
- pytest for backend testing
- Jest and React Testing Library for frontend testing

## Components and Interfaces

### Screen Capture Module

**Responsibility**: Capture screenshots of the developer's screen at configurable intervals.

**Implementation Approach**:
- Use `mss` library for cross-platform screen capture (faster than PIL)
- Support multiple monitor configurations
- Implement configurable capture intervals (1-10 seconds)
- Store captures temporarily in memory as numpy arrays for efficient processing

**Interface**:
```python
class ScreenCaptureModule:
    def __init__(self, capture_interval: int = 3, monitor_index: int = 0):
        """Initialize screen capture with interval in seconds"""
        
    def start_capture(self) -> None:
        """Start periodic screen capture"""
        
    def stop_capture(self) -> None:
        """Stop screen capture"""
        
    def get_latest_capture(self) -> Optional[np.ndarray]:
        """Get the most recent screenshot as numpy array"""
        
    def set_capture_interval(self, interval: int) -> None:
        """Update capture interval"""
```

**Key Design Decisions**:
- Use threading for non-blocking periodic capture
- Store only the latest capture to minimize memory usage
- Convert screenshots to numpy arrays immediately for OpenCV compatibility

### Vision Processor

**Responsibility**: Analyze screenshots to detect errors and regions of interest.

**Implementation Approach**:
- Use OpenCV for image processing
- Detect error patterns through color analysis (red text, error icons)
- Use template matching for common error indicators
- Apply OCR selectively to regions of interest to improve performance
- Implement confidence scoring for error detection

**Interface**:
```python
class VisionProcessor:
    def __init__(self, confidence_threshold: float = 0.7):
        """Initialize with error detection confidence threshold"""
        
    def analyze_screenshot(self, image: np.ndarray) -> AnalysisResult:
        """Analyze screenshot for errors and return regions of interest"""
        
    def detect_error_patterns(self, image: np.ndarray) -> List[ErrorRegion]:
        """Detect error patterns using color and template matching"""
        
    def extract_regions_of_interest(self, image: np.ndarray, 
                                    error_regions: List[ErrorRegion]) -> List[np.ndarray]:
        """Extract image regions for OCR processing"""

class AnalysisResult:
    has_error: bool
    confidence: float
    error_regions: List[ErrorRegion]
    error_type: Optional[str]  # 'syntax', 'runtime', 'compilation'

class ErrorRegion:
    x: int
    y: int
    width: int
    height: int
    confidence: float
```

**Key Design Decisions**:
- Focus on red text detection (HSV color space filtering)
- Use template matching for common IDE error icons
- Expand detected regions slightly to capture full context
- Skip processing if no errors detected to save resources

### OCR Engine

**Responsibility**: Extract text from screenshots and error regions.

**Implementation Approach**:
- Use Tesseract OCR via pytesseract wrapper
- Preprocess images for better OCR accuracy (grayscale, contrast enhancement)
- Configure Tesseract for code-optimized text extraction
- Preserve formatting including line breaks and indentation

**Interface**:
```python
class OCREngine:
    def __init__(self, language: str = 'eng'):
        """Initialize Tesseract OCR"""
        
    def extract_text(self, image: np.ndarray) -> str:
        """Extract text from image region"""
        
    def extract_text_with_layout(self, image: np.ndarray) -> StructuredText:
        """Extract text preserving layout information"""
        
    def preprocess_image(self, image: np.ndarray) -> np.ndarray:
        """Preprocess image for better OCR accuracy"""

class StructuredText:
    text: str
    lines: List[str]
    confidence: float
```

**Key Design Decisions**:
- Use PSM (Page Segmentation Mode) 6 for uniform text blocks
- Apply grayscale conversion and adaptive thresholding
- Use OEM (OCR Engine Mode) 3 for best accuracy
- Cache Tesseract configuration for performance

### AI Analyzer

**Responsibility**: Send context to AI models and process responses for display.

**Implementation Approach**:
- Support multiple AI backends (OpenAI API, local models via Ollama)
- Construct prompts with error context, code snippets, and user intent
- Format responses for beginner-friendly display
- Implement retry logic for API failures
- Cache similar queries to reduce API costs

**Interface**:
```python
class AIAnalyzer:
    def __init__(self, model_type: str = 'openai', model_name: str = 'gpt-4'):
        """Initialize with AI model configuration"""
        
    def analyze_error(self, context_data: ContextData) -> AIResponse:
        """Analyze error context and generate explanation"""
        
    def process_voice_command(self, command: str, 
                             context_data: ContextData) -> AIResponse:
        """Process voice command with current context"""
        
    def format_response(self, raw_response: str) -> FormattedResponse:
        """Format AI response for display"""

class ContextData:
    error_text: str
    error_type: Optional[str]
    screenshot_region: Optional[np.ndarray]
    additional_context: Dict[str, Any]

class AIResponse:
    explanation: str
    suggestions: List[str]
    code_examples: List[CodeExample]
    learning_resources: List[str]

class FormattedResponse:
    title: str
    explanation: str
    suggestions: List[str]
    code_examples: List[CodeExample]
    timestamp: datetime
```

**Key Design Decisions**:
- Use system prompts to enforce beginner-friendly explanations
- Include error type and context in prompts for better responses
- Implement exponential backoff for API retries
- Support switching between cloud and local models without code changes

### Voice Command Handler

**Responsibility**: Process voice input and trigger appropriate system actions.

**Implementation Approach**:
- Use SpeechRecognition library with Google Speech API
- Implement command pattern matching for recognized commands
- Support commands: "Explain this error", "Fix line X", "What is wrong with this code"
- Provide feedback when commands are recognized

**Interface**:
```python
class VoiceCommandHandler:
    def __init__(self):
        """Initialize speech recognition"""
        
    def start_listening(self) -> None:
        """Start listening for voice commands"""
        
    def stop_listening(self) -> None:
        """Stop listening"""
        
    def process_command(self, audio_input) -> VoiceCommand:
        """Convert audio to text and parse command"""

class VoiceCommand:
    command_type: str  # 'explain', 'fix', 'analyze'
    parameters: Dict[str, Any]
    raw_text: str
    confidence: float
```

**Key Design Decisions**:
- Use push-to-talk or hotkey activation to avoid false triggers
- Implement fuzzy matching for command recognition
- Provide visual feedback when listening
- Fall back to text input if speech recognition fails

### Backend API

**Responsibility**: Coordinate all components and provide REST API for frontend.

**Implementation Approach**:
- Use FastAPI for high-performance async API
- Implement WebSocket for real-time updates
- Use background tasks for screen capture and processing
- Implement proper error handling and logging

**API Endpoints**:
```python
# Screen Capture
POST /api/capture/start - Start screen monitoring
POST /api/capture/stop - Stop screen monitoring
GET /api/capture/status - Get capture status

# Analysis
GET /api/analysis/latest - Get latest analysis result
GET /api/analysis/history - Get analysis history
POST /api/analysis/manual - Manually trigger analysis

# Voice Commands
POST /api/voice/command - Process voice command
POST /api/voice/start - Start voice listening
POST /api/voice/stop - Stop voice listening

# Configuration
GET /api/config - Get current configuration
PUT /api/config - Update configuration

# WebSocket
WS /ws/updates - Real-time updates for new analyses
```

**Key Design Decisions**:
- Use async/await for non-blocking operations
- Implement CORS for frontend communication
- Use Pydantic models for request/response validation
- Store analysis history in memory (configurable limit)

### Frontend Interface

**Responsibility**: Provide intuitive UI for viewing AI responses and controlling the system.

**Implementation Approach**:
- Use React.js with functional components and hooks
- Implement real-time updates via WebSocket
- Use Tailwind CSS for responsive design
- Display screenshot with highlighted error regions
- Show analysis history with timestamps

**Key Components**:
```typescript
// Main dashboard
<Dashboard />
  - <ControlPanel /> // Start/stop, settings
  - <ScreenshotViewer /> // Display captured screen with highlights
  - <AnalysisDisplay /> // Show AI explanation and suggestions
  - <HistoryPanel /> // Recent analyses
  - <VoiceControl /> // Voice command button

// Configuration
<SettingsModal />
  - Capture interval
  - AI model selection
  - Voice command settings
  - Privacy options
```

**Key Design Decisions**:
- Use React Context for global state management
- Implement lazy loading for history items
- Use WebSocket for real-time updates instead of polling
- Provide keyboard shortcuts for common actions
- Mobile-responsive design for viewing on tablets

## Data Models

### Screenshot Data

```python
class Screenshot:
    id: str
    timestamp: datetime
    image_data: np.ndarray
    monitor_index: int
    width: int
    height: int
```

### Error Detection Result

```python
class ErrorDetectionResult:
    screenshot_id: str
    has_error: bool
    confidence: float
    error_regions: List[ErrorRegion]
    error_type: Optional[str]
    processing_time_ms: float
```

### Analysis Record

```python
class AnalysisRecord:
    id: str
    timestamp: datetime
    screenshot_id: str
    error_text: str
    error_type: Optional[str]
    ai_response: AIResponse
    voice_command: Optional[str]
```

### Configuration

```python
class SystemConfiguration:
    capture_interval: int  # seconds
    capture_enabled: bool
    ai_model_type: str  # 'openai' or 'local'
    ai_model_name: str
    voice_enabled: bool
    error_detection_threshold: float
    max_history_items: int
    privacy_mode: bool  # if True, don't store screenshots
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property Reflection

After analyzing all acceptance criteria, I've identified the following testable properties and eliminated redundancy:

**Redundancy Analysis:**
- Properties 1.1, 1.2, and 1.3 (screen capture) can be combined into a comprehensive capture behavior property
- Properties 2.1 and 2.3 (error detection and region marking) are related and can be combined
- Properties 3.1 and 3.2 (OCR extraction) can be unified into a single extraction property
- Properties 7.1-7.4 (API endpoints) are all examples of the same endpoint existence pattern
- Properties 8.1-8.4 (configuration) are all examples of configuration management

**Consolidated Properties:**
The following properties provide unique validation value without redundancy:

1. Screen capture timing and retrieval
2. Error detection with region identification
3. OCR text extraction with formatting preservation
4. AI analysis invocation and response formatting
5. Voice command processing
6. API error handling
7. Configuration hot-reload
8. Privacy controls
9. Error recovery and retry logic
10. System resilience under component failure

### Core Properties

**Property 1: Screen Capture Consistency**
*For any* valid capture interval configuration between 1 and 10 seconds, starting the Screen_Capture_Module should result in screenshots being captured at approximately that interval, and each captured screenshot should be retrievable from memory.
**Validates: Requirements 1.1, 1.2, 1.3**

**Property 2: Error Detection Completeness**
*For any* screenshot containing error indicators (red text, error icons, or stack traces), the Vision_Processor should detect at least one error region with confidence above the threshold, and each detected error should have valid region coordinates.
**Validates: Requirements 2.1, 2.2, 2.3**

**Property 3: Error Type Classification**
*For any* detected error, the Vision_Processor should classify it as one of the known error types (syntax, runtime, compilation, or unknown), and the classification should be consistent for similar error patterns.
**Validates: Requirements 2.4**

**Property 4: Efficient Processing Skip**
*For any* screenshot without error indicators, the Vision_Processor should return a result indicating no errors and skip OCR processing, completing in less time than error-containing screenshots.
**Validates: Requirements 2.5**

**Property 5: OCR Text Extraction**
*For any* image region containing text, the OCR_Engine should extract the text and return it as Context_Data, preserving line breaks and indentation where the text is clearly formatted.
**Validates: Requirements 3.1, 3.2, 3.3, 3.4**

**Property 6: OCR Font Robustness**
*For any* text rendered in common development fonts (monospace fonts like Consolas, Monaco, Courier) at sizes between 10pt and 16pt, the OCR_Engine should successfully extract the text with reasonable accuracy.
**Validates: Requirements 3.5**

**Property 7: AI Model Invocation**
*For any* Context_Data provided to the AI_Analyzer, the system should invoke the configured AI model (cloud or local) and return a response, regardless of which model type is configured.
**Validates: Requirements 4.1, 4.6**

**Property 8: Response Formatting Consistency**
*For any* raw AI response, the AI_Analyzer should format it into a FormattedResponse with all required fields (title, explanation, suggestions, timestamp), and the formatted response should be valid for display.
**Validates: Requirements 4.5**

**Property 9: Code Example Inclusion**
*For any* AI response that contains code blocks (indicated by markdown code fences), the formatted response should include those code blocks in the code_examples field.
**Validates: Requirements 4.4**

**Property 10: Voice Command Recognition**
*For any* voice input containing recognized command keywords ("explain", "fix", "analyze"), the Voice_Command_Handler should parse it into a VoiceCommand with the correct command_type and parameters.
**Validates: Requirements 5.2, 5.4**

**Property 11: Voice Command Feedback**
*For any* successfully recognized voice command, the Voice_Command_Handler should provide feedback (audio or visual) indicating recognition success.
**Validates: Requirements 5.5**

**Property 12: API Error Response Quality**
*For any* API request that fails due to invalid input or server error, the Backend_API should return an HTTP error response with a descriptive error message in the response body.
**Validates: Requirements 7.5**

**Property 13: Concurrent Request Handling**
*For any* set of concurrent API requests to different endpoints, the Backend_API should process all requests without data corruption or race conditions, and each request should receive the correct response.
**Validates: Requirements 7.6**

**Property 14: Configuration Hot-Reload**
*For any* configuration parameter change (capture interval, AI model, detection threshold), the system should apply the new configuration immediately without requiring restart, and subsequent operations should use the new configuration.
**Validates: Requirements 8.5**

**Property 15: Capture Privacy Control**
*For any* state where screen capture is disabled, the Screen_Capture_Module should not capture any screenshots, and attempting to retrieve a capture should return None or empty result.
**Validates: Requirements 9.1**

**Property 16: Screenshot Retention Policy**
*For any* screenshot captured during normal operation (privacy mode enabled), the Backend_API should not store the screenshot permanently, and it should be cleared from memory after processing is complete.
**Validates: Requirements 9.2**

**Property 17: Data Minimization**
*For any* Context_Data sent to cloud-based AI models, the transmitted data should contain only the extracted text and error type, not the raw screenshot image data.
**Validates: Requirements 9.3**

**Property 18: Capture Failure Recovery**
*For any* screenshot capture failure, the Backend_API should log the error, continue operation, and successfully capture the next screenshot when the capture interval elapses.
**Validates: Requirements 11.1**

**Property 19: OCR Retry Logic**
*For any* OCR extraction failure, the Backend_API should retry the extraction exactly once, and if both attempts fail, should return an error result without further retries.
**Validates: Requirements 11.2**

**Property 20: Component Failure Isolation**
*For any* individual component failure (Screen_Capture_Module, Vision_Processor, OCR_Engine, or AI_Analyzer), the Backend_API should continue operating and other components should remain functional.
**Validates: Requirements 11.5**

## Error Handling

### Error Categories

1. **Screen Capture Errors**
   - Permission denied: Request permissions and inform user
   - Monitor not found: Fall back to primary monitor
   - Capture timeout: Log error and continue with next interval

2. **Vision Processing Errors**
   - Invalid image format: Log error and skip processing
   - Processing timeout: Return no-error result and continue
   - Out of memory: Reduce image resolution and retry

3. **OCR Errors**
   - Tesseract not installed: Display clear setup instructions
   - Text extraction failure: Retry once, then return empty text
   - Invalid image region: Log error and skip OCR

4. **AI Analysis Errors**
   - API key missing: Display configuration error
   - API rate limit: Implement exponential backoff
   - Network timeout: Retry with backoff, then display error
   - Invalid response: Log error and display generic message

5. **Voice Command Errors**
   - Microphone not available: Display error and disable voice
   - Recognition failure: Prompt user to try again
   - Unrecognized command: Provide list of valid commands

6. **API Errors**
   - Invalid request: Return 400 with validation errors
   - Server error: Return 500 with error ID for debugging
   - Not found: Return 404 with helpful message

### Error Recovery Strategies

**Graceful Degradation:**
- If Vision_Processor fails, fall back to full-screenshot OCR
- If OCR fails, allow manual text input
- If AI model unavailable, display cached similar responses
- If voice recognition fails, provide text input alternative

**Retry Logic:**
- OCR failures: 1 retry with image preprocessing
- AI API calls: 3 retries with exponential backoff (1s, 2s, 4s)
- Screen capture: Continue with next interval
- Voice recognition: Prompt for immediate retry

**User Communication:**
- Display clear error messages in the UI
- Provide actionable steps for resolution
- Log detailed errors for debugging
- Show system status indicators

## Testing Strategy

### Dual Testing Approach

VisionDev AI requires both unit tests and property-based tests for comprehensive coverage:

**Unit Tests** focus on:
- Specific examples of error detection (e.g., Python syntax error, Java NullPointerException)
- Edge cases like empty screenshots, malformed images, or missing configuration
- Integration points between components (e.g., Vision_Processor → OCR_Engine)
- API endpoint behavior with specific request/response examples
- Error conditions and recovery logic

**Property-Based Tests** focus on:
- Universal properties that hold for all inputs (e.g., all captures are retrievable)
- Randomized testing across different configurations and inputs
- Invariants that must be maintained (e.g., privacy controls always prevent capture when disabled)
- Round-trip properties (e.g., configuration set then get returns same value)
- Comprehensive input coverage that unit tests cannot achieve

Together, these approaches provide comprehensive coverage: unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across all possible inputs.

### Property-Based Testing Configuration

**Framework Selection:**
- **Python Backend**: Use `hypothesis` library for property-based testing
- **React Frontend**: Use `fast-check` library for property-based testing

**Test Configuration:**
- Each property test MUST run minimum 100 iterations
- Each test MUST include a comment tag referencing the design property
- Tag format: `# Feature: visiondev-ai, Property {number}: {property_text}`

**Example Property Test Structure:**

```python
from hypothesis import given, strategies as st
import pytest

# Feature: visiondev-ai, Property 1: Screen Capture Consistency
@given(capture_interval=st.integers(min_value=1, max_value=10))
def test_screen_capture_consistency(capture_interval):
    """For any valid capture interval, screenshots should be captured and retrievable"""
    module = ScreenCaptureModule(capture_interval=capture_interval)
    module.start_capture()
    time.sleep(capture_interval + 0.5)  # Wait for at least one capture
    
    screenshot = module.get_latest_capture()
    assert screenshot is not None
    assert isinstance(screenshot, np.ndarray)
    
    module.stop_capture()
```

### Unit Testing Strategy

**Backend Unit Tests (pytest):**
- Test each component class independently with mocked dependencies
- Test API endpoints with FastAPI TestClient
- Test error handling with simulated failures
- Test configuration management
- Aim for 80%+ code coverage

**Frontend Unit Tests (Jest + React Testing Library):**
- Test component rendering with various props
- Test user interactions (button clicks, form submissions)
- Test WebSocket message handling
- Test state management
- Test error boundary behavior

**Integration Tests:**
- Test full pipeline: Screen Capture → Vision → OCR → AI → Display
- Test WebSocket real-time updates
- Test voice command end-to-end flow
- Test configuration changes affecting behavior

### Test Organization

```
tests/
├── unit/
│   ├── test_screen_capture.py
│   ├── test_vision_processor.py
│   ├── test_ocr_engine.py
│   ├── test_ai_analyzer.py
│   ├── test_voice_handler.py
│   └── test_api.py
├── property/
│   ├── test_capture_properties.py
│   ├── test_vision_properties.py
│   ├── test_ocr_properties.py
│   ├── test_ai_properties.py
│   ├── test_api_properties.py
│   └── test_privacy_properties.py
├── integration/
│   ├── test_full_pipeline.py
│   ├── test_error_recovery.py
│   └── test_configuration.py
└── frontend/
    ├── components/
    │   ├── Dashboard.test.tsx
    │   ├── AnalysisDisplay.test.tsx
    │   └── VoiceControl.test.tsx
    └── property/
        └── ui_properties.test.tsx
```

### Continuous Testing

- Run unit tests on every commit
- Run property tests on pull requests
- Run integration tests before deployment
- Monitor test execution time and optimize slow tests
- Maintain test coverage reports
