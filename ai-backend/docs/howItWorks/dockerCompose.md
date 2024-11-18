# Docker Compose Setup for AudioStreamer Project

This document describes the Docker Compose configuration for the AudioStreamer project, which consists of multiple services that work together to provide an audio streaming solution with GPU support.

## Services

### 1. Ollama Container

- **Container Name**: `ollama`
- **Image**: `ollama/ollama`
- **Volumes**:
  - `./data/ollama:/root/.ollama`: Mounts the local `data/ollama` directory to the container's `.ollama` directory.
  - `./custom:/custom`: Mounts the local `custom` directory to the container.
- **Network**: Connected to `apinetwork`.
- **Environment Variables**:
  - `OLLAMA_ORIGINS`: Set to `http://host.docker.internal`.
- **Deploy Configuration**:
  - **Resources**:
    - **Reservations**:
      - **Devices**: 
        - **Driver**: `nvidia`
        - **Count**: `all`
        - **Capabilities**: `gpu` (Enables GPU support for the container)
- **Command**: Runs `serve` and then `list`.

### 2. API Gateway

- **Container Name**: `apigateway`
- **Build Context**: `./api`
- **Ports**: Exposes the API on `${API_PORT}`.
- **Volumes**:
  - `./api:/usr/src/app`: Mounts the local `api` directory to the container's application directory.
- **Depends On**:
  - `ollama-container`: Waits for this service to be ready.
- **Network**: Connected to `apinetwork`.
- **Deploy Configuration**:
  - **Resources**:
    - **Reservations**:
      - **Devices**: 
        - **Driver**: `nvidia`
        - **Count**: `all`
        - **Capabilities**: `gpu` (Enables GPU support for the container)
- **Health Check**:
  - **Test**: Uses `curl` to check if the service is responsive at `http://ollama:11434`.
  - **Interval**: 1 minute 30 seconds.
  - **Timeout**: 30 seconds.
  - **Retries**: 5 attempts.
  - **Start Period**: 30 seconds.

### 3. Backend Code

- **Container Name**: `backend`
- **Build Context**: `./code`
- **Ports**: Exposes the backend on `${BACKEND_PORT}`.
- **Volumes**:
  - `./code:/usr/src/app`: Mounts the local `code` directory to the container's application directory.
- **Depends On**:
  - `apigateway`: Requires the API gateway to be healthy before starting.
  - `streaming`: Requires the streaming service to be healthy before starting.
- **Network**: Connected to `apinetwork`.
- **Deploy Configuration**:
  - **Resources**:
    - **Reservations**:
      - **Devices**: 
        - **Driver**: `nvidia`
        - **Count**: `all`
        - **Capabilities**: `gpu` (Enables GPU support for the container)

### 4. Streaming Service

- **Container Name**: `streaming`
- **Build Context**: `./streaming`
- **Ports**: Exposes the streaming service on `${STREAMING_PORT}`.
- **Volumes**:
  - `./streaming:/usr/src/app`: Mounts the local `streaming` directory to the container's application directory.
- **Depends On**:
  - `ollama-container`: Waits for this service to be ready.
- **Network**: Connected to `apinetwork`.
- **Deploy Configuration**:
  - **Resources**:
    - **Reservations**:
      - **Devices**: 
        - **Driver**: `nvidia`
        - **Count**: `all`
        - **Capabilities**: `gpu` (Enables GPU support for the container)
- **Health Check**:
  - **Test**: Uses `curl` to check if the service is responsive at `http://ollama:11434`.
  - **Interval**: 1 minute 30 seconds.
  - **Timeout**: 30 seconds.
  - **Retries**: 5 attempts.
  - **Start Period**: 30 seconds.

### 5. Text-to-Speech (TTS) Service

- **Container Name**: `tts`
- **Build Context**: `./tts`
- **Ports**: Exposes the TTS service on `${TEXT_TO_SPEECH}`.
- **Volumes**:
  - `./tts:/usr/src/app`: Mounts the local `tts` directory to the container's application directory.
- **Network**: Connected to `apinetwork`.
- **Deploy Configuration**:
  - **Resources**:
    - **Reservations**:
      - **Devices**: 
        - **Driver**: `nvidia`
        - **Count**: `all`
        - **Capabilities**: `gpu` (Enables GPU support for the container)

## Networks

- **apinetwork**: 
  - **Driver**: `bridge`

This setup provides a comprehensive architecture for an audio streaming application utilizing Docker containers with GPU acceleration for relevant services.
