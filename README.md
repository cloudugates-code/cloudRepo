import com.azure.core.credential.AccessToken;
import com.azure.core.credential.TokenCredential;
import com.azure.core.credential.TokenRequestContext;
import reactor.core.publisher.Mono;
import org.springframework.web.reactive.function.client.WebClient;

import java.time.OffsetDateTime;

public class MyCustomTokenCredential implements TokenCredential {

    private final String tokenUrl;
    private final String clientId;
    private final String clientSecret;
    private final String scope;

    private String cachedToken;
    private OffsetDateTime expiry;

    private final WebClient webClient = WebClient.create();

    public MyCustomTokenCredential(String tokenUrl, String clientId, String clientSecret, String scope) {
        this.tokenUrl = tokenUrl;
        this.clientId = clientId;
        this.clientSecret = clientSecret;
        this.scope = scope;
    }

    @Override
    public Mono<AccessToken> getToken(TokenRequestContext request) {
        return Mono.fromSupplier(() -> new AccessToken(getTokenSync(), expiry));
    }

    public synchronized String getTokenSync() {
        if (cachedToken == null || OffsetDateTime.now().isAfter(expiry)) {
            fetchToken();
        }
        return cachedToken;
    }

    private void fetchToken() {
        // POST to tokenUrl with form params
        TokenResponse response = webClient.post()
                .uri(tokenUrl)
                .bodyValue("grant_type=client_credentials&client_id=" + clientId +
                        "&client_secret=" + clientSecret + "&scope=" + scope)
                .retrieve()
                .bodyToMono(TokenResponse.class)
                .block();

        if (response == null || response.access_token == null) {
            throw new RuntimeException("Failed to fetch token from fam-uat");
        }

        this.cachedToken = response.access_token;
        this.expiry = OffsetDateTime.now().plusSeconds(response.expires_in - 60); // buffer 1 min
    }

    private static class TokenResponse {
        public String access_token;
        public long expires_in;
    }
}
