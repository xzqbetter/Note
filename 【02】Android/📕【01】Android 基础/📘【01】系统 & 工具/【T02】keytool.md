# keytool

**`keytool` 密钥库管理工具**：用来管理 "密钥库 keystore" 和 "证书"

- `作用`：通常用于生成、导入、导出、查看密钥和证书

- `密钥库`：是一个文件，通常用于存储加密密钥和证书，例如用于 Android 应用的签名密钥

  通常以 <u>.keystore（通用类型）、 .jks/.pkcs12（特定类型的密钥库）</u> 结尾

- `目录`：位于 `jdk/bin` 目录下

<br/>

## 查看

**`keytool -keystore`**：查看密钥库信息

- `-list`：列出密钥库中的所有条目（例如密钥对或证书）

- `-v`：表示“详细模式”（verbose），会显示每个条目的详细信息，例如：

  - 别名（alias）
  - 证书的创建日期
  - 证书的有效期
  - 证书的指纹（SHA1、SHA256 等）
  - 证书拥有者的详细信息等。

- `-keystore`：指定要操作的密钥库文件

  密钥库是一个文件，通常用于存储加密密钥和证书，例如用于 Android 应用的签名密钥

- `-storepass`：密钥库的密码

- `-storetype`：密钥库的类型

  默认情况下，keytool 假设密钥库是 PKCS12 或 JKS 格式。如果是其他格式，可能需要指定 -storetype 参数

```sh
keytool -list -v -keystore my-release-key.keystore
```

```properties
Keystore type: PKCS12									// 类型
Keystore provider: SunJSSE

Your keystore contains 1 entry

Alias name: myalias										// 别名
Creation date: Apr 24, 2025
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=Your Name, OU=Your Org, O=Your Company, L=City, ST=State, C=Country
Issuer: CN=Your Name, OU=Your Org, O=Your Company, L=City, ST=State, C=Country
Serial number: 123456789
Valid from: Wed Apr 24 12:00:00 CST 2025 until: Mon Apr 24 12:00:00 CST 2050
Certificate fingerprints:							// 证书指纹
         SHA1: 12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78
         SHA256: AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3
```

