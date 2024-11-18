# Proxy Header Configuration Explanation

This section contains directives for setting HTTP headers in NGINX when acting as a reverse proxy. These headers are used to pass information about the client request and the original protocol used, ensuring that the upstream server can correctly interpret the request.

---

### `proxy_set_header Host $http_host;`

- **Description**: This directive sets the `Host` header in the request that is forwarded to the upstream server. 
- **$http_host**: This variable represents the original `Host` header sent by the client in the HTTP request.
- **Purpose**: Ensures that the upstream server sees the original `Host` header, allowing it to properly respond based on the requested domain or subdomain.

### `proxy_set_header X-Real-IP $remote_addr;`

- **Description**: This directive forwards the client's original IP address to the upstream server by setting the `X-Real-IP` header.
- **$remote_addr**: This variable contains the IP address of the client making the request.
- **Purpose**: Passes the client's real IP address to the upstream server, which may be useful for logging, analytics, or security purposes, especially when NGINX is behind a load balancer or proxy.

### `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`

- **Description**: This directive sets the `X-Forwarded-For` header, which is a comma-separated list of IP addresses representing the chain of proxies that forwarded the request.
- **$proxy_add_x_forwarded_for**: This variable appends the client's IP address (`$remote_addr`) to the existing `X-Forwarded-For` header, or creates it if it does not exist.
- **Purpose**: Allows the upstream server to trace the entire path of the client request, especially when there are multiple proxies involved. This is crucial for logging the real client IP in multi-layered proxy setups.

### `proxy_set_header X-Forwarded-Proto $scheme;`

- **Description**: This directive sets the `X-Forwarded-Proto` header, which indicates the protocol (HTTP or HTTPS) that was used by the client to connect to the original NGINX server.
- **$scheme**: This variable represents the protocol used in the original request, either `http` or `https`.
- **Purpose**: Communicates to the upstream server whether the original client connection was secured (HTTPS) or not (HTTP). This is useful for generating correct links or handling mixed-content issues.

---

## Summary

These `proxy_set_header` directives are used in reverse proxy configurations to pass relevant client information from the NGINX proxy to the upstream server. The headers help the upstream server correctly handle:
- The **requested domain** (`Host` header),
- The **real client IP address** (`X-Real-IP`, `X-Forwarded-For`),
- And the **protocol used** (`X-Forwarded-Proto`).

This setup ensures that the upstream server has all the necessary context to handle requests properly in a proxy or load-balanced environment.
