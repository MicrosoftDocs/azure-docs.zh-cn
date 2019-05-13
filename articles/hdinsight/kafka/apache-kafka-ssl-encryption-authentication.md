---
title: 设置 SSL 加密和 Azure HDInsight 中的 Apache Kafka 的身份验证
description: 设置 Kafka 客户端和 Kafka 中转站也之间 Kafka 中转站之间的通信的 SSL 加密。 设置 SSL 身份验证的客户端。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/01/2019
ms.author: hrasheed
ms.openlocfilehash: 5d567074a0038915cc43a585b34c9c71ccf3eb1b
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464999"
---
# <a name="set-up-secure-sockets-layer-ssl-encryption-and-authentication-for-apache-kafka-in-azure-hdinsight"></a>设置安全套接字层 (SSL) 加密和 Azure HDInsight 中的 Apache Kafka 的身份验证

本文介绍如何设置 Apache Kafka 客户端和 Apache Kafka 代理之间的 SSL 加密。 它还演示如何设置身份验证的客户端 （有时称为双向 SSL）。

> [!Important]
> 有两个客户端可以使用的 Kafka 应用程序： Java 客户端和控制台客户端。 Java 客户端`ProducerConsumer.java`可以使用 SSL 进行生成和使用。 控制台制造者客户端`console-producer.sh`并不适用于 SSL。

## <a name="apache-kafka-broker-setup"></a>Apache Kafka 代理安装程序

Kafka SSL 代理安装程序将使用四种 HDInsight 群集 Vm，如下所示：

* 头节点 0-证书颁发机构 (CA)
* 辅助角色节点 0、 1 和 2-中转站

> [!Note] 
> 本指南使用自签名证书，但最安全的解决方案是使用受信任 CA 颁发的证书。

代理安装过程的摘要如下所示：

1. 每个三个辅助角色节点上，则重复以下步骤：

    1. 生成的证书。
    1. 创建证书签名请求。
    1. 发送证书签名请求到证书颁发机构 (CA)。
    1. 登录到 CA 并登录请求。
    1. SCP 的签名证书备份到辅助角色节点。
    1. SCP 到辅助角色节点的 CA 的公用证书。

1. 一次你具有的所有证书，将证书放入的证书存储。
1. 转到 Ambari 和更改的配置。

使用以下详细的说明完成代理安装程序：

> [!Important]
> 以下代码片段中 wnX 是三个辅助角色节点之一的缩写，并应替换`wn0`，`wn1`或`wn2`根据需要。 `WorkerNode0_Name` 并`HeadNode0_Name`应如具有各自的计算机，名称替换`wn0-abcxyz`或`hn0-abcxyz`。

1. 头节点 0，这对于 HDInsight 中将充当的角色的证书颁发机构 (CA) 上执行初始设置。

    ```bash
    # Create a new directory 'ssl' and change into it
    mkdir ssl
    cd ssl
    ```

1. 每个代理 （辅助角色节点 0、 1 和 2） 上执行相同的初始设置。

    ```bash
    # Create a new directory 'ssl' and change into it
    mkdir ssl
    cd ssl
    ```

1. 在每个辅助角色节点上执行以下步骤使用以下代码段。
    1. 创建密钥存储并使用新的私有证书填充它。
    1. 创建证书签名请求。
    1. SCP 证书签名请求到 CA (headnode0)

    ```bash
    keytool -genkey -keystore kafka.server.keystore.jks -validity 365 -storepass "MyServerPassword123" -keypass "MyServerPassword123" -dname "CN=FQDN_WORKER_NODE" -storetype pkcs12
    keytool -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass "MyServerPassword123" -keypass "MyServerPassword123"
    scp cert-file sshuser@HeadNode0_Name:~/ssl/wnX-cert-sign-request
    ```

1. 将更改为 CA 计算机并签署所有收到的证书签名请求：

    ```bash
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn0-cert-sign-request -out wn0-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn1-cert-sign-request -out wn1-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn2-cert-sign-request -out wn2-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    ```

