---
title: Java根据模板生成PDF文件
date: 2018/5/19 11:29
tags: itext5
category:  blog
---

### 需求场景
    根据业务自定义模板，按照模板导出相应格式数据，包括文字、图片等
 
### 解决方案
    1. Adobe Acrobаt编辑PDF模板
    2. itextpdf5 填入业务参数
### 依赖
    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itextpdf</artifactId>
        <version>5.5.13</version>
    </dependency>
    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itext-asian</artifactId>
        <version>5.2.0</version>
    </dependency>
### 参考资料

[官网例子](https://developers.itextpdf.com/examples-itext5)

### 过程  

#### 制作PDF模板
这个过程，没啥技术含量，只讲下经验：
1. 先用Word编辑模板，然后导出成PDF，比直接编辑PDF方便
2. Acrobаt打开PDF，准备表单，需要注意
    - 字体一定要选带中文的，比如宋体、雅黑等
    - 属性一定选可见
    - 表单域名 一定要和业务参数名对应
    
#### 根据模板生成文件
用了PdfReader、PdfStamper和AcroFields，实现文本域和图像域内容的写入。
params参数集的key必须与表单域名一一对应，其他的直接看代码：
````java
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Map;

import com.itextpdf.text.DocumentException;
import com.itextpdf.text.Image;
import com.itextpdf.text.pdf.AcroFields;
import com.itextpdf.text.pdf.PdfReader;
import com.itextpdf.text.pdf.PdfStamper;
import com.itextpdf.text.pdf.PushbuttonField;

public class PdfUtil {

    /**
     *
     * @param src PDF模板路径
     * @param dest 生成文件路径
     * @param params 参数集。若为图片，key中需包含image，value为图片地址
     * @throws DocumentException
     * @throws IOException
     */
    public static void generatePDF(String src, String dest, Map<String,String> params) throws DocumentException, IOException {

        File file = new File(dest);
        file.getParentFile().mkdirs();
        PdfReader reader =null;
        PdfStamper stamper = null;
        try{
            reader= new PdfReader(src);
            stamper = new PdfStamper(reader, new FileOutputStream(dest));
            AcroFields form = stamper.getAcroFields();
            for (String key :form.getFields().keySet()){
                if (params.get(key)!=null){
                    //根据表单域名区分类型
                    if (!key.startsWith("Image")){
                        //设置文本域参数值
                        form.setField(key,params.get(key));
                    }else{
                        //将图片写入指定的field
                        Image image = Image.getInstance(params.get(key));
                        PushbuttonField pb = form.getNewPushbuttonFromField(key);
                        pb.setImage(image);
                        form.replacePushbuttonField(key, pb.getField());
                    }
                }
            }
            stamper.setFormFlattening(true);
        }finally {
            try{
                if (stamper!=null){
                    stamper.close();
                }
            }catch (Exception e){
            }
            try{
                if (reader!=null){
                    reader.close();
                }
            }catch (Exception e){
            }
        }
    }
}
````
### 结尾
1. IText类库的功能非常强大，支持各种控件的操作，包括表单、表格、字体、图片等，还具有文件合并、转格式、加密、签名等功能。
可惜的是中文的资料不多，如果需要深入使用，建议直接去官网上看API文档和Examples示例。
2. 另外，由于是国外的产品，对中文的处理有一些坑在官网上并未提及，需要想办法解决，比如前面依赖的itext-asian包和模板编辑时字体的选择 就属此类。