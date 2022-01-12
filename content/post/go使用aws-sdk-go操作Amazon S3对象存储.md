---
title: "go使用aws-sdk-go操作Amazon S3对象存储"
date: 2022-01-12T11:54:41+08:00
lastmod: 2022-01-12T11:54:41+08:00
draft: false
keywords: ["go","s3","aws"]
description: ""
tags: ["go","s3","aws"]
categories: ["go","s3","aws"]
author: "tureo"
---

# 说明
使用aws-sdk-go操作Amazon S3对象存储，包括文件上传（分段）、文件下载、查看文件列表、查看bucket列表、获取预签名url文件，通过context设置超时时间，文件上传使用分段上传。
示例使用aws-sdk-go的`v1`版本
[v2版本参考官方文档](https://github.com/aws/aws-sdk-go-v2)

# 安装aws-sdk-go
`go get github.com/aws/aws-sdk-go`
[参考](https://github.com/aws/aws-sdk-go)

# 导入包
```Go
package aws

import (
	"bytes"
	"context"
	"fmt"
	"os"

	"time"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/s3"
	"github.com/aws/aws-sdk-go/service/s3/s3manager"
)
```

# 上传文件
使用s3manager的方法进行分段上传
[参考](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#Uploader)
## 读文件上传
直接传入文件路径，方法中读取文件内容上传（分段）到S3。
```Go
func S3PutFileObject(key string, filename string) (location string, err error) {
	var (
		accessKey = "AKIA4AAD"
		secretKey = "eI6adadsaMtQ"
		region    = "eu-west-1"
		bucket    = "my-test"
		prefix    = "dev"
		timeout   = time.Duration(180) * time.Second
	)
	sess := session.Must(session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
		Region:      aws.String(region),
	}))

	options := func(u *s3manager.Uploader) {
		u.PartSize = 50 * 1024 * 1024 // 设置分段上传的PartSize大小为50MB
	}

	uploader := s3manager.NewUploader(sess, options)

	f, err := os.Open(filename)
	if err != nil {
		_ = fmt.Errorf("failed to open file %q, %v", filename, err)
	}
	defer f.Close()

	// 使用context设置超时时间
	ctx := context.Background()
	var cancelFn func()
	if timeout > 0 {
		ctx, cancelFn = context.WithTimeout(ctx, timeout)
	}
	if cancelFn != nil {
		defer cancelFn()
	}

	result, err := uploader.UploadWithContext(ctx, &s3manager.UploadInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(fmt.Sprintf("%s/%s", prefix, key)),
		Body:   f,
	})
	if err != nil {
		_ = fmt.Errorf("failed to upload file, %v", err)
		return
	}
	location = result.Location
	fmt.Printf("file uploaded to, %s\n", aws.StringValue(&location))
	return
}
```

## 字节数组上传
有时候需要直接将字节数组或字符串上传（分段）为S3文件，可通过下面的方法实现，跟上面的[S3PutContentObject](#读文件上传)方法的区别只是入参和文件内容处理的区别。
```Go
func S3PutContentObject(key string, content []byte) (location string, err error) {
	var (
		accessKey = "AKIA4AAD"
		secretKey = "eI6adadsaMtQ"
		region    = "eu-west-1"
		bucket    = "my-test"
		prefix    = "dev"
		timeout   = time.Duration(180) * time.Second
	)
	sess := session.Must(session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
		Region:      aws.String(region),
	}))

	options := func(u *s3manager.Uploader) {
		u.PartSize = 50 * 1024 * 1024 // 设置分段上传的PartSize大小为50MB
	}

	uploader := s3manager.NewUploader(sess, options)

	// 使用context设置超时时间
	ctx := context.Background()
	var cancelFn func()
	if timeout > 0 {
		ctx, cancelFn = context.WithTimeout(ctx, timeout)
	}
	if cancelFn != nil {
		defer cancelFn()
	}

	result, err := uploader.UploadWithContext(ctx, &s3manager.UploadInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(fmt.Sprintf("%s/%s", prefix, key)),
		Body:   bytes.NewReader(content),
	})
	if err != nil {
		_ = fmt.Errorf("failed to upload file, %v", err)
		return
	}
	location = result.Location
	fmt.Printf("file uploaded to, %s\n", aws.StringValue(&location))
	return
}
```

# 下载文件
[参考](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#Downloader)
```Go
func S3GetObject(key string) (content []byte, err error) {
	var (
		accessKey = "AKIA4AAD"
		secretKey = "eI6adadsaMtQ"
		region    = "eu-west-1"
		bucket    = "my-test"
		prefix    = "dev"
		timeout   = time.Duration(180) * time.Second
	)

	sess := session.Must(session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
		Region:      aws.String(region),
	}))

	downloader := s3manager.NewDownloader(sess)
	buffer := aws.NewWriteAtBuffer([]byte{})

	// 使用context设置超时时间
	ctx := context.Background()
	var cancelFn func()
	if timeout > 0 {
		ctx, cancelFn = context.WithTimeout(ctx, timeout)
	}
	if cancelFn != nil {
		defer cancelFn()
	}

	n, err := downloader.DownloadWithContext(ctx, buffer, &s3.GetObjectInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(fmt.Sprintf("%s/%s", prefix, key)),
	})
	if err != nil {
		_ = fmt.Errorf("failed to download file %q, %v", key, err)
		return
	}

	fmt.Printf("file downloaded, %d bytes\n", n)
	return buffer.Bytes(), nil
}
```
# 查看文件列表
```Go
func S3ListObjects() (objects []*s3.Object, err error) {
	var (
		accessKey = "AKIA4AAD"
		secretKey = "eI6adadsaMtQ"
		region    = "eu-west-1"
		bucket    = "my-test"
		timeout   = time.Duration(180) * time.Second
	)
	sess := session.Must(session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
		Region:      aws.String(region),
	}))

	svc := s3.New(sess)

	// 使用context设置超时时间
	ctx := context.Background()
	var cancelFn func()
	if timeout > 0 {
		ctx, cancelFn = context.WithTimeout(ctx, timeout)
	}
	if cancelFn != nil {
		defer cancelFn()
	}

	resp, err := svc.ListObjectsV2WithContext(ctx, &s3.ListObjectsV2Input{Bucket: aws.String(bucket)})
	if err != nil {
		_ = fmt.Errorf("Unable to list items in bucket %q, %v", bucket, err)
		return
	}

	objects = resp.Contents

	// for _, item := range objects {
	// 	fmt.Println("Name:         ", *item.Key)
	// 	fmt.Println("Last modified:", *item.LastModified)
	// 	fmt.Println("Size:         ", *item.Size)
	// 	fmt.Println("Storage class:", *item.StorageClass)
	// 	fmt.Println("")
	// }

	fmt.Println("Found", len(resp.Contents), "items in bucket", bucket)
	return
}
```

# 查看bucket列表
```Go
func S3ListBuckets() (buckets []*s3.Bucket, err error) {
	var (
		accessKey = "AKIA4AAD"
		secretKey = "eI6adadsaMtQ"
		region    = "eu-west-1"
		timeout   = time.Duration(180) * time.Second
	)
	sess := session.Must(session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
		Region:      aws.String(region),
	}))

	svc := s3.New(sess)

	// 使用context设置超时时间
	ctx := context.Background()
	var cancelFn func()
	if timeout > 0 {
		ctx, cancelFn = context.WithTimeout(ctx, timeout)
	}
	if cancelFn != nil {
		defer cancelFn()
	}

	result, err := svc.ListBucketsWithContext(ctx, nil)
	if err != nil {
		_ = fmt.Errorf("Unable to list buckets, %v", err)
		return
	}

	buckets = result.Buckets

	// for _, b := range buckets {
	// 	fmt.Printf("%s created on %s\n", aws.StringValue(b.Name), aws.TimeValue(b.CreationDate))
	// }

	return
}
```

# 获取预签名文件
[参考](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html)
```Go
func GetS3PreSignedURL(cosFilePath string) (string, error) {
	var (
		accessKey = "AKIA4AAD"
		secretKey = "eI6adadsaMtQ"
		region    = "eu-west-1"
		bucket    = "my-test"
	)

	sess := session.Must(session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(accessKey, secretKey, ""),
		Region:      aws.String(region),
	}))

	svc := s3.New(sess)

	req, _ := svc.GetObjectRequest(&s3.GetObjectInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(cosFilePath),
	})

	urlStr, err := req.Presign(7 * 24 * time.Hour)

	if err != nil {
		_ = fmt.Errorf("get s3 pre signed url err: %v", err)
		return "", err
	}

	return urlStr, nil
}
```
