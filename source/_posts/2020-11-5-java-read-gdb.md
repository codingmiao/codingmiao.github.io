---
layout: post
title:  "java读取arcgis的gdb文件"
categories:
- gis
- 服务端
tags:  arcgis
author: liuyu
date: 2020/11/5 20:46:25
---


gdb,是arcgis重要的数据格式之一，它把矢量数据按以下层级结构存为本地文件
```
图层
-- 要素
---- 属性
---- 形状
---- 样式
```

gdal提供了函数允许我们读取gdb，因此，我们可以通过jni调用gdal来再java中读取gdal

# 1、配置gdal环境

liunx下按照gdal需要编译，而windows下就比较简单了，下载编译好的dll文件，然后环境变量path里加入dll目录即可
```
链接：https://pan.baidu.com/s/1kH3djGQsow-llQiy82068g 
提取码：uuwn 
```

# 2、建一个maven工程
前面下载的那个包里的lib文件夹拷到工程中，然后maven里添加gdal的依赖:
```
    <dependencies>
        <dependency>
            <groupId>org.gdal</groupId>
            <artifactId>gdal</artifactId>
            <version>2.2.3</version>
            <scope>system</scope>
            <!--<systemPath>${project.basedir}/lib/gdal-centos7x64.jar</systemPath>-->
            <systemPath>${project.basedir}/lib/gdal-win64.jar</systemPath>
        </dependency>
    </dependencies>
```

# 3、写代码
```java
package com;

import org.gdal.gdal.gdal;
import org.gdal.ogr.*;

import java.sql.Date;
import java.sql.Time;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;

/**
 * gdal读取gdb示例
 * @author liuyu
 * @date 2020/11/5
 */
public class Test {
    static {
        gdal.AllRegister();
    }

    public static void main(String[] args) {
        Driver driver = ogr.GetDriverByName("OpenFileGDB");
        DataSource dataSource = driver.Open("D:\\data\\云南省矢量地图gdb\\云南省地图综合成果.gdb", 0);
        for (int i = 0; i < dataSource.GetLayerCount(); i++) {
            //获取图层
            Layer layer = dataSource.GetLayer(i);
            System.out.println(i + "\t" + layer.GetName());
            do {//获取图层下的要素
                Feature feature = layer.GetNextFeature();
                if (null == feature) {
                    break;
                }
                //获取样式，这里是空，待确定
                System.out.println(feature.GetStyleString());
                //获取geometry
                System.out.println(feature.GetGeometryRef().ExportToWkt());//也可以ExportToWkb() gml等等
                //获取属性
                for (int p = 0; p < feature.GetFieldCount(); p++) {
                    System.out.println(feature.GetFieldDefnRef(p).GetName()
                            + "\t" +
                            getProperty(feature, p));
                }
                System.out.println("---------");
            } while (true);
        }
    }

    private static Object getProperty(Feature feature, int index) {
        int type = feature.GetFieldType(index);
        PropertyGetter propertyGetter;
        if (type < 0 || type >= propertyGetters.length) {
            propertyGetter = stringPropertyGetter;
        } else {
            propertyGetter = propertyGetters[type];
        }
        try {
            return propertyGetter.get(feature, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 属性获取器
     */
    @FunctionalInterface
    private interface PropertyGetter {
        Object get(Feature feature, int index);
    }

    private static final PropertyGetter stringPropertyGetter = (feature, index) -> feature.GetFieldAsString(index);

    /**
     * feature.GetFieldType(index)得到一个属性类型的int值,该值对应具体类型
     */
    private static final PropertyGetter[] propertyGetters = new PropertyGetter[]{
            (feature, index) -> feature.GetFieldAsInteger(index),//0	Integer
            (feature, index) -> feature.GetFieldAsIntegerList(index),//1	IntegerList
            (feature, index) -> feature.GetFieldAsDouble(index),//2	Real
            (feature, index) -> feature.GetFieldAsDoubleList(index),//3	RealList
            stringPropertyGetter,//4	String
            (feature, index) -> feature.GetFieldAsStringList(index),//5	StringList
            stringPropertyGetter,//6	(unknown)
            stringPropertyGetter,//7	(unknown)
            (feature, index) -> feature.GetFieldAsBinary(index),//8	Binary
            (feature, index) -> {
                int[] pnYear = new int[1];
                int[] pnMonth = new int[1];
                int[] pnDay = new int[1];
                int[] pnHour = new int[1];
                int[] pnMinute = new int[1];
                float[] pfSecond = new float[1];
                int[] pnTZFlag = new int[1];
                feature.GetFieldAsDateTime(index, pnYear, pnMonth, pnDay, pnHour, pnMinute, pfSecond, pnTZFlag);
                Date date = Date.valueOf(LocalDate.of(pnYear[0], pnMonth[0], pnDay[0]));
                return date;
            },//9	Date
            (feature, index) -> {
                int[] pnYear = new int[1];
                int[] pnMonth = new int[1];
                int[] pnDay = new int[1];
                int[] pnHour = new int[1];
                int[] pnMinute = new int[1];
                float[] pfSecond = new float[1];
                int[] pnTZFlag = new int[1];
                feature.GetFieldAsDateTime(index, pnYear, pnMonth, pnDay, pnHour, pnMinute, pfSecond, pnTZFlag);
                float fSecond = pfSecond[0];
                int s = (int) fSecond;
                int ns = (int) (1000000000 * fSecond - s);
                Time time = Time.valueOf(LocalTime.of(pnHour[0], pnMinute[0], s, ns));
                return time;
            },// 10	Time
            (feature, index) -> {
                int[] pnYear = new int[1];
                int[] pnMonth = new int[1];
                int[] pnDay = new int[1];
                int[] pnHour = new int[1];
                int[] pnMinute = new int[1];
                float[] pfSecond = new float[1];
                int[] pnTZFlag = new int[1];
                feature.GetFieldAsDateTime(index, pnYear, pnMonth, pnDay, pnHour, pnMinute, pfSecond, pnTZFlag);
                float fSecond = pfSecond[0];
                int s = (int) fSecond;
                int ns = (int) (1000000000 * fSecond - s);
                LocalDateTime localDateTime = LocalDateTime.of(
                        LocalDate.of(pnYear[0], pnMonth[0], pnDay[0]),
                        LocalTime.of(pnHour[0], pnMinute[0], s, ns)
                );
                Timestamp timestamp = Timestamp.valueOf(localDateTime);
                return timestamp;
            },//11	DateTime
            (feature, index) -> feature.GetFieldAsInteger64(index),//12	Integer64
            (feature, index) -> feature.GetFieldAsIntegerList(index),//13 Integer64List
            //>=14	(unknown)
    };

}

```
