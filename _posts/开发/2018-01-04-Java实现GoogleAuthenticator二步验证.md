---
title: Java 实现 Google Authenticator 二步验证
categories: 开发
toc: true
permalink: /java-google-authenticator.html
---

Google Authenticator 采用 TOTP 算法（Time-Based One-Time Password，基于时间的一次性密码），算法参考最后的参考资料，使用 30 秒的时间片。

## maven依赖

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

## 生成密钥

```java
    public static String generateSecretKey(String seed) throws NoSuchAlgorithmException {
        SecureRandom sr = SecureRandom.getInstance("SHA1PRNG");
        sr.setSeed(Base64.decodeBase64(seed));
        byte[] buffer = sr.generateSeed(10);
        Base32 codec = new Base32();
        byte[] bEncodedKey = codec.encode(buffer);
        return new String(bEncodedKey);
    }

    // 输出类似：I6FSPKBM3PTAY5GT
```

## 生成TOTP字符串

前端可根据服务端生成的TOTP字符串生成二维码图片，用户在身份验真器扫码添加。

```java
    public static String generateTotpUrl(String user, String secretKey, String issuer) {
        String format = "otpauth://totp/%s?secret=%s&issuer=%s";
        return String.format(format, user, secretKey, issuer);
    }

    // 输出类似：otpauth://totp/username?secret=I6FSPKBM3PTAY5GT&issuer=hiwzc.com

```

## 生成验证码

身份验证器每30秒更新一次验证码，服务端校验时根据系统时间和密钥生成验证码。

```java
    // 这里的c值已经是除以30s后的数值
    public static int generateCode(byte[] key, long c) throws NoSuchAlgorithmException, InvalidKeyException {
        byte[] data = new byte[8];
        long value = c;
        for (int i = 8; i-- > 0; value >>>= 8) {
            data[i] = (byte) value;
        }
        SecretKeySpec signKey = new SecretKeySpec(key, "HmacSHA1");
        Mac mac = Mac.getInstance("HmacSHA1");
        mac.init(signKey);
        byte[] hash = mac.doFinal(data);
        int offset = hash[20 - 1] & 0xF;
        long truncatedHash = 0;
        for (int i = 0; i < 4; ++i) {
            truncatedHash <<= 8;
            truncatedHash |= (hash[offset + i] & 0xFF);
        }
        truncatedHash &= 0x7FFFFFFF;
        truncatedHash %= 1000000;
        return (int) truncatedHash;
    }
```

## 校验验证码

为了避免用户在验证码快过期才输入，或者网络延时等因素造成验证失败，或者需要提前使用后面的验证码，服务端通常会校验当前时间前后的若干个验证码。

说明一下什么情况会需要提前使用后面的验证码，为防止攻击，一个验证码通过验证后会被服务端记录为失效，假如用户需要连续做好几个需验证码的操作，比如企业内部需要登陆好几个系统时，就需要使用后面时间生成的验证码。

```java
    public static boolean checkCode(String secret, long code, long c) {
        Base32 codec = new Base32();
        byte[] decodedKey = codec.decode(secret);
        for (int i = -1; i < 5; ++i) {
            long hash;
            try {
                hash = generateCode(decodedKey, c + i);
            } catch (Exception e) {
                e.printStackTrace();
                throw new RuntimeException(e.getMessage());
            }
            if (hash == code) {
                return true;
            }
        }
        return false;
    }
```

## 测试程序

这里使用 Android APP 腾讯身份验证器，如下图。

```java
    public static void main(String[] args) throws Exception {
        // 生成密钥
        String secretKey = generateSecretKey("hiwzc.com");
        System.out.println(secretKey);

        // 生成协议字符串
        String totpUrl = generateTotpUrl("username", secretKey, "hiwzc.com");
        System.out.println(totpUrl);

        // 校验验证码
        while (true) {
            Scanner scanner = new Scanner(System.in);
            int input = scanner.nextInt();
            if (input < 0) {
                break;
            }
            Date now = new Date();
            long c = now.getTime() / 30000;
            System.out.println(checkCode(secretKey, input, c));

        }

        // 生成验证码
        SimpleDateFormat format = new SimpleDateFormat("hh:mm:ss");
        while (true) {
            Date now = new Date();
            long c = now.getTime() / 30000;
            int code = generateCode(new Base32().decode(secretKey), c);
            System.out.printf("%s %06d\n",  format.format(now), code);
            Thread.sleep(1000);
        }
    }
```

![腾讯身份验证器界面](/assets/img/totp-app.jpg)

## 参考资料

- [TOTP: Time-based One-time Password Algorithm, RFC Draft](http://tools.ietf.org/id/draft-mraihi-totp-timebased-06.html)
- [HOTP: An HMAC-Based One-Time Password Algorithm, RFC 4226](http://tools.ietf.org/html/rfc4226)
- [Google Authenticator project](http://code.google.com/p/google-authenticator/)