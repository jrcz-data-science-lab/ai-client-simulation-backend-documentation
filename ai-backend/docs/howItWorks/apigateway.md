# API gateway
This FastAPI application allows users to interact with AI models through a variety of endpoints, including those for generating text prompts, managing models, caching conversation history, and handling real-time transcription through WebSockets. The application integrates with the Ollama service, which handles large language models (LLMs).

Key Features
------------

-   **Conversation Caching**: The application stores conversation history in a file for each user, allowing the model to generate more context-aware responses.
-   **Model Management**: Users can pull, delete, or create models in Ollama using dedicated API endpoints.
-   **Real-time Transcription**: Transcription functionality is available through WebSocket and HTTP endpoints, enabling audio data to be processed and transcribed in real time.

API Endpoints
-------------

### 1\. Health Check

```http
GET /
```

**Description**: Verifies that the API is running.

**Response**:
```json
"Apigateway is running"
```
### 2\. Generate Prompt

```http
POST /api/prompt/
```

**Description**: Generates a response from the AI model based on the provided text prompt.

**Request Body**:


```json
{
    "model": "gemma2:2b",
    "text": "What is the meaning of life?",
    "username": "user123"
}
```

-   **model**: The name of the AI model to use (e.g., `gemma2:2b`).
-   **text**: The input text for the AI model to generate a response.
-   **username**: A unique identifier for the user (used for caching conversation history).

**Response**:

 ```json
{
    "model": "gemma2:2b",
    "text": "The meaning of life is to seek happiness, purpose, and connection with others."
}
```

**Functionality**:

-   The model is invoked using the provided prompt.
-   The AI's response is cached along with the prompt for future context-aware interactions.

### 3\. Pull Model

```http
POST /api/pull/
```

**Description**: Pulls a model from the Ollama service, making it available for use.

**Request Body**:


```json
{
    "model_name": "llama3.1"
}
```

**Response**:

```json
{
    "status": "Model pulled successfully"
}
```

### 4\. Delete Model

```http
POST /api/delete/
```

**Description**: Deletes a model from the Ollama service.

**Request Body**:

```json
{
    "model": "llama3.1"
}
```

**Response**:

```json
{
    "status": "Model deleted successfully"
}
```

### 5\. Create Custom Model

```http
POST /api/create/
```

**Description**: Creates a new custom model using a base model and personality traits.

**Request Body**:



```json
{
    "name": "custom-chat",
    "basemodel": "gemma2:2b",
    "personality": "Friendly and engaging"
}
```

**Response**:

```json
{
    "status": "Model created successfully"
}
```

### 6\. WebSocket Proxy for Transcription

```http
WebSocket /proxy-transcribe
```

**Description**: Establishes a WebSocket connection for real-time transcription of audio data. The audio is sent to a streaming service for processing, and the transcribed text is sent back to the client.

**WebSocket Flow**:

1.  The client connects to the WebSocket endpoint and sends audio data (along with the username) to the server.
2.  The server sends the audio data to an external streaming service for transcription.
3.  The transcribed text is sent back to the client.

**Request**:

-   The client sends audio data to the server with a `username` key.


```json
{
    "username": "user123",
    "audio": "<audio_base64>" // does not to be discribed
}
```

**Response**:

```json
{
    "username": "user123",
    "transcription": "Hello, this is a test transcription."
}
```

### 7\. Send Transcription via HTTP

```http
POST /proxy-send-transcription
```

**Description**: Sends a transcription request to the streaming service, triggering transcription of audio for a specified user.

**Request Body**:

```json
{
    "username": "user123"
}
```

**Response**:


```json
{
    "status": "Transcription sent successfully"
}
```

**Error Handling**:

-   If there's an issue with the request, the server responds with an error message.

### 8\. Get Audio


```http
GET /audio/{username}
```

**Description**: Fetches the audio file for a specific user from an external streaming service.

**Response**:

-   The audio file is streamed as the response, with the correct `Content-Type` (default: `audio/wav`).

* * * * *

Conversation Caching
--------------------

The application uses a conversation cache to store and recall past interactions, improving the context for future requests. The cache is updated each time a new prompt is submitted, and the full conversation history is saved to a file for each user. This allows the AI to generate responses that account for previous interactions.

### How It Works:

-   When a prompt is received, the AI generates a response using the conversation history.
-   The conversation is then stored in a file specific to the user (`conversationMemory/{username}_memory.txt`).
-   The conversation history is formatted and used as context in future requests, allowing the AI to provide more coherent and contextually aware answers.

**Example of Conversation Cache Format**:


```vbnet
User: What is the meaning of life?
AI: The meaning of life is to seek happiness, purpose, and connection with others.

User: Can you elaborate on happiness?
AI: Happiness is often found in meaningful relationships, personal growth, and living with purpose.
```

### File Storage:

-   The conversation is saved in a file named `conversationMemory/{username}_memory.txt` (where `{username}` is the unique identifier).
-   The conversation cache is updated with each new user interaction.

* * * * *

Model Invocation
----------------

The AI models are invoked through the `Ollama` class, which interacts with the Ollama service. This service manages the large language models (LLMs) and allows the API to generate text based on the provided prompt.


```python
llm = Ollama(model=model_name, base_url="http://ollama:11434")
```

Each time a prompt is received, the entire conversation history (stored in `previous_convos`) is included as part of the prompt, enabling context-sensitive responses from the model.

```python
full_prompt = f'{previous_convos}\nUser: {prompt_text}\nAI:'
return llm.invoke("Continue the conversation like a human\n" + full_prompt)
```

* * * * *

Error Handling
--------------

The application uses standard HTTP error codes for failure scenarios:

-   **400 Bad Request**: The request is malformed or missing required fields.
-   **500 Internal Server Error**: There is a problem on the server, such as an issue with the Ollama service or external dependencies.

Example of error response for an unsuccessful transcription request:

```json
{
    "error": "HTTP error occurred",
    "details": "Error details here"
}
```



### Summary of Available Endpoints

| Endpoint | Method | Description |
| --- | --- | --- |
| `/` | GET | Health check |
| `/api/prompt/` | POST | Generate prompt response |
| `/api/pull/` | POST | Pull a model from Ollama |
| `/api/delete/` | POST | Delete a model from Ollama |
| `/api/create/` | POST | Create a custom model |
| `/proxy-transcribe` | WebSocket | Real-time audio transcription |
| `/proxy-send-transcription` | POST | Send transcription request |
| `/audio/{username}` | GET | Fetch user audio file |

* * * * *

This API allows seamless interaction with the Ollama LLM service while handling user-specific conversation histories and supporting real-time transcription of audio data.