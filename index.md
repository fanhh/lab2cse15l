Here is my code:
```
import com.sun.net.httpserver.HttpServer;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.util.HashMap;
import java.util.Map;

public class ChatServer {
    private static String chatHistory = "";

    public static void main(String[] args) throws IOException {
        int port = 2000;
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);
        server.createContext("/add-message", new MessageHandler());
        server.setExecutor(null); 
        server.start();
        System.out.println("Server started on port " + port);
    }

    static class MessageHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            Map<String, String> params = queryToMap(exchange.getRequestURI().getQuery());
            String user = params.get("user");
            String message = params.get("s");
            synchronized (chatHistory) {
                chatHistory += user + ": " + message + "\n";
            }

            exchange.sendResponseHeaders(200, chatHistory.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(chatHistory.getBytes());
            os.close();
        }

        private Map<String, String> queryToMap(String query) {
            Map<String, String> result = new HashMap<>();
            for (String param : query.split("&")) {
                String[] entry = param.split("=");
                if (entry.length > 1) {
                    result.put(entry[0], entry[1]);
                } else {
                    result.put(entry[0], "");
                }
            }
            return result;
        }
    }
}


```





![First Screenshot](f1.png)

1. The handle method in MessageHandler is called.
2. The relevant arguments: user: "jpolitz", message: "Hello".
3. The elevant fields of the class changes: chatHistory changes from "" to "jpolitz: Hello\n".


!![second Screenshot](f2.png)
1. The handle method in MessageHandler is called again.
2. The relevant arguments: user: "yash", message: "How are you".
3. The elevant fields of the class changes: chatHistory changes from "jpolitz: Hello\n" to "jpolitz: Hello\nyash: How are you\n"