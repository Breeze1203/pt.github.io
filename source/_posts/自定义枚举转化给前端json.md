---
title: 自定义枚举转化给前端json
date: 2025-01-16 16:22:47
categories:
 - springboot
 - java
tags: 
 - java
 - springboot
---

##### json转化问题
在开发过程中，我们时常遇到更改json的格式，返回特定的类型，比如日期类型等，之前写到过一片文章处理这类问题；
这几天就遇到一个问题，之前项目中遇到了大量枚举，平常枚举一般是这种
```
public enum Channel {
    app(1),
    H5(1);
    Channel(int code) {
        this.code = code;
    }
    private final int code;

    public int getCode() {
        return code;
    }
    // 通过整数值获取枚举常量
    public static Channel getValue(int value) {
        for (Channel channel : Channel.values()) {
            if (channel.getCode() == value) {
                return channel;
            }
        }
        throw new IllegalArgumentException("Invalid channel value: " + value);
    }
}
```
在springmvc中，前端我们经常接收code 1或2，返回的则是app或h5，但是我们现在后端也需要返回的是数字1和2，这怎么办，我们就只能定义json的序列化和发序列化，类似与下面
```
@JsonComponent
public class Config {

    public static class Serializer extends JsonSerializer<Channel> {


        @Override
        public void serialize(Channel channel, JsonGenerator json, SerializerProvider serializerProvider) throws IOException {
            json.writeNumber(channel.getCode());
        }
    }

    public static class Deserializer extends JsonDeserializer<Channel> {
        @Override
        public Channel deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
            int i = jsonParser.getIntValue();
            return Channel.getValue(i);
        }
    }

    public static class DateSerializer extends JsonSerializer<Date> {
        private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yy-MM-dd 00:00:00");

        @Override
        public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
            String formattedDate = dateFormat.format(date);
            jsonGenerator.writeString(formattedDate);
        }
    }
    public static class DateDeserializer extends JsonDeserializer<Date> {
        private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yy-MM-dd 00:00:00");

        @Override
        public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
            String dateStr = jsonParser.getText().trim();
            try {
                return dateFormat.parse(dateStr);
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```
注意，不同springboot版本有不同处理方案，建议阅读官方文档；