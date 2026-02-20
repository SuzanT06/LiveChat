# Real-Time Chat Box (Spring Boot WebSocket Demo)

Small demo Spring Boot application that demonstrates a real-time chat using STOMP over WebSocket (with SockJS fallback). The project includes a minimal UI (`src/main/resources/templates/chat.html`) that connects to a server-side STOMP endpoint and broadcasts messages to subscribed clients.

## Key points

- Spring Boot application (entry: `src/main/java/com/chatbox/Application/Application.java`)
- WebSocket/STOMP endpoint: `/chat` (configured in `WebSocketConfig`)
- Client sends to `/app/sendMessage` (mapped by `ChatController`) and subscribes to `/topic/messages`
- Minimal message model: `ChatMessage` (fields: `id`, `sender`, `content`)


## Prerequisites

- JDK 17 (the project `pom.xml` declares `<java.version>17</java.version>`)
- Windows (instructions below use the included Maven wrapper `mvnw.cmd`)
- Internet access to download frontend CDN assets (Bootstrap, SockJS, stomp.js) used by the HTML client


## Build & run (Windows / cmd.exe)

Open a `cmd.exe` terminal in the project root (where `mvnw.cmd` and `pom.xml` live) and run:

```bat
:: Clean and build the project
mvnw.cmd clean package

:: Run the Spring Boot application
mvnw.cmd spring-boot:run
```

Or run the packaged jar (after `clean package`):

```bat
java -jar target\Application-0.0.1-SNAPSHOT.jar
```

By default the app listens on port 8080. Open the browser to:

http://localhost:8080/chat

This page loads the chat UI and connects the client to the WebSocket/STOMP endpoint.


## Endpoints & messaging

- HTTP UI
  - GET `/chat` — returns the Thymeleaf template `chat.html` (controller mapping in `ChatController`)

- WebSocket/STOMP (server-side configuration in `WebSocketConfig`)
  - STOMP endpoint (SockJS fallback): `/chat`
    - The config currently allows origin `http://localhost:8080` (see `WebSocketConfig.java`)
  - Message broker
    - Application destination prefix: `/app`
    - Simple broker destination prefix: `/topic`

- Controller mappings (see `ChatController.java`)
  - Client -> Server (send): send to `/app/sendMessage` (annotated with `@MessageMapping("/sendMessage")`)
  - Server -> Clients (broadcast): messages are forwarded to `/topic/messages` (via `@SendTo("/topic/messages")`)

- Message payload (JSON) — corresponds to `ChatMessage`:

```json
{
  "sender": "Alice",
  "content": "Hello everyone!"
}
```

The client-side `chat.html` shows how to send and receive messages using SockJS + STOMP: it subscribes to `/topic/messages` and sends to `/app/sendMessage`.


## How the client works (quick)

- On page load the client opens a SockJS connection to `/chat` and upgrades to STOMP.
- The client subscribes to `/topic/messages` and appends incoming messages to the page.
- When the user clicks "Send" the client sends a JSON message to `/app/sendMessage`.


## Project file map (important files)

- `src/main/java/com/chatbox/Application/Application.java` — application entry point
- `src/main/java/com/chatbox/Application/config/WebSocketConfig.java` — STOMP/WebSocket endpoints and broker config
- `src/main/java/com/chatbox/Application/controller/ChatController.java` — message mappings and `/chat` route
- `src/main/java/com/chatbox/Application/model/ChatMessage.java` — message DTO (Lombok is used for getters/setters)
- `src/main/resources/templates/chat.html` — minimal chat UI (Bootstrap + SockJS + stomp.js)
- `pom.xml` — Maven build file and dependencies


## Troubleshooting & notes

- Port conflicts: if port 8080 is in use, set `server.port` in `src/main/resources/application.properties` (or pass `-Dserver.port=<port>` to the JVM).

- CORS/origins: `WebSocketConfig` currently allows `http://localhost:8080`. If you serve the UI from a different origin, update `setAllowedOrigins(...)` accordingly or use a wildcard with caution.

- SockJS fallback: If the browser cannot establish a native WebSocket, SockJS provides fallback transports. The client code in `chat.html` uses SockJS and stomp.js.

- Lombok: Lombok is declared in `pom.xml` and used in `ChatMessage`. Make sure your IDE supports annotation processing (enable it) to avoid missing getters/setters at compile-time.


## Running tests

Run unit/integration tests (if any) with:

```bat
mvnw.cmd test
```


## Contributing

- Fork or clone the repository and create a feature branch.
- Run the app locally and add your changes.
- Ensure you keep the project Java version (17) unless you update `pom.xml`.
- Create a PR with a short description and testing steps.


## License

No license is provided in the repository. Add a `LICENSE` file if you want to specify one.


## Questions / Next steps

If you want, I can:
- Add cross-platform run instructions (macOS/Linux)
- Add a Dockerfile for containerized runs
- Improve the UI (display timestamps, different styles)
- Add user join/leave notifications or persistence

