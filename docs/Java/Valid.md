<a name="eKhI5"></a>
# 注解定义
```java
	@Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.PARAMETER, ElementType.FIELD})
    @Constraint(validatedBy = CustomValidatorClass.class)
    @Documented
    public @interface CustomValidation{

        String message() default "自定义提示信息";

        Class<?>[] groups() default { };

        Class<? extends Payload>[] payload() default { };
    }
```
 
<a name="UJJAQ"></a>
# 规则定义
```java
public class CustomValidatorClass implements ConstraintValidator<CustomValidation,Object> {

    private CustomValidation validation;

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext context) {
        //自定义校验逻辑
        return false;
    }

    @Override
    public void initialize(CustomValidation customValidation) {
        this.validation = customValidation;
    }
}
```

# 使用方法
Pom添加依赖
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**controller添加注解@Validated**

**需要校验的参数添加自定义注解@CustomValidation**