1. 从 CA (headnode0) 返回到辅助角色节点发送的签名的证书。

    ```bash
    scp wn0-cert-signed sshuser@WorkerNode0_Name:~/ssl/cert-signed
    scp wn1-cert-signed sshuser@WorkerNode1_Name:~/ssl/cert-signed
    scp wn2-cert-signed sshuser@WorkerNode2_Name:~/ssl/cert-signed
    ```

1. 将 CA 的公用证书发送到每个辅助角色节点。

    ```bash
    scp ca-cert sshuser@WorkerNode0_Name:~/ssl/ca-cert
    scp ca-cert sshuser@WorkerNode1_Name:~/ssl/ca-cert
    scp ca-cert sshuser@WorkerNode2_Name:~/ssl/ca-cert
    ```

1. 在每个辅助角色节点上，将添加到信任存储和密钥存储的 Ca 公用证书。 将辅助角色节点的签名的证书添加到密钥存储

    ```bash
    keytool -keystore kafka.server.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt
    keytool -keystore kafka.server.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt
    keytool -keystore kafka.server.keystore.jks -import -file cert-signed -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt

    ```

## <a name="update-kafka-configuration-to-use-ssl-and-restart-brokers"></a>将 Kafka 配置更新为使用 SSL 并重启代理

你现在已设置密钥存储和 truststore，每个 Kafka 中转站并导入正确的证书。 接下来，请使用 Ambari 修改相关的 Kafka 配置属性，然后重启 Kafka 代理。

若要完成配置修改，请按照以下步骤操作：

1. 登录到 Azure 门户，并选择你的 Azure HDInsight Apache Kafka 群集。
1. 单击**群集仪表板**下面的“Ambari 主页”转到 Ambari UI。
1. 在“Kafka 代理”下，将 **listeners** 属性设置为 `PLAINTEXT://localhost:9092,SSL://localhost:9093`
1. 在“高级 kafka-broker”下，将 **security.inter.broker.protocol** 属性设置为 `SSL`

    ![在 Ambari 中编辑 Kafka ssl 配置属性](./media/apache-kafka-ssl-encryption-authentication/editing-configuration-ambari.png)

1. 在“自定义 kafka-broker”下，将 **ssl.client.auth** 属性设置为 `required`。 此步骤才需要如果您设置了身份验证和加密。

    ![在 Ambari 中编辑 kafka ssl 配置属性](./media/apache-kafka-ssl-encryption-authentication/editing-configuration-ambari2.png)

1. 将配置属性添加到 Kafka `server.properties` 文件，以播发 IP 地址而不是完全限定的域名 (FQDN)。

    ```bash
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092,SSL://$IP_ADDRESS:9093" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.keystore.location=/home/sshuser/ssl/kafka.server.keystore.jks" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.keystore.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.key.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.truststore.location=/home/sshuser/ssl/kafka.server.truststore.jks" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.truststore.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

1. 若要验证前面所做的更改是否正确，可以选择性地检查以下行是否在 Kafka `server.properties` 文件中存在。

    ```bash
    advertised.listeners=PLAINTEXT://10.0.0.11:9092,SSL://10.0.0.11:9093
    ssl.keystore.location=/home/sshuser/ssl/kafka.server.keystore.jks
    ssl.keystore.password=MyServerPassword123
    ssl.key.password=MyServerPassword123
    ssl.truststore.location=/home/sshuser/ssl/kafka.server.truststore.jks
    ssl.truststore.password=MyServerPassword123
    ```

1. 重启所有 Kafka 代理。
1. 开始生成者和使用者的选项，以确认生产者和使用者使用端口 9093 上管理客户端。

## <a name="client-setup-with-authentication"></a>客户端设置（使用身份验证）

> [!Note]
> 仅当同时设置了 SSL 加密**和**身份验证时，才需要执行以下步骤。 如果仅设置了加密，请转到[不使用身份验证的客户端设置](apache-kafka-ssl-encryption-authentication.md#client-setup-without-authentication)

完成以下步骤以完成客户端设置：

1. 登录到客户端计算机 (hn1)。
1. 创建 Java 密钥存储并获取代理的已签名证书。 然后将该证书复制到运行 CA 的 VM。
1. 切换到 CA 计算机 (hn0) 客户端证书进行签名。
1. 转到客户端计算机 (hn1) 并导航到 `~/ssl` 文件夹。 将已签名的证书复制到客户端计算机。

```bash
cd ssl

