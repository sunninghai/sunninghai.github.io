---
layout: post
title: "工作问题总结－01"
date: 2016-05-26 17:36:44 +0800
comments: true
categories: 工作问题总结
---
# service层参数或者返回值验证

需要在service类上添加注解@Validated,会抛出ConstraintViolationException异常
# restClient 返回实体类List

默认返回的是List<Map>,需要从Map转化成实体类

```
public List listExamByClassId(String classId, String loginUserId,String tenantId) 
{
        return getList("/exam/class/{classId}?loginUserId={userId}&tenantId={tenantId}", new ParameterizedTypeReference<List<Exam>>() {
        },classId,loginUserId,tenantId);
    }
```

# BigDecimal精度问题
restTemplate 返回数据反序列化时，默认情况下BigDecimal 直接转换成Double,需要自定义实现ObjectMapper


```
<bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
		<constructor-arg ref="clientHttpRequestFactory"/>
		<property name="errorHandler" ref="responseErrorHandler">
		</property>
		<property name="messageConverters">
			<list>
				<bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
					<constructor-arg ref="objectMapper"/>
				</bean>
				<bean class="org.springframework.http.converter.StringHttpMessageConverter">
				<property name="supportedMediaTypes">
						<list>
							<value>text/plain;charset=UTF-8</value>
						</list>
					</property>



				</bean>
			</list>
		</property>
        <property name="interceptors">
            <bean class="com.gxb.sites.client.sys.UserAgentHeaderHttpInterceptor">
            </bean>
        </property>
	</bean>

public class MyObjectMapper extends ObjectMapper {
    private static final long serialVersionUID = -780913908269501532L;

    public MyObjectMapper(){
        super();


        enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS);
        //config to ignore unknown property
    }
}
```


