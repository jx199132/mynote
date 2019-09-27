# elasticsearch提供的客户端
https://www.elastic.co/guide/en/elasticsearch/client/index.html

官网提供了以下客户端方式连接elasticsearch，进行通信

- Java REST Client [6.5] — other versions
- Java API [6.5] — other versions
- JavaScript API
- Groovy API [2.4] — other versions
- .NET API [6.x] — other versions
- PHP API [6.0] — other versions
- Perl API
- Python API
- Ruby API
- Community Contributed Clients


# Java REST Client介绍

ES提供了两个JAVA REST client 版本


```
Java Low Level REST Client: 低级别的REST客户端，通过http与集群交互，用户需自己编组请求JSON串，及解析响应JSON串。兼容所有ES版本
```

```
Java High Level REST Client: 高级别的REST客户端，基于低级别的REST客户端，增加了编组请求JSON串、解析响应JSON串等相关api。使用的版本需要保持和ES服务端的版本一致，否则会有版本问题
```

官网建议使用高级别版本客户端，如果高级别客户端不支持的特殊功能，使用低级别客户端进行补充！


# 低级别客户端使用
pom.xml

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>6.5.1</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20180813</version>
</dependency>
```

EsClientUtil.java

```
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;

import java.io.IOException;

/**
 * @author qiyang
 * @email q348002671@qq.com
 */
public class EsClientUtil {
    private static RestClient restClient = null;

    static {
        restClient = RestClient.builder(
                new HttpHost("192.168.204.129", 9200, "http")
        ).build();
    }

    /**
     * 返回ES RestClient
     * @return restClient
     */
    public static RestClient getRestClient(){
        return restClient;
    }

    /**
     * 关闭客户端连接
     */
    public static void closeClient(){
        try {
            restClient.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

测试类 Test_ES_Low_Level_REST_Client

```
import org.apache.http.Header;
import org.apache.http.HttpEntity;
import org.apache.http.HttpHost;
import org.apache.http.RequestLine;
import org.apache.http.entity.ContentType;
import org.apache.http.nio.entity.NStringEntity;
import org.apache.http.util.EntityUtils;
import org.elasticsearch.client.Request;
import org.elasticsearch.client.Response;
import org.elasticsearch.client.RestClient;
import org.json.JSONArray;
import org.json.JSONObject;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import util.EsClientUtil;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * @author qiyang
 * @email q348002671@qq.com
 * ES客户端操作示例demo
 */
public class Test_ES_Low_Level_REST_Client {

    private static RestClient restClient;

    @Before
    public void before(){
        restClient = EsClientUtil.getRestClient();
    }

    @After
    public void end(){
        EsClientUtil.closeClient();
    }

    @Test
    public void test_connection(){
        try {
            Request request = new Request("GET", "/");
            request.addParameter("pretty", "true");
            Response response = restClient.performRequest(request);
            RequestLine requestLine = response.getRequestLine();
            HttpHost host = response.getHost();
            int statusCode = response.getStatusLine().getStatusCode();
            Header[] headers = response.getHeaders();
            String responseBody = EntityUtils.toString(response.getEntity());
            System.out.println(responseBody);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void test_createIndex(){
        try {
            Request request = new Request("PUT", "/index2");
            Response response = restClient.performRequest(request);
            RequestLine requestLine = response.getRequestLine();
            HttpHost host = response.getHost();
            int statusCode = response.getStatusLine().getStatusCode();
            Header[] headers = response.getHeaders();
            String responseBody = EntityUtils.toString(response.getEntity());
            System.out.println(responseBody);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void test_deleteIndex(){
        try {
            Request request = new Request("DELETE", "/index2");
            Response response = restClient.performRequest(request);
            RequestLine requestLine = response.getRequestLine();
            HttpHost host = response.getHost();
            int statusCode = response.getStatusLine().getStatusCode();
            Header[] headers = response.getHeaders();
            String responseBody = EntityUtils.toString(response.getEntity());
            System.out.println(responseBody);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void addDoc(){
        try {
            Request request = new Request("POST", "/index/fulltext/5");
            Map<String ,Object> map = new HashMap<>();
            map.put("content","中国地质大学");
            JSONObject jsonObject = new JSONObject(map);
            HttpEntity entity = new NStringEntity(jsonObject.toString(), ContentType.APPLICATION_JSON);
            request.setEntity(entity);
            Response response = restClient.performRequest(request);
            RequestLine requestLine = response.getRequestLine();
            HttpHost host = response.getHost();
            int statusCode = response.getStatusLine().getStatusCode();
            Header[] headers = response.getHeaders();
            String responseBody = EntityUtils.toString(response.getEntity());
            System.out.println(responseBody);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void test_search(){
        try {
            Request request = new Request("POST", "/index/fulltext/_search");
            JSONObject jsonObject = new JSONObject();
            jsonObject.put("query",
                    new JSONObject().put("match",
                            new JSONObject().put("content","中国")
                    )
            );

            jsonObject.put("highlight",
                    new JSONObject().
                            put("pre_tags",new JSONArray()
                                    .put("<tag1>").put("<tag2>")
                            ).
                            put("post_tags",new JSONArray()
                                    .put("</tag1>").put("</tag2>")
                            ).
                            put("fields",new JSONObject()
                                    .put("content",new JSONObject())
                            )
            );
//        {
//            "query" : { "match" : { "content" : "中国" }},
//            "highlight" : {
//                "pre_tags" : ["<tag1>", "<tag2>"],
//                "post_tags" : ["</tag1>", "</tag2>"],
//                "fields" : {
//                    "content" : {}
//                }
//            }
//        }

            request.setJsonEntity(jsonObject.toString());
            Response response = restClient.performRequest(request);
            RequestLine requestLine = response.getRequestLine();
            HttpHost host = response.getHost();
            int statusCode = response.getStatusLine().getStatusCode();
            Header[] headers = response.getHeaders();
            String responseBody = EntityUtils.toString(response.getEntity());
            System.out.println(responseBody);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```