# Create a java keystore and get a signed certificate for the broker. Then copy the certificate to the VM where the CA is running.

keytool -genkey -keystore kafka.client.keystore.jks -validity 365 -storepass "MyClientPassword123" -keypass "MyClientPassword123" -dname "CN=mylaptop1" -alias my-local-pc1 -storetype pkcs12

keytool -keystore kafka.client.keystore.jks -certreq -file client-cert-sign-request -alias my-local-pc1 -storepass "MyClientPassword123" -keypass "MyClientPassword123"

# Copy the cert to the CA
scp client-cert-sign-request3 sshuser@HeadNode0_Name:~/tmp1/client-cert-sign-request

# Switch to the CA machine (hn0) to sign the client certificate.
cd ssl
openssl x509 -req -CA ca-cert -CAkey ca-key -in /tmp1/client-cert-sign-request -out /tmp1/client-cert-signed -days 365 -CAcreateserial -passin pass:MyServerPassword123

# Return to the client machine (hn1), navigate to ~/ssl folder and copy signed cert from the CA (hn0) to client machine
scp -i ~/kafka-security.pem sshuser@HeadNode0_Name:/tmp1/client-cert-signed

# Import CA cert to trust store
keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt

# Import CA cert to key store
keytool -keystore kafka.client.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt

# Import signed client (cert client-cert-signed1) to keystore
keytool -keystore kafka.client.keystore.jks -import -file client-cert-signed -alias my-local-pc1 -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
```

最后，运行命令 `cat client-ssl-auth.properties`，以查看文件 `client-ssl-auth.properties`。 该文件应包含以下行：

```bash
security.protocol=SSL
ssl.truststore.location=/home/sshuser/ssl/kafka.client.truststore.jks
ssl.truststore.password=MyClientPassword123
ssl.keystore.location=/home/sshuser/ssl/kafka.client.keystore.jks
ssl.keystore.password=MyClientPassword123
ssl.key.password=MyClientPassword123
```

## <a name="client-setup-without-authentication"></a>客户端设置（不使用身份验证）

如果不需要身份验证，以设置仅 SSL 加密的步骤如下：

1. 登录客户端计算机 (hn1)，并转到 `~/ssl` 文件夹
1. 将已签名的证书从 CA 计算机 (wn0) 复制到客户端计算机。
1. 将 CA 证书导入信任存储
1. 将 CA 证书导入密钥存储

下面的代码片段展示了这些步骤。

```bash
cd ssl

# Copy signed cert to client machine
scp -i ~/kafka-security.pem sshuser@wn0-umakaf:/home/sshuser/ssl/ca-cert .

# Import CA cert to truststore
keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt

# Import CA cert to keystore
keytool -keystore kafka.client.keystore.jks -alias CARoot -import -file cert-signed -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
```

最后，运行命令 `cat client-ssl-auth.properties`，以查看文件 `client-ssl-auth.properties`。 该文件应包含以下行：

```bash
security.protocol=SSL
ssl.truststore.location=/home/sshuser/ssl/kafka.client.truststore.jks
ssl.truststore.password=MyClientPassword123
```

## <a name="next-steps"></a>后续步骤

* [什么是 Apache Kafka on HDInsight？](apache-kafka-introduction.md)
