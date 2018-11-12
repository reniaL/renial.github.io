---
layout: post
title:  "God Damn Code"
tags: [code]
---

曾在 StackOverflow 上看到过一个问题 [What is the best comment in source code you have ever encountered?](http://stackoverflow.com/questions/184618/what-is-the-best-comment-in-source-code-you-have-ever-encountered) ，里面的回答真是非一般的爆笑。

然后想起自己写代码这么多年，也见过不少鬼斧神工、偷龙转凤、我行我素的代码（嗯就是这么有个性）。不信，你看：

**虽然长着 Get 的外表，但我有一颗 Post 的心**

```java
PostMethod getMethod = new PostMethod(url);
```

**我真的只是一个工具类**

```java
public class CommonUtil extends ActionSupport {
```

**你猜我是Service还是DAO**

```java
@Service
@Transactional
public class UserService extends SimpleJdbcSupport
```

**3层不多，5层太少，7、8层才有味道**

```java
for (Element e : es) {
    ...
    if (billingGroupId.equalsIgnoreCase(queryBillingGroupId)) {
        List<Element> eb = e.elements();
        for (Element be : eb) {
            long totalData = 0;
            if (!be.getName().endsWith("BillingGroupId")) {
                ...
                for (String region : regions) {
                    ...
                    for (String billType : billTypeMap.keySet()) {
                        if (billTypeMap.get(billType).contains(billingId)) {
                            if (billType.equals(DataType.BILLING_TYPE_TOTAL)) {
                                datas = DataUtils.mergeData(inData, outData);
                            } else if (billType.equals(DataType.BILLING_TYPE_OUT)) {
                                datas = outData;
                            } else if (billType.equals(DataType.BILLING_TYPE_IN)) {
                                ...
```

**我传，我传，我传传传**

```java
            try {
                dur = dfsClient.uploadFile(groupname,localFile, fileType);
            } catch (Exception e) {
                ret = 0;
                logger.error("#上传文件错误#, localFile={}", localFile, e);
            }
            if (dur == null) {
                logger.error("#上传文件错误#, localFile={}", localFile, e);
                dur = dfsClient.uploadFile(localFile, fileType);
            }
            if (dur == null) {
                logger.error("#再次上传文件错误#, localFile={}", localFile, e);
                dur = dfsClient.uploadFile(localFile, fileType);
            }
            if (dur == null) {
                logger.error("#再再次上传文件错误#, localFile={}", localFile, e);
                return 0;
            }
```
