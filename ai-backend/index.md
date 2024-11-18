# General overview of the workings

```
.
├── backend
│   ├── api
│   ├── code
│   ├── custom
│   ├── data
│   ├── docker-compose.yml
│   ├── docs
│   ├── .env
│   ├── .env.example
│   ├── streaming
│   └── tts
├── frontend
│   ├── node_modules
│   ├── package.json
│   ├── package-lock.json
│   ├── public
│   ├── server.js
│   └── ssl
├── .gitignore
└── readme.md
```

This project structure represents a multi-service application, organized into distinct components as follows:

- [**api**](./howItWorks/apigateway.md): Contains a Dockerfile, the main Python script (`main.py`), and a `requirements.txt` file listing dependencies for the API service.
- [**code**](./howItWorks/code.md): Another service similar to the API, with its own Dockerfile, main logic script (`main.py`), and `requirements.txt` for dependencies.
- **data**: Stores data or models related to the Ollama service.
- [**docker-compose.yml**](./howItWorks/dockerCompose.md): Defines how these services interact using Docker Compose.
- [**streaming**](./howItWorks/streaming.md): A separate service like API and code, responsible for handling real-time data or streams.
- [**text to speech**](./howItWorks/tts.md): A service that deals with text to speech from the incoming text.


### NGINX documentation
The NGINX reverse proxy is a key part of the architecture, acting as a traffic manager that directs incoming requests to the appropriate services based on the URL paths. It ensures secure communication, handles WebSocket connections for real-time features, and redirects HTTP traffic to HTTPS for enhanced security.

- [**NGINX code**](./nginx/nginxCode.md)
- [**NGINX configuration**](./nginx/nginxConf.md)
- [**NGINX proxy parameters**](./nginx/proxyParams.md)
- [**NGINX proxy configuration**](./nginx/proxyConf.md)

### Functionality

The system facilitates real-time audio transcription through a WebSocket connection from the frontend. As the audio streams, it is transcribed in real-time. Once the user ends the stream, the inputs are concatenated and passed to the API gateway. 

