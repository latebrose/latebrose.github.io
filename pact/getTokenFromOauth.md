#

spring-security-oauth、pact

1. yml配置
```
security:
  oauth2:
    client:
      clientId: account-service
      clientSecret: account
      accessTokenUri: http://localhost:8888/uaa/oauth/token
      grant-type: client_credentials
      scope: server
    resource:
      user-info-uri: http://localhost:8888/uaa/user/current
```

2. 
```
/**
 * pact工具
 */
public class PactUtils {

    public static Map<String, String> getJsonHeaders() {
        Map<String, String> jsonHeaders = new HashMap<>();
        jsonHeaders.put("Content-Type", "application/json;charset=UTF-8");
        return jsonHeaders;
    }

    public static PactDslResponse addTokenPactResponse(PactDslWithProvider provider) {
        return provider
                .given("")
                .uponReceiving("get token")
                .path("/uaa/oauth/token")
                .method("POST").headers(getTokenHeaders())
                // account服务的授权类型和范围
                .body("grant_type=client_credentials&scope=server")
                .willRespondWith()
                .headers(getJsonHeaders())
                .status(200)
                // 随便返回个token，骗过oauth客户端
                .body(new PactDslJsonBody()
                        .stringType("access_token", "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzY29wZSI6WyJzZXJ2ZXIiXSwiZXhwIjoxNTQ4MTAyNDc3LCJqdGkiOiJjNGMwMmVkZS00ZGNhLTRjNTUtODE1Ny1iZGIyOGUxOTU2NWIiLCJjbGllbnRfaWQiOiJhY2NvdW50LXNlcnZpY2UifQ.SgmkF-wRqoudLPgP8pa8KlW2OvKvp0zNJoc-p_EDzrALbKCbqgw7FJUSv6IXIGBGzVXe229Vrj2uPXU4Zxvmp1ZErAyvDvUa5YqQflqNjpmzr5sSNQ0ns49fWB_5VkA0FKOzKcireHPBEl0yP_5dwIPsNuuwZZiAT_p7DMkj-wJAgbFheooUiXuq4A5Cp-iYwNkdIVYVSijM6k_jOMWQSHhhRtgBBNqZ-E81yp-meEWrH6oErF5rzZdPpntn6gbeyqb3pLTQ2Nix2l81VSR3mZwhcAucK4qctaf6FqgPFj8pjY7dzZO2HRxuPDkg7jomqfs31JS0jFYri1fK7Hg-IQ")
                        .stringValue("token_type", "bearer")
                        .integerType("expires_in", 43199)
                        .stringValue("scope", "server")
                        .stringType("jti", "c4c02ede-4dca-4c55-8157-bdb28e19565b")
                );
    }

    /**
     * 生成能获取oauth token的pact response
     * @param consumer 契约的调用方
     * @param provider 契约的提供方，不是特殊情况下，应该都是auth-service
     * @return
     */
    public static PactDslResponse getTokenPactResponse(String consumer, String provider) {
        return addTokenPactResponse(ConsumerPactBuilder
                .consumer(consumer)
                .hasPactWith(provider));
    }

    private static Map<String, String> getTokenHeaders() {
        Map<String, String> tokenHeaders = new HashMap<>();

        // body数据格式
        tokenHeaders.put("Content-Type", "application/x-www-form-urlencoded;charset=UTF-8");
        // account服务登录auth服务的密钥（account-service:account的base64）
        tokenHeaders.put("Authorization", "Basic YWNjb3VudC1zZXJ2aWNlOmFjY291bnQ=");

        return tokenHeaders;
    }
}
```