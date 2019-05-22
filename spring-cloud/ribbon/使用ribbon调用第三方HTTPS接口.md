# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里讲述一下如何通过ribbon中的RestTemplate实现对第三方https接口的请求。话不多说,直接上代码。

# 实现

## 配置文件

```yml
ribbon-client-config:
  ConnectTimeout: 3000
  ReadTimeout:  3000
```

## 加入配置

```java

package com.didapinche.thrift.cmstaxi.config;

import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.config.Registry;
import org.apache.http.config.RegistryBuilder;
import org.apache.http.conn.socket.ConnectionSocketFactory;
import org.apache.http.conn.socket.PlainConnectionSocketFactory;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

    @Value("${ribbon-client-config.ConnectTimeout}")
    private int connectTimeout;

    @Value("${ribbon-client-config.ReadTimeout}")
    private int readTimeout;


    @Bean
    public RestTemplate restTemplate()
    {
        ClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient());
        return new RestTemplate(requestFactory);
    }

    private HttpClient httpClient()
    {
        // 支持HTTP、HTTPS
        Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory> create()
                .register("http", PlainConnectionSocketFactory.getSocketFactory())
                .register("https", SSLConnectionSocketFactory.getSocketFactory())
                .build();
        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager(registry);
        connectionManager.setMaxTotal(200);
        connectionManager.setDefaultMaxPerRoute(100);
        connectionManager.setValidateAfterInactivity(2000);
        RequestConfig requestConfig = RequestConfig.custom()
                // 服务器返回数据(response)的时间，超时抛出read timeout
                .setSocketTimeout(readTimeout)
                // 连接上服务器(握手成功)的时间，超时抛出connect timeout
                .setConnectTimeout(connectTimeout)
                .build();
        return HttpClientBuilder.create().setDefaultRequestConfig(requestConfig).setConnectionManager(connectionManager).build();
    }
}

```

## util工具类

```java
package com.didapinche.thrift.cmstaxi.util;

import com.didapinche.server.commons.common.json.JsonMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.web.client.RestTemplate;

public class RestTemplateUtil {

    private static Logger logger = LoggerFactory.getLogger(RestTemplateUtil.class);

    private static RestTemplate restTemplate = SpringUtils.getBean("restTemplate",RestTemplate.class);

    public static <T>T  postForObject (String url, Object object, Class<T> tClass) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
        HttpEntity<Object> requestEntity = new HttpEntity<>(object,headers);
        T post = restTemplate.postForObject(url, object, tClass);
        logger.info("RestTemplateUtil postForObject url:{},params:{},result:{}",url,JsonMapper.toJson(object),
                JsonMapper.toJson(post));
        return post;
    }
}

```

## 测试类

```java

    @Test
    public void testHttps()
    {
        String url  = "https://www.so.com/";
        String responseBody;

        responseBody = restTemplate.getForObject(url, String.class);
        logger.info("responseBody={}", responseBody);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
        HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<>(null,headers);
        responseBody = restTemplate.postForObject(url, requestEntity, String.class);
        logger.info("responseBody={}", responseBody);
    }

```