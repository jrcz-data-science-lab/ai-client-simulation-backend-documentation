# API Interaction Script

This Python script demonstrates how to interact with an API gateway and a streaming service using the `requests` library. It includes functions for making `GET` and `POST` requests, and it tests the connection to the services defined by the API and streaming URLs.

## Environment Variables

The script uses two environment variables to configure the ports for the API gateway and the streaming service:

- `API_PORT`: The port for the API gateway (default is `1200`).
- `STREAMING_PORT`: The port for the streaming service (default is `1000`).

You can set these variables in your environment or use the defaults provided.

## Functions

### `post_request(url, payload)`

Performs a `POST` request to the specified `url` with the given `payload`.

#### Parameters

- `url` (str): The URL to which the request is sent.
- `payload` (dict): The JSON payload to be sent in the request body.

#### Returns

- (dict): The JSON response from the server.

### `get_request(url)`

Performs a `GET` request to the specified `url`.

#### Parameters

- `url` (str): The URL to which the request is sent.

#### Returns

- (dict): The JSON response from the server.

## Main Execution

The script contains a main execution block that:

1. Defines the URLs for the API gateway and the streaming service based on the configured ports.
2. Tests the connection by sending `GET` requests to both services and prints their responses.
3. Sends a `POST` request to the API gateway with a prompt payload, which includes:
   - `model`: The name of the model (e.g., "llama3.1").
   - `text`: The text prompt (e.g., "Hi!").
4. Prints the response text from the `POST` request.
