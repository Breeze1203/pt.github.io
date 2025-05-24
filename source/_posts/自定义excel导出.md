---
title: "自定义Excel导出"
date: 2024-11-18 15:41:52
---

## 自定义导出Excel

需求背景：对象集合，某些字段无需添加到excel表格中，我们只需要给实体类字段添加特定注解实现，尽可能充满灵活性；

具体代码实现：

<img src="/upload/截屏2024-04-04%2023.25.41.png" style="display: inline-block;width:100.0%;height:100.0%" /><img src="/upload/截屏2024-04-04%2023.25.35.png" style="display: inline-block;width:100.0%;height:100.0%" />第一个注解的作用是更好实现Excel表格第一行属性

实体类

<img src="/upload/截屏2024-04-04%2023.27.19.png" style="display: inline-block;width:100.0%;height:100.0%" />具体实现代码

``` java
package org.pt.excelutil;

import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.pt.annotation.ExcludeFromExcel;
import org.pt.annotation.FiledName;


import java.io.FileOutputStream;
import java.io.IOException;
import java.lang.reflect.Field;
import java.util.List;

/*
Excel导出
 */
public class ExcelExport {
    public static void writeToExcel(List<?> object,String path) {
        HSSFWorkbook workbook = new HSSFWorkbook();
        HSSFSheet sheet = workbook.createSheet();
        // 第一行写入自定义的属性名
        HSSFRow headerRow = sheet.createRow(0);
        Field[] fields = object.get(0).getClass().getDeclaredFields();
        // 遍历并访问每个字段
        for (int i = 0; i < fields.length; i++) {
            Field field = fields[i];
            field.setAccessible(true);
            if (field.isAnnotationPresent(FiledName.class) && !field.isAnnotationPresent(ExcludeFromExcel.class)) {
                FiledName annotation = field.getAnnotation(FiledName.class);
                Cell cell = headerRow.createCell(i);
                cell.setCellValue(annotation.value());
            }
        }
        // 从第二行开始写入属性值
        for (int rowNum = 1; rowNum <= object.size(); rowNum++) {
            HSSFRow row = sheet.createRow(rowNum);
            for (int i = 0; i < fields.length; i++) {
                Field field = fields[i];
                field.setAccessible(true);
                // 判断这个字段是否有注解
                if (field.isAnnotationPresent(FiledName.class) && !field.isAnnotationPresent(ExcludeFromExcel.class)) {
                    Cell cell = row.createCell(i);
                    Object value = null;
                    try {
                        value = field.get(object.get(rowNum - 1));
                        if (value != null) {
                            cell.setCellValue(value.toString());
                        } else {
                            cell.setCellValue(""); // 或者设置为其他默认值
                        }
                    } catch (IllegalAccessException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        }

        try (FileOutputStream fileOut = new FileOutputStream(path)) {
            workbook.write(fileOut);
            System.out.println("Excel file written successfully.");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

    }
}
```

效果图

<img src="/upload/截屏2024-04-04%2023.28.19.png" style="display: inline-block;width:100.0%;height:100.0%" />
