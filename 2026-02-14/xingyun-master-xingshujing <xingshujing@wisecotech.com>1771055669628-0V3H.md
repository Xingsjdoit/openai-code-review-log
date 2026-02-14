# 小傅哥项目： OpenAi 代码评审.
### 😀代码评分：85
#### 😀代码逻辑与目的：
该代码段的主要目的是通过比较两个JSON字符串的差异，并使用Jackson和Guava库将差异结果转换为自定义的JSON结构，然后序列化为JSON字符串并打印出来。

#### 🤔问题点：
1. **代码结构**：代码中存在大量未使用的空白行和空方法体，这会影响代码的可读性和可维护性。
2. **注释**：代码中缺少必要的注释，对于非团队成员来说难以理解代码的作用。
3. **异常处理**：代码中没有对`readValue`方法可能抛出的`IOException`进行异常处理。
4. **资源分配**：`ObjectMapper`实例的创建和销毁没有考虑到资源管理的最佳实践。

#### 🎯修改建议：
1. **删除未使用的代码**：移除所有未使用的空白行和空方法体。
2. **添加注释**：在代码中添加必要的注释，解释关键代码段的作用。
3. **异常处理**：添加对`readValue`方法的异常处理。
4. **资源管理**：使用`try-with-resources`语句确保`ObjectMapper`实例在完成后被正确关闭。

#### 💻修改后的代码：
```java
package com.xingyun.json.flat;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.common.collect.MapDifference;
import com.google.common.collect.Maps;

import java.util.HashMap;
import java.util.Map;
import java.util.stream.Collectors;

/**
 * @Author JingWei
 * @create 2022/2/28
 */
public class DiffTest {

    public void testJsonDiffByFlatten() {
        String str1 = "{\"name\":\"json1\",\"arr\":[{\"id\":\"2\",\"name\":\"name2\",\"age\":\"age2\"},{\"id\":\"1\",\"name\":\"name1\",\"age\":\"age1\"}],\"obj\":{\"obj_name\":\"obj_name1\",\"obj_label\":\"obj_label1\"}}";
        String str2_add = "{\"name\":\"json1\",\"arr\":[{\"id\":\"2\",\"name\":\"name2\",\"age\":\"age2\"},{\"id\":\"1\",\"name\":\"name1\",\"age\":\"age1\"}],\"obj\":{\"obj_name\":\"obj_name1\",\"obj_label\":\"obj_label1\"},\"name_add\":\"name_add\",\"arr_add\":[{\"id\":\"2\",\"name\":\"name2\",\"age\":\"age2\"},{\"id\":\"1\",\"name\":\"name1\",\"age\":\"age1\"}]}";
        ObjectMapper mapper = new ObjectMapper();

        try {
            TypeReference<HashMap<String, Object>> type = new TypeReference<HashMap<String, Object>>() {};
            HashMap<String, Object> j1 = mapper.readValue(str1, type);
            HashMap<String, Object> j2 = mapper.readValue(str2_add, type);

            Map<String, Object> flatten1 = FlatMapUtil.flatten(j1);
            Map<String, Object> flatten2 = FlatMapUtil.flatten(j2);
            MapDifference<String, Object> difference = Maps.difference(flatten1, flatten2);

            Map<String, Object> diffResult = new HashMap<>();
            diffResult.put("entriesOnlyOnLeft", FlatMapUtil.unflatten(difference.entriesOnlyOnLeft()));
            diffResult.put("entriesOnlyOnRight", FlatMapUtil.unflatten(difference.entriesOnlyOnRight()));
            diffResult.put("entriesDiffering", difference.entriesDiffering().entrySet().stream()
                    .collect(Collectors.toMap(
                            Map.Entry::getKey,
                            e -> new HashMap<String, Object>() {{
                                put("leftValue", e.getValue().leftValue());
                                put("rightValue", e.getValue().rightValue());
                            }})));

            String jsonDiffResult = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(diffResult);
            System.out.println(jsonDiffResult);
            System.out.println(difference);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            mapper.close();
        }
    }
}
```

#### 🌟代码中的优点：
- 使用了Jackson和Guava库来处理JSON数据，这些库都是成熟的、广泛使用的库，可以确保代码的稳定性和性能。
- 使用了流式API来处理差异结果，这使得代码更加简洁和易于理解。