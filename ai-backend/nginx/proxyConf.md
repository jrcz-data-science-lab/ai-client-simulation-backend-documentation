# NGINX SSL and Proxy Configuration Explanation

This NGINX configuration defines two server blocks: one for handling HTTPS requests and the other for redirecting HTTP requests to HTTPS. It also sets up reverse proxying to multiple backend services based on different URL paths.

---

## First Server Block (HTTPS)

This block listens for secure HTTPS connections on port `443` using SSL/TLS.

### `listen 443 ssl;` and `listen [::]:443 ssl;`

- **Description**: This directive tells NGINX to listen for incoming connections on port `443` for both IPv4 and IPv6, using SSL for encryption.
  
### `server_name your_ip;`

- **Description**: Specifies the server name (or IP address) that this server block will respond to. In this case, it listens for requests sent to the IP `your_ip`.
  
### SSL Settings

The following directives configure SSL/TLS for secure communication.

#### `ssl_certificate /etc/nginx/ssl/servercert.pem;`

- **Description**: Specifies the path to the SSL certificate file used to encrypt communication.

#### `ssl_certificate_key /etc/nginx/ssl/serverkey.pem;`

- **Description**: Specifies the path to the private key file corresponding to the SSL certificate. This key is used to decrypt incoming encrypted connections.

#### `ssl_protocols TLSv1.2 TLSv1.3;`

- **Description**: Defines the TLS protocols that are allowed. In this case, only TLS versions 1.2 and 1.3 are supported, which are considered secure.

#### `ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256';`

- **Description**: Defines the allowed cipher suites for SSL/TLS connections. The specified ciphers offer strong encryption while ensuring compatibility with modern clients.

### Proxy Configuration

The following `location` blocks define how requests for different paths should be proxied to various backend services.

#### `location / {}`

- **Description**: For requests to the root path (`/`), this block proxies traffic to an internal service running on `localhost:{your_frontend_port}`.

  - **proxy_pass https://localhost:{your_frontend_port};**: Proxies the request to an HTTPS backend running on port `{your_frontend_port}`.
  - **include proxy_params;**: Includes additional parameters for proxying, such as forwarding the original client IP and headers.
  - **proxy_set_header Upgrade $http_upgrade;** and **proxy_set_header Connection 'upgrade';**: These headers are necessary for handling WebSocket upgrades.
  - **proxy_http_version 1.1;**: Forces the use of HTTP/1.1 to support WebSocket connections.

#### `location /api/ {}`

- **Description**: Proxies requests for paths starting with `/api/` to an internal service running on `localhost:{your_api_port}`.
  - **proxy_pass http://localhost:{your_api_port}/;**: Proxies HTTP traffic to the backend service.

#### `location /ollama/ {}`

- **Description**: Proxies requests for paths starting with `/ollama/` to `localhost:11434`.
  - **proxy_pass http://localhost:11434/;**: Proxies HTTP traffic to the backend service.

#### `location /streaming/ {}`

- **Description**: Proxies requests for paths starting with `/streaming/` to a service running on `localhost:{your_streaming_port}`.

  - **proxy_set_header Upgrade $http_upgrade;** and **proxy_set_header Connection 'upgrade';**: These headers are necessary for WebSocket connections.
  - **proxy_http_version 1.1;**: Forces HTTP/1.1 for WebSocket support.

#### `location /tts/ {}`

- **Description**: Proxies requests for paths starting with `/tts/` to a service on `localhost:{your_tts_port}`.

  - **proxy_pass http://localhost:{your_tts_port}/;**: Proxies HTTP traffic to the backend service.

---

## Second Server Block (HTTP to HTTPS Redirect)

This server block handles HTTP requests and redirects them to HTTPS.

### `listen 80;` and `listen [::]:80;`

- **Description**: This tells NGINX to listen for incoming HTTP connections on port `80` (both IPv4 and IPv6).

### `server_name your_ip;`

- **Description**: The server name for this block, matching the IP address `your_ip`.

### HTTP to HTTPS Redirect

#### `return 301 https://$host$request_uri;`

- **Description**: Redirects all incoming HTTP requests to HTTPS. The `301` status code indicates a permanent redirect.
  - **$host**: Preserves the original hostname from the request.
  - **$request_uri**: Includes the original requested URI (path and query string).
  
  This ensures that any HTTP request is automatically redirected to its HTTPS equivalent.

---

## Summary

This NGINX configuration handles:
- **HTTPS traffic** (on port `443`) with SSL/TLS encryption, including the following reverse proxy setups:
  - Root path (`/`) proxies to `localhost:{your_frontend_port}` (with WebSocket support).
  - `/api/`, `/ollama/`, `/streaming/`, and `/tts/` paths proxy to their respective backend services.
- **HTTP traffic** (on port `80`) is redirected to the HTTPS version of the same request, ensuring all traffic is secured.

### SSL and Proxy Highlights:
- SSL encryption is enabled using TLSv1.2 and TLSv1.3 protocols with strong cipher suites.
- Requests are proxied to various internal services based on the path, with WebSocket support enabled for certain paths (`/`, `/streaming/`).
