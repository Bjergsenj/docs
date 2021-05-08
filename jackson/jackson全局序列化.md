### spring boot使用jackson全局序列化配置

- 此序列化配置针对于全局
  - 如果想全局对某个类型做一些特殊处理可配置Jackson2ObjectMapperBuilderCustomizer该bean来实现

```java
@Configuration
public class GlobalConfig {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer enumCustomizer() {
        // 全局设置BigDecimal的序列化与反序列化
        return jacksonObjectMapperBuilder -> jacksonObjectMapperBuilder.serializerByType(BigDecimal.class, this.jsonSerializer()).deserializerByType(BigDecimal.class, this.jsonDeserialize());
    }

    private JsonSerializer<BigDecimal> jsonSerializer() {
        return new JsonSerializer<BigDecimal>() {
            @Override
            public void serialize(BigDecimal value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
                // BigDecimal类型时返回前端为0,可根据业务调整
                gen.writeNumber(new BigDecimal(0));
            }
        };
    }

    private JsonDeserializer<BigDecimal> jsonDeserialize() {
        return new JsonDeserializer<BigDecimal>() {
            @Override
            public BigDecimal deserialize(JsonParser p, DeserializationContext ctx) throws IOException {
                // 入参model中BigDecimal类型的参数会做保留两位小数四舍五入操作
                return p.getDecimalValue() == null ? new BigDecimal(0) : p.getDecimalValue().setScale(2, RoundingMode.HALF_UP);
            }
        };
    }

}
```

