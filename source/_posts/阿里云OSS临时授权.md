---
title: '阿里云OSS临时授权'
date: 2020-03-31
categories: 
  - 阿里云OSS
tags:
  - 存储
---
测试环境:

```
1. 待操作的BucketName: ram-test
```

一.创建子账户
可以在云账号登录[RAM控制台](https://ram.console.aliyun.com/users)添加子账户
该步骤关键在于
a.获取并保存子账户的 访问密钥（AccessKeyId 和 AccessKeySecret）
b.以及为已创建的子账号添加AliyunSTSAssumeRoleAccess权限

二.添加自定义策略权限
添加自定义策略权限,该步骤可以限定临时授权的用户有什么操作权限(示例中ram-test指代某一个bucketName)

```
{
    "Version": "1",
    "Statement": [
     {
           "Effect": "Allow",
           "Action": [
             "oss:PutObject",
             "oss:GetObject"
           ],
           "Resource": [
             "acs:oss:*:*:ram-test",
             "acs:oss:*:*:ram-test/*"
           ]
     }
    ]
}
```

三、创建角色并记录角色的ARN
创建角色后,关联第二步所自定义的权限.创建完成后获取该角色的ARN

```java
import com.alibaba.fastjson.JSON;
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.auth.sts.AssumeRoleRequest;
import com.aliyuncs.auth.sts.AssumeRoleResponse;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.http.MethodType;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;

public class TestAliyunOssStsToken {

    public static void main(String[] args) {
        String endpoint = "<sts-endpoint>";
        // String endpoint = sts.cn-hangzhou.aliyuncs.com;
        String AccessKeyId = "<access-key-id>";
        String accessKeySecret = "<access-key-secret>";
        String roleArn = "<role-arn>";
        String roleSessionName = "<session-name>";
        String policy = "<policy>";

        try {
            // 添加endpoint（直接使用STS endpoint，前两个参数留空，无需添加region ID）
            DefaultProfile.addEndpoint("", "", "Sts", endpoint);
            // 构造default profile（参数留空，无需添加region ID）
            IClientProfile profile = DefaultProfile.getProfile("", AccessKeyId, accessKeySecret);
            // 用profile构造client
            DefaultAcsClient client = new DefaultAcsClient(profile);
            final AssumeRoleRequest request = new AssumeRoleRequest();
            request.setMethod(MethodType.POST);
            request.setRoleArn(roleArn);
            request.setRoleSessionName(roleSessionName);
            request.setDurationSeconds(1000L); // 设置凭证有效时间
            final AssumeRoleResponse response = client.getAcsResponse(request);
            System.out.println("User:"+JSON.toJSONString(response.getAssumedRoleUser()));
            System.out.println("Credentials:"+ JSON.toJSONString(response.getCredentials()));
            System.out.println("Expiration: " + response.getCredentials().getExpiration());
            System.out.println("Access Key Id: " + response.getCredentials().getAccessKeyId());
            System.out.println("Access Key Secret: " + response.getCredentials().getAccessKeySecret());
            System.out.println("Security Token: " + response.getCredentials().getSecurityToken());
            System.out.println("RequestId: " + response.getRequestId());
        } catch (ClientException e) {
            System.out.println("Failed：");
            System.out.println("Error code: " + e.getErrCode());
            System.out.println("Error message: " + e.getErrMsg());
            System.out.println("RequestId: " + e.getRequestId());
        }
    }

}

```

需要特别说明的是

1. endpoint:STS接入地址，例如sts.cn-hangzhou.aliyuncs.com(杭州,注意没有http之类)。各地域的STS接入地址请参见[接入地址](https://help.aliyun.com/document_detail/66053.html?spm=a2c4g.11186623.2.23.3cb1107emZlciL#reference-sdg-3pv-xdb)
2. AccessKeyId、AccessKeySecret：步骤1中保存的访问密钥。
3. RoleArn：步骤3中保存的角色ARN。
4. RoleSessionName：用来标识临时访问凭证的名称，建议使用不同的应用程序用户来区分。[这个由用户自定义来区分凭证使用场景,这个会在返回值中AssumedRoleUser有所体现]
5. Policy：在扮演角色的时候额外添加的权限限制。请参见基于RAM Policy的权限控制。[笔者一般情况是是放空的,交给步骤2完成了,如果还需要个性化定制才需要设置该Policy]

[STS临时授权访问OSS](https://help.aliyun.com/document_detail/100624.html?spm=a2c4g.11186623.6.706.1deb7774EN7jjQ)
