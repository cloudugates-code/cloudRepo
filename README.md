import org.springframework.ai.azure.openai.AzureOpenAIClientBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AzureAiConfig {

    @Bean
    public MyCustomTokenCredential myCustomTokenCredential() {
        return new MyCustomTokenCredential(
                "https://fam-uat.kp.org/as/token.oauth2",
                "your-client-id",
                "your-client-secret",
                "your-scope"
        );
    }

    @Bean
    public AzureOpenAIClientBuilderCustomizer customizer(MyCustomTokenCredential credential) {
        return builder -> {
            // Base endpoint (everything before /chat/completions)
            builder.endpoint("https://api-gtwy-dev.kp.org/api/itops/siae/kpaigateway/v1r");
            builder.credential(credential);

            // Add custom headers (appId, sessionId)
            builder.addPolicy((context, next) -> {
                context.getHttpRequest().setHeader("appId", "myApp");
                context.getHttpRequest().setHeader("sessionId", "session123");
                return next.process();
            });
        };
    }
}
