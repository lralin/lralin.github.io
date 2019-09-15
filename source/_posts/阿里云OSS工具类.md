---
title: 阿里云OSS工具类
date: 2019-09-15 17:57:25
updated: 2019-09-15 19:57:25
tags: 
- java
- 工具类

---

##### Spring Boot工具类实现

```java
package com.xgdfin.aliyunoss;

import com.aliyun.oss.ClientException;
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.aliyun.oss.OSSException;
import com.aliyun.oss.model.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

@Component
public class AliyunOSSUtil {

    private static final Logger logger = LoggerFactory.getLogger(AliyunOSSUtil.class);

    @Value("${aliyun.file.endpoint}")
    private String endpoint;
    @Value("${aliyun.file.accessKeyId}")
    private String accessKeyId;
    @Value("${aliyun.file.accessKeySecret}")
    private String accessKeySecret;
    @Value("${aliyun.file.folder}")
    private String folder;
    @Value("${aliyun.file.bucketName}")
    private String bucketName;
    @Value("${aliyun.file.webUrl}")
    private String webUrl;

    /**
     * 上传文件
     */
    public FileDTO upLoad(File file) {
        logger.info("------OSS文件上传开始--------" + file.getName());
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        String dateStr = format.format(new Date());
        String uuid = UUID.randomUUID().toString().replace("-", "");
        String suffix = file.getName().substring(file.getName().lastIndexOf(".") + 1);
        // 判断文件
        if (file == null) {
            return null;
        }

        OSS client = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);

        try {
            // 判断容器是否存在,不存在就创建
            if (!client.doesBucketExist(bucketName)) {
                client.createBucket(bucketName);
                CreateBucketRequest createBucketRequest = new CreateBucketRequest(bucketName);
                //TODO 是否需要公共可读权限待确认
                createBucketRequest.setCannedACL(CannedAccessControlList.PublicRead);
                client.createBucket(createBucketRequest);
            }
            // 设置文件路径和名称
            String fileUrl = folder + "/" + (dateStr + "/" + uuid) + "-" + file.getName();
            // 设置权限(公开读) //TODO 是否需要公共可读权限待确认
            client.setBucketAcl(bucketName, CannedAccessControlList.PublicRead);
            // 上传文件
            PutObjectResult result = client.putObject(new PutObjectRequest(bucketName, fileUrl, file));
            if (result != null) {
                logger.info("返回结果：{}", result);
                logger.info("------OSS文件上传成功------{}", fileUrl);
                return new FileDTO(
                        file.length(),//文件大小
                        fileUrl,//文件的绝对路径
                        webUrl + "/" + fileUrl,//文件的web访问地址
                        suffix,//文件后缀
                        bucketName,//存储的bucket
                        file.getName(),//原文件名
                        folder,//存储的文件夹
                        result.getRequestId()
                );

            }
        } catch (OSSException oe) {
            logger.error(oe.getMessage());
        } catch (ClientException ce) {
            logger.error(ce.getErrorMessage());
        } finally {
            if (client != null) {
                client.shutdown();
            }
        }
        return null;
    }

    /**
     * 下载文件
     */
    public String download(String fileUrl) {
        logger.info("------OSS文件下载开始--------");
        OSS client = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
        try {
            // 判断容器是否存在,不存在就创建
            if (!client.doesBucketExist(bucketName)) {
                logger.info("容器bucket不存在");
                return null;
            }
            // 下载文件
            OSSObject object = client.getObject(bucketName, fileUrl);
            if (object != null) {
                logger.info("返回结果：{}", object);
                logger.info("------OSS文件下载获取成功------{}", fileUrl);
                //TODO
//                return object;
            }
        } catch (OSSException oe) {
            logger.error(oe.getMessage());
        } catch (ClientException ce) {
            logger.error(ce.getErrorMessage());
        } finally {
            if (client != null) {
                client.shutdown();
            }
        }
        return null;
    }

    /**
     * 下载文件
     */
    public void deleteFile(String fileUrl) {
        logger.info("------OSS文件删除开始--------");
        OSS client = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
        try {
            // 判断容器是否存在,不存在就创建
            if (!client.doesBucketExist(bucketName)) {
                logger.info("容器bucket不存在");
            }
            // 下载文件
            client.deleteObject(bucketName, fileUrl);
            logger.info("------OSS文件删除成功------{}", fileUrl);
        } catch (OSSException oe) {
            logger.error(oe.getMessage());
        } catch (ClientException ce) {
            logger.error(ce.getErrorMessage());
        } finally {
            if (client != null) {
                client.shutdown();
            }
        }
    }
}

```

### application.yml配置文件

```java
aliyun:
  file:
    endpoint: http://xxxx.xxx.com
    accessKeyId: xxxx
    accessKeySecret: xxx
    bucketName: xx
    folder: xxxx
    webUrl: http://xxxx.xxx.com
```

Maven pom.xml文件引用的jar包

```xml
 				<!-- aliyun begin  -->
        <dependency>
            <groupId>com.aliyun.oss</groupId>
            <artifactId>aliyun-sdk-oss</artifactId>
            <version>3.6.0</version>
        </dependency>

        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>
        <!-- aliyun end  -->
```

