# I/O error on POST request for "xxx": Connection reset; Connection reset

## 错误提示

I/O error on POST request for "http://cloud-gateway.cmp-stg.svc.cluster.local:31802/cv-console-iam/oauth/token": Connection reset; nested exception is java.net.SocketException: Connection reset

## 查到的解决方案：

加载证书，但是不知道为什么加载证书以及为什么要加载证书，等出现问题必须解决的时候再考虑。还有，同样调用相同的api，为什么别的没问题？

```java
@RequestMapping(value ="/gettokens", method = RequestMethod.POST, produces ="application/json")
public @ResponseBody ResponseEntity<TokenModel> GetTokens(@RequestBody RequestBodyJson requestBodyJson)
        throws KeyManagementException, NoSuchAlgorithmException, KeyStoreException {
    ResponseEntity<TokenModel> response = null;
    TrustStrategy acceptingTrustStrategy = (X509Certificate[] chain, String authType) -> true;
    SSLContext sslContext = org.apache.http.ssl.SSLContexts.custom()
            .loadTrustMaterial(null, acceptingTrustStrategy).build();
    SSLConnectionSocketFactory csf = new SSLConnectionSocketFactory(sslContext);
    CloseableHttpClient httpClient = HttpClients.custom()
            .setSSLSocketFactory(csf).build();
    HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory();
    requestFactory.setHttpClient(httpClient);
    RestTemplate restTemplate = new RestTemplate(requestFactory);      
    try {                          
        MultiValueMap<String, String> headers = new LinkedMultiValueMap<String, String>();
        Map map = new HashMap<String, String>();
        map.put("Content-Type","application/json");
        headers.setAll(map);
        HttpEntity< ? > _HttpEntityRequestBodyJson = new HttpEntity<>(requestBodyJson, headers);
        response= restTemplate.exchange(url, HttpMethod.POST,_HttpEntityRequestBodyJson, new ParameterizedTypeReference<TokenModel>() {});  
        System.out.println(response.getBody());
    } catch (Exception e) {
        System.out.println(e.getMessage());
    }
    return response;
}
```

