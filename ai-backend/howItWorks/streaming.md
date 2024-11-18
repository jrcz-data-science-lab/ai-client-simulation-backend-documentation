# Overview  
This FastAPI application provides real-time audio transcription using OpenAI's Whisper model. It allows clients to send audio data over HTTP or WebSocket, processes the audio, transcribes it, and optionally sends the transcription to an external API. Additionally, it supports generating audio from the transcription using a Text-to-Speech (TTS) model.


Environment Variables
---------------------

The application utilizes the following environment variables:

-   **`API_PORT`**: The port on which the FastAPI application will run (default: `1000`).
-   **`WHISPER_MODEL`**: The Whisper model to be loaded (default: `base`).
-   **`TEXT_TO_SPEECH`**: The port for the Text-to-Speech service (default: `base`).

You can set these variables in your environment or use a `.env` file.

Endpoints
---------

### GET /

This endpoint performs a health check to verify that the server is running and the Whisper model is loaded.

#### Response:

-   `200 OK`: Returns a message indicating the Whisper model in use.

Example Response:


```json
{
  "message": "The model is running on base model"
}
```

### POST /process-audio

This HTTP endpoint accepts base64-encoded audio data, transcribes it using OpenAI's Whisper model, and returns the transcription.

#### Request Body:


```json
{
  "username": "user123",
  "audio": "base64_encoded_audio_data"
}
```

-   `username`: A string representing the user's unique identifier.
-   `audio`: The audio data in base64 encoding (e.g., a WebM or WAV file).

#### Response:

Returns the transcription result.

```json
{
  "username": "user123",
  "transcription": "This is the transcribed text."
}
```

### POST /send-transcription

This endpoint triggers the sending of the transcribed text to an external API and sends the synthesized audio (via TTS) to the user.

#### Request Body:

```json
{
  "username": "user123"
}
```

-   `username`: The unique identifier for the user.

#### Response:


```json
{
  "text": "This is the transcribed text.",
  "voice": "audio/user123_audio.wav"
}
```

-   `text`: The response from the external API after sending the transcription.
-   `voice`: The URL of the generated audio file from the Text-to-Speech service.

### GET /get-audio/{username}

This endpoint serves the synthesized audio file (in WAV format) generated from the transcription.

#### Parameters:

-   `username`: The user's unique identifier.

#### Response:

-   `200 OK`: Returns the audio file (`audio/{username}_audio.wav`).
-   `404 Not Found`: If the audio file does not exist.



Functions
---------

### `process_wav_bytes(webm_bytes: bytes, sample_rate: int = 12000)`

Processes incoming WebM audio bytes to convert and load them as a waveform suitable for transcription.

#### Parameters:

-   `webm_bytes` (bytes): The audio bytes in WebM format.
-   `sample_rate` (int): The sample rate for audio processing (default: 12000).

#### Returns:

-   The processed audio waveform, ready for transcription by Whisper.

### `send_transcription_to_apigateway(file_path: str, username: str)`

Sends the transcription text to an external API for further processing.

#### Parameters:

-   `file_path` (str): The path to the transcription text file.
-   `username` (str): The user's unique identifier.

#### Returns:

-   The JSON response from the external API, if successful.

### `send_transcription_to_tts(text: str, username: str)`

Sends the transcription text to a Text-to-Speech (TTS) service and saves the resulting audio.

#### Parameters:

-   `text` (str): The transcription text to convert to speech.
-   `username` (str): The user's unique identifier.

#### Side Effects:

-   Saves the resulting audio as a WAV file in the `audio/` directory under the user's name (`audio/{username}_audio.wav`).