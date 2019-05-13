---
title: 将数据导出到 Azure Blob 存储 | Microsoft Docs
description: 如何将数据从 Azure IoT Central 应用程序导出到 Azure Blob 存储
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 03/20/2019
ms.topic: conceptual
ms.service: iot-central
manager: peterpr
ms.openlocfilehash: 9ae57b8ab26780ea975ad74f3348a0deaf8c9cc8
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464628"
---
# <a name="export-your-data-to-azure-blob-storage"></a>将数据导出到 Azure Blob 存储

*本主题适用于管理员。*

本文介绍如何在 Azure IoT Central 中使用连续数据导出功能，以定期将数据导出到您**Azure Blob 存储帐户**。 可以将“度量”、“设备”和“设备模板”导出到 Apache Avro 格式的文件中。 导出的数据可用于冷路径分析，例如 Azure 机器学习中的训练模型或 Microsoft Power BI 中的长期趋势分析。

> [!Note]
> 同样，启用连续数据导出时，只能获得从那时之后的数据。 目前，关闭连续数据导出后将暂时无法检索数据。 若要保留更多的历史数据，请及早启用连续数据导出。


## <a name="prerequisites"></a>必备组件

- 你必须是 IoT Central 应用程序中的管理员


## <a name="set-up-export-destination"></a>设置导出目标

如果没有将导出到一个现有存储，请按照下列步骤：

## <a name="create-storage-account"></a>创建存储帐户

1. [在 Azure 门户中创建新的存储帐户](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM)。 可以在 [Azure 存储文档](https://aka.ms/blobdocscreatestorageaccount)中进行详细的了解。
2. 对于帐户类型，选择“常规用途”或“Blob 存储”。
3. 请选择订阅。 

    > [!Note] 
    > 可以将数据导出到其他订阅，此类订阅**不同于**那些适用于即用即付 IoT Central 应用程序的订阅。 在此示例中，将使用连接字符串进行连接。

4. 在存储帐户中创建容器。 转到存储帐户。 在“Blob 服务”下选择“浏览 Blob”。 选择 **+ 容器**顶部创建一个新的容器


## <a name="set-up-continuous-data-export"></a>设置连续数据导出

现在，已将数据导出到的存储目标，请按照下列步骤来设置持续的数据导出。 

1. 登录到 IoT Central 应用程序。

2. 在左侧菜单中，选择**连续数据导出**。

    > [!Note]
    > 如果在左侧菜单中看不到“连续数据导出”，则说明你在应用中不是管理员。 请与管理员联系以设置数据导出。

    ![创建新的 cde 事件中心](media/howto-export-data/export_menu1.png)

3. 选择 **+ 新建**在右上角的按钮。 选择**Azure Blob 存储**为导出的目标。 

    > [!NOTE] 
    > 每个应用的最大导出数目是 5。 

    ![创建新的连续数据导出](media/howto-export-data/export_new1.png)

4. 在下拉列表框中，选择你**存储帐户命名空间**。 也可选取列表中的最后一个选项，即“输入连接字符串”。 

    > [!NOTE] 
    > 您将只看到存储帐户中的命名空间**与你的 IoT Central 应用相同的订阅**。 若要导出到此订阅外部的某个目标，请选择“输入连接字符串”，然后参阅步骤 5。

    > [!NOTE] 
    > 若要通过 7 天试用期的应用来配置连续事件导出，则唯一的方式是使用连接字符串。 这是因为 7 天试用期的应用没有关联的 Azure 订阅。

    ![创建新的 cde 事件中心](media/howto-export-data/export-create-blob.png)

5. （可选）如果选中了“输入连接字符串”，则会出现一个用于粘贴连接字符串的新框。 若要获取连接字符串，请执行以下操作：
    - 存储帐户，请转到 Azure 门户中的存储帐户。
        - 下**设置**，选择**访问密钥**
        - 复制 key1 连接字符串或 key2 连接字符串
 
6. 从下拉列表框中选择一个容器。

7. 在“要导出的数据”下，通过将类型设置为“打开”来指定要导出的各类数据。

6. 若要启用连续数据导出，请确保将“数据导出”设置为“打开”。 选择“保存”。

   ![配置连续数据导出](media/howto-export-data/export-list-blob.png)

7. 几分钟后，数据便会出现在所选目标中。


## <a name="export-to-azure-blob-storage"></a>导出到 Azure Blob 存储

每分钟一次将度量、设备和设备模板数据导出到存储帐户，每个文件包含自上次导出文件以来所做的批量更改。 导出的数据采用 [Apache Avro](https://avro.apache.org/docs/current/index.html) 格式，并且将导出到三个文件夹中。 存储帐户中的默认路径是：
- 消息：{container}/measurements/{hubname}/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}.avro
- 设备：{container}/devices/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}.avro
- 设备模块：{container}/deviceTemplates/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}.avro

### <a name="measurements"></a>度量

导出的度量数据包含 IoT Central 在此期间从所有设备接收的所有新消息。 导出的文件所用格式与通过 [IoT 中心消息路由](https://docs.microsoft.com/azure/iot-hub/iot-hub-csharp-csharp-process-d2c)导出到 Blob 存储中的消息文件格式相同。

> [!NOTE]
> 发送度量的设备由设备 ID 表示（请参阅以下部分）。 若要获取设备名称，请导出设备快照。 使用与设备记录的 deviceId 匹配的 connectionDeviceId 来关联每条消息记录。

以下示例显示了已解码的 Avro 文件中的记录：

```json
{
    "EnqueuedTimeUtc": "2018-06-11T00:00:08.2250000Z",
    "Properties": {},
    "SystemProperties": {
        "connectionDeviceId": "<connectionDeviceId>",
        "connectionAuthMethod": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "connectionDeviceGenerationId": "<generationId>",
        "enqueuedTime": "2018-06-11T00:00:08.2250000Z"
    },
    "Body": "{\"humidity\":80.59100954598546,\"magnetometerX\":0.29451796907056726,\"magnetometerY\":0.5550332126050068,\"magnetometerZ\":-0.04116681874733441,\"connectivity\":\"connected\",\"opened\":\"triggered\"}"
}
```

### <a name="devices"></a>设备

首次启用连续数据导出时，会导出包含所有设备的单个快照。 每个设备包括：
- IoT Central 中设备的 `id`
- 设备的 `name`
- [设备预配服务](https://aka.ms/iotcentraldocsdps)中的 `deviceId`
- 设备模板信息
- 属性值
- 设置值

每分钟写入一次新的快照。 快照包含：

- 上一快照导出后添加的新设备。
- 上一快照导出后其属性和设置值已更改的设备。

> [!NOTE]
> 上一快照导出后删除的设备不会导出。 目前，快照没有已删除设备标记。
>
> 每台设备所属的设备模板由设备模板 ID 表示。 若要获取设备模板的名称，请导出设备模板快照。

已解码的 Avro 文件中的记录如下所示：

```json
{
    "id": "<id>",
    "name": "Refrigerator 2",
    "simulated": true,
    "deviceId": "<deviceId>",
    "deviceTemplate": {
        "id": "<template id>",
        "version": "1.0.0"
    },
    "properties": {
        "cloud": {
            "location": "New York",
            "maintCon": true,
            "tempThresh": 20
        },
        "device": {
            "lastReboot": "2018-02-09T22:22:47.156Z"
        }
    },
    "settings": {
        "device": {
            "fanSpeed": 0
        }
    }
}
```

### <a name="device-templates"></a>设备模板

首次启用连续数据导出时，会导出包含所有设备模板的单个快照。 每个设备模板包括：
- 设备模板的 `id`
- 设备模板的 `name`
- 设备模板的 `version`
- 度量数据类型和最小/最大值。
- 属性数据类型和默认值。
- 设置数据类型和默认值。

每分钟写入一次新的快照。 快照包含：

- 上一快照导出后添加的新设备模板。
- 上一快照导出后其度量、属性和设置定义已更改的设备模板。

> [!NOTE]
> 上一快照导出后删除的设备模板不会导出。 目前，快照没有已删除设备模板标记。

已解码的 Avro 文件中的记录如下所示：

```json
{
    "id": "<id>",
    "name": "Refrigerated Vending Machine",
    "version": "1.0.0",
    "measurements": {
        "telemetry": {
            "humidity": {
                "dataType": "double",
                "name": "Humidity"
            },
            "magnetometerX": {
                "dataType": "double",
                "name": "Magnetometer X"
            },
            "magnetometerY": {
                "dataType": "double",
                "name": "Magnetometer Y"
            },
            "magnetometerZ": {
                "dataType": "double",
                "name": "Magnetometer Z"
            }
        },
        "states": {
            "connectivity": {
                "dataType": "enum",
                "name": "Connectivity"
            }
        },
        "events": {
            "opened": {
                "name": "Door Opened",
                "category": "informational"
            }
        }
    },
    "settings": {
        "device": {
            "fanSpeed": {
                "dataType": "double",
                "name": "Fan Speed",
                "initialValue": 0
            }
        }
    },
    "properties": {
        "cloud": {
            "location": {
                "dataType": "string",
                "name": "Location",
                "initialValue": "Seattle"
            },
            "maintCon": {
                "dataType": "boolean",
                "name": "Maintenance Contract",
                "initialValue": true
            },
            "tempThresh": {
                "dataType": "double",
                "name": "Temperature Alert Threshold",
                "initialValue": 30
            }
        },
        "device": {
            "lastReboot": {
                "dataType": "dateTime",
                "name": "Last Reboot"
            }
        }
    }
}
```

## <a name="read-exported-avro-files"></a>读取导出的 Avro 文件

Avro 是一种二进制格式，因此无法在其原始状态下读取文件。 这些文件可以解码为 JSON 格式。 下面的示例展示了如何分析度量、设备和设备模板 Avro 文件。 这些示例对应于前一部分中所述的示例。

### <a name="read-avro-files-by-using-python"></a>使用 Python 读取 Avro 文件

#### <a name="install-pandas-and-the-pandavro-package"></a>安装 pandas 和 pandavro 包

```python
pip install pandas
pip install pandavro
```

#### <a name="parse-a-measurements-avro-file"></a>分析度量 Avro 文件

```python
import json
import pandavro as pdx
import pandas as pd

def parse(filePath):
    # Pandavro loads the Avro file into a pandas DataFrame
    # where each record is a single row.
    measurements = pdx.from_avro(filePath)

    # This example creates a new DataFrame and loads a series
    # for each column that's mapped into a column in our new DataFrame.
    transformed = pd.DataFrame()

    # The SystemProperties column contains a dictionary
    # with the device ID located under the connectionDeviceId key.
    transformed["device_id"] = measurements["SystemProperties"].apply(lambda x: x["connectionDeviceId"])

    # The Body column is a series of UTF-8 bytes that is stringified
    # and parsed as JSON. This example pulls the humidity property
    # from each column to get the humidity field.
    transformed["humidity"] = measurements["Body"].apply(lambda x: json.loads(bytes(x).decode('utf-8'))["humidity"])

    # Finally, print the new DataFrame with our device IDs and humidities.
    print(transformed)

```

#### <a name="parse-a-devices-avro-file"></a>分析设备 Avro 文件

```python
import json
import pandavro as pdx
import pandas as pd

def parse(filePath):
    # Pandavro loads the Avro file into a pandas DataFrame
    # where each record is a single row.
    devices = pdx.from_avro(filePath)

    # This example creates a new DataFrame and loads a series
    # for each column that's mapped into a column in our new DataFrame.
    transformed = pd.DataFrame()

    # The device ID is available in the id column.
    transformed["device_id"] = devices["deviceId"]

    # The template ID and version are present in a dictionary under
    # the deviceTemplate column.
    transformed["template_id"] = devices["deviceTemplate"].apply(lambda x: x["id"])
    transformed["template_version"] = devices["deviceTemplate"].apply(lambda x: x["version"])

    # The fanSpeed setting value is located in a nested dictionary
    # under the settings column.
    transformed["fan_speed"] = devices["settings"].apply(lambda x: x["device"]["fanSpeed"])

    # Finally, print the new DataFrame with our device and template
    # information, along with the value of the fan speed.
    print(transformed)

```

#### <a name="parse-a-device-templates-avro-file"></a>分析设备模板 Avro 文件

```python
import json
import pandavro as pdx
import pandas as pd

def parse(filePath):
    # Pandavro loads the Avro file into a pandas DataFrame
    # where each record is a single row.
    templates = pdx.from_avro(filePath)

    # This example creates a new DataFrame and loads a series
    # for each column that's mapped into a column in our new DataFrame.
    transformed = pd.DataFrame()

    # The template and version are available in the id and version columns.
    transformed["template_id"] = templates["id"]
    transformed["template_version"] = templates["version"]

    # The fanSpeed setting value is located in a nested dictionary
    # under the settings column.
    transformed["fan_speed"] = templates["settings"].apply(lambda x: x["device"]["fanSpeed"])

    # Finally, print the new DataFrame with our device and template
    # information, along with the value of the fan speed.
    print(transformed)
```

### <a name="read-avro-files-by-using-c"></a>使用 C# 读取 Avro 文件

#### <a name="install-the-microsofthadoopavro-package"></a>安装 Microsoft.Hadoop.Avro 包

```csharp
Install-Package Microsoft.Hadoop.Avro -Version 1.5.6
```

#### <a name="parse-a-measurements-avro-file"></a>分析度量 Avro 文件

```csharp
using Microsoft.Hadoop.Avro;
using Microsoft.Hadoop.Avro.Container;
using Newtonsoft.Json;

public static async Task Run(string filePath)
{
    using (var fileStream = File.OpenRead(filePath))
    {
        using (var reader = AvroContainer.CreateGenericReader(fileStream))
        {
            // For one Avro container, where a container can contain multiple blocks,
            // loop through each block in the container.
            while (reader.MoveNext())
            {
                // Loop through the Avro records in the block and extract the fields.
                foreach (AvroRecord record in reader.Current.Objects)
                {
                    var systemProperties = record.GetField<IDictionary<string, object>>("SystemProperties");
                    var deviceId = systemProperties["connectionDeviceId"] as string;
                    Console.WriteLine("Device ID: {0}", deviceId);

                    using (var stream = new MemoryStream(record.GetField<byte[]>("Body")))
                    {
                        using (var streamReader = new StreamReader(stream, Encoding.UTF8))
                        {
                            var body = JsonSerializer.Create().Deserialize(streamReader, typeof(IDictionary<string, dynamic>)) as IDictionary<string, dynamic>;
                            var humidity = body["humidity"];
                            Console.WriteLine("Humidity: {0}", humidity);
                        }
                    }
                }
            }
        }
    }
}
```

#### <a name="parse-a-devices-avro-file"></a>分析设备 Avro 文件

```csharp
using Microsoft.Hadoop.Avro;
using Microsoft.Hadoop.Avro.Container;

public static async Task Run(string filePath)
{
    using (var fileStream = File.OpenRead(filePath))
    {
        using (var reader = AvroContainer.CreateGenericReader(fileStream))
        {
            // For one Avro container, where a container can contain multiple blocks,
            // loop through each block in the container.
            while (reader.MoveNext())
            {
                // Loop through the Avro records in the block and extract the fields.
                foreach (AvroRecord record in reader.Current.Objects)
                {
                    // Get the field value directly. You can also yield return
                    // records and make the function IEnumerable<AvroRecord>.
                    var deviceId = record.GetField<string>("deviceId");

                    // The device template information is stored in a sub-record
                    // under the deviceTemplate field.
                    var deviceTemplateRecord = record.GetField<AvroRecord>("deviceTemplate");
                    var templateId = deviceTemplateRecord.GetField<string>("id");
                    var templateVersion = deviceTemplateRecord.GetField<string>("version");

                    // The settings and properties are nested two levels deep.
                    // The first level indicates settings or properties.
                    // The second level indicates the type of setting or property.
                    var settingsRecord = record.GetField<AvroRecord>("settings");
                    var deviceSettingsRecord = settingsRecord.GetField<IDictionary<string, dynamic>>("device");
                    var fanSpeed = deviceSettingsRecord["fanSpeed"];
                    
                    Console.WriteLine(
                        "Device ID: {0}, Template ID: {1}, Template Version: {2}, Fan Speed: {3}",
                        deviceId,
                        templateId,
                        templateVersion,
                        fanSpeed
                    );
                }
            }
        }
    }
}

```

#### <a name="parse-a-device-templates-avro-file"></a>分析设备模板 Avro 文件

```csharp
using Microsoft.Hadoop.Avro;
using Microsoft.Hadoop.Avro.Container;

public static async Task Run(string filePath)
{
    using (var fileStream = File.OpenRead(filePath))
    {
        using (var reader = AvroContainer.CreateGenericReader(fileStream))
        {
            // For one Avro container, where a container can contain multiple blocks,
            // loop through each block in the container.
            while (reader.MoveNext())
            {
                // Loop through the Avro records in the block and extract the fields.
                foreach (AvroRecord record in reader.Current.Objects)
                {
                    // Get the field value directly. You can also yield return
                    // records and make the function IEnumerable<AvroRecord>.
                    var id = record.GetField<string>("id");
                    var version = record.GetField<string>("version");

                    // The settings and properties are nested two levels deep.
                    // The first level indicates settings or properties.
                    // The second level indicates the type of setting or property.
                    var settingsRecord = record.GetField<AvroRecord>("settings");
                    var deviceSettingsRecord = settingsRecord.GetField<IDictionary<string, dynamic>>("device");
                    var fanSpeed = deviceSettingsRecord["fanSpeed"];
                    
                    Console.WriteLine(
                        "ID: {1}, Version: {2}, Fan Speed: {3}",
                        id,
                        version,
                        fanSpeed
                    );
                }
            }
        }
    }
}
```

### <a name="read-avro-files-by-using-javascript"></a>使用 Javascript 读取 Avro 文件

#### <a name="install-the-avsc-package"></a>安装 avsc 包

```javascript
npm install avsc
```

#### <a name="parse-a-measurements-avro-file"></a>分析度量 Avro 文件

```javascript
const avro = require('avsc');

// Read the Avro file. Parse the device ID and humidity from each record.
async function parse(filePath) {
    const records = await load(filePath);
    for (const record of records) {
        // Fetch the device ID from the system properties.
        const deviceId = record.SystemProperties.connectionDeviceId;

        // Convert the body from a buffer to a string and parse it.
        const body = JSON.parse(record.Body.toString());

        // Get the humidty property from the body.
        const humidity = body.humidity;

        // Log the retrieved device ID and humidity.
        console.log(`Device ID: ${deviceId}`);
        console.log(`Humidity: ${humidity}`);
    }
}

function load(filePath) {
    return new Promise((resolve, reject) => {
        // The file decoder emits each record as a data event on a stream.
        // Collect the records into an array and return them at the end.
        const records = [];
        avro.createFileDecoder(filePath)
            .on('data', record => { records.push(record); })
            .on('end', () => resolve(records))
            .on('error', reject);
    });
}
```

#### <a name="parse-a-devices-avro-file"></a>分析设备 Avro 文件

```javascript
const avro = require('avsc');

// Read the Avro file. Parse the device and template identification
// information and the fanSpeed setting for each device record.
async function parse(filePath) {
    const records = await load(filePath);
    for (const record of records) {
        // Fetch the device ID from the deviceId property.
        const deviceId = record.deviceId;

        // Fetch the template ID and version from the deviceTemplate property.
        const deviceTemplateId = record.deviceTemplate.id;
        const deviceTemplateVersion = record.deviceTemplate.version;

        // Get the fanSpeed from the nested device settings property.
        const fanSpeed = record.settings.device.fanSpeed;

        // Log the retrieved device ID and humidity.
        console.log(`deviceID: ${deviceId}, Template ID: ${deviceTemplateId}, Template Version: ${deviceTemplateVersion}, Fan Speed: ${fanSpeed}`);
    }
}

function load(filePath) {
    return new Promise((resolve, reject) => {
        // The file decoder emits each record as a data event on a stream.
        // Collect the records into an array and return them at the end.
        const records = [];
        avro.createFileDecoder(filePath)
            .on('data', record => { records.push(record); })
            .on('end', () => resolve(records))
            .on('error', reject);
    });
}
```

#### <a name="parse-a-device-templates-avro-file"></a>分析设备模板 Avro 文件

```javascript
const avro = require('avsc');

// Read the Avro file. Parse the device and template identification
// information and the fanSpeed setting for each device record.
async function parse(filePath) {
    const records = await load(filePath);
    for (const record of records) {
        // Fetch the template ID and version from the id and verison properties.
        const templateId = record.id;
        const templateVersion = record.version;

        // Get the fanSpeed from the nested device settings property.
        const fanSpeed = record.settings.device.fanSpeed;

        // Log the retrieved device id and humidity.
        console.log(`Template ID: ${templateId}, Template Version: ${templateVersion}, Fan Speed: ${fanSpeed}`);
    }
}

function load(filePath) {
    return new Promise((resolve, reject) => {
        // The file decoder emits each record as a data event on a stream.
        // Collect the records into an array and return them at the end.
        const records = [];
        avro.createFileDecoder(filePath)
            .on('data', record => { records.push(record); })
            .on('end', () => resolve(records))
            .on('error', reject);
    });
}
```

## <a name="next-steps"></a>后续步骤

了解如何导出数据后，继续进行下一步：

> [!div class="nextstepaction"]
> [如何在 Power BI 中直观显示数据](howto-connect-powerbi.md)
