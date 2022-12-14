# 定义注解
```java
	@Target({ElementType.METHOD,ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @RequestMapping
    public @interface ApiVersion {

        @AliasFor(annotation = RequestMapping.class)

        String name() default "";

        @AliasFor(annotation = RequestMapping.class)
        String[] value() default {};

        @AliasFor(annotation = RequestMapping.class)
        String[] path() default {};
        @AliasFor(annotation = RequestMapping.class)

        RequestMethod[] method() default {};

        @AliasFor(annotation = RequestMapping.class)
        String[] params() default {};

        @AliasFor(annotation = RequestMapping.class)
        String[] headers() default {};

        @AliasFor(annotation = RequestMapping.class)
        String[] consumes() default {};

        @AliasFor(annotation = RequestMapping.class)
        String[] produces() default {};

        //版本号字段
        String version() default "1.0.0"; ;
    }
```
 
# 实现自定义RequestCondition 
```java
@AllArgsConstructor
    @Getter
    @Slf4j
    public class ApiVersionCondition implements RequestCondition<ApiVersionCondition> {

        private String version;

        private final static Pattern VERSION_PREFIX_PATTERN = Pattern.compile(".*v(\\d+(.\\d+){0,2}).*");

        public final static String API_VERSION_CONDITION_NULL_KEY = "API_VERSION_CONDITION_NULL_KEY";

        @Override
        public ApiVersionCondition combine(ApiVersionCondition condition) {
            // 方法上的注解优于类上的注解
            return new ApiVersionCondition(condition.getVersion());
        }

        @Override
        public ApiVersionCondition getMatchingCondition(HttpServletRequest request) {
            Matcher m = VERSION_PREFIX_PATTERN.matcher(request.getRequestURI());
            if (m.find()) {
                return this;
            }
            // 将错误放在request中，可以在错误页面明确提示，此处可重构为抛出运行时异常
            request.setAttribute(API_VERSION_CONDITION_NULL_KEY, true);
            return null;
        }

        @Override
        public int compareTo(ApiVersionCondition other, HttpServletRequest request) {
            return this.compareTo(other.getVersion()) ? 1 : -1;
        }

        private boolean compareTo(String version) {
            return Objects.equals(version, this.version);
        }
    }
```
 
# 配置HandlerMapping 规则
```java
public class ApiRequestMappingHandlerMapping extends RequestMappingHandlerMapping {

    @Override
    protected RequestCondition<?> getCustomTypeCondition(Class<?> handlerType) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class);
        return createCondition(apiVersion);
    }

    @Override
    protected RequestCondition<?> getCustomMethodCondition(Method method) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
        return createCondition(apiVersion);
    }

    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo requestMappingInfo = super.getMappingForMethod(method, handlerType);
        if (requestMappingInfo == null) {
            return null;
        }
        return createCustomRequestMappingInfo(method, handlerType, requestMappingInfo);
    }

    private RequestMappingInfo createCustomRequestMappingInfo(Method method, Class<?> handlerType, RequestMappingInfo requestMappingInfo) {
        ApiVersion methodApi = AnnotatedElementUtils.findMergedAnnotation(method, ApiVersion.class);
        ApiVersion handlerApi = AnnotatedElementUtils.findMergedAnnotation(handlerType, ApiVersion.class);
        if (methodApi != null || handlerApi != null) {
            String version = methodApi == null ? handlerApi.version() : methodApi.version();
            return RequestMappingInfo.paths("v" + version).options(this.config).build().combine(requestMappingInfo);
        }
        return requestMappingInfo;
    }


    private RequestMappingInfo.BuilderConfiguration config = new RequestMappingInfo.BuilderConfiguration();


    private RequestCondition<ApiVersionCondition> createCondition(ApiVersion apiVersion) {
        return apiVersion == null ? null : new ApiVersionCondition(apiVersion.version());
    }

}
```
 
# 注册覆盖HandlerMapping 
```java
@Component
    public class WebMvcRegistrationsConfig implements WebMvcRegistrations {
        @Override
        public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
            RequestMappingHandlerMapping handlerMapping = new ApiRequestMappingHandlerMapping();
            handlerMapping.setOrder(0);
            return handlerMapping;
        }
    }
```
 
