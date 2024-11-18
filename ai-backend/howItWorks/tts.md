Overview
--------

This API provides functionality for converting text into speech. It allows users to send text data, which will then be processed using a text-to-speech (TTS) engine. The synthesized speech is returned as an audio file in WAV format. This API uses FastAPI as the framework and provides a simple interface for text-to-speech synthesis.

Endpoints
---------

### 1\. Root Endpoint

#### `GET /`

The root endpoint provides a welcome message and basic information about the API.

##### Response

-   **Status Code**: `200 OK`
-   **Response Body**:

    ```json
    {
      "message": "Welcome to the Text-to-Speech API"
    }
    ```

### 2\. Text-to-Speech Synthesis

#### `POST /synthesize`

This endpoint allows you to send a text string, and the API will return a WAV file of the synthesized speech.

##### Request Body

-   **Content-Type**: `application/json`
-   **Body**: A JSON object with the following structure:

    ```json
    {
      "text": "Your text here"
    }
    ```

    -   `text`: (string) The text you want to convert to speech.

##### Response

-   **Success**: If the synthesis is successful, the response will contain the audio file as a WAV file.

    **Status Code**: `200 OK`

    **Response Body**: The synthesized audio file will be returned as a binary stream with the following headers:

    -   **Content-Type**: `audio/wav`
    -   **Content-Disposition**: `attachment; filename="synthesized_audio.wav"`
-   **Error**: If there is an error (e.g., missing reference audio file or failure during synthesis), the response will contain an error message.

    **Example Error Response**:

    ```json
    {
      "error": "Reference audio file not found."
    }
    ```

#### Example Request


```bash
curl -X 'POST'\
  'http://{tts_url}/synthesize'\
  -H 'Content-Type: application/json'\
  -d '{
  "text": "Hello, world!"
}'
```

#### Example Success Response

The response will return a WAV file of the synthesized speech.

#### Example Error Response

If there is an issue (such as a missing reference audio file), the response will be:

```json
{
  "error": "Reference audio file not found."
}
```

Configuration
-------------

### Environment Variables

-   **TEXT_TO_SPEECH**: The port number on which the FastAPI server will run. If not set, the default value will be `1000`.

#### Example:


`export TEXT_TO_SPEECH=8080`

Logging
-------

The API includes logging at the `DEBUG` level. Logs will capture key events such as:

-   Accessing endpoints.
-   Command execution.
-   Any errors that occur during synthesis.

You can adjust the logging level as necessary to control verbosity.

Error Handling
--------------

The API includes basic error handling for issues such as:

-   Missing reference audio file (`refaudio/audio.mp3`).
-   Issues during the synthesis process.
-   File-related errors (e.g., file not found, file empty).

In case of an error, a JSON response will be returned with the error message, and the status code will indicate failure (e.g., `500 Internal Server Error`).

Local Development
-----------------



Command Line Synthesis (`f5-tts_infer-cli`)
-------------------------------------------

The actual synthesis of text to speech is handled by the `f5-tts_infer-cli` command-line tool. It is executed with the following parameters:

-   `-m E2-TTS`: Specifies the TTS model to use.
-   `-r ./refaudio/audio.mp3`: Specifies the reference audio file used for synthesis.
-   `-s`: An optional parameter for speaker selection.
-   `-t <text>`: The text to be synthesized into speech.

Ensure that the `f5-tts_infer-cli` tool is correctly installed and accessible from the system's `PATH`. The tool generates a WAV file (`infer_cli_out.wav`) that is returned to the client.

