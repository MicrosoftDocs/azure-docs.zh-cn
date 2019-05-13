---
title: 将品牌添加到组织的登录页面 - Azure Active Directory | Microsoft Docs
description: 有关如何将组织的品牌添加到 Azure Active Directory 登录页面的说明。
services: active-directory
author: eross-msft
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 09/18/2018
ms.author: lizross
ms.reviewer: kexia
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 277e7663c978e64ee1440e14583e884b768b3139
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65441639"
---
# <a name="add-branding-to-your-organizations-azure-active-directory-sign-in-page"></a>将品牌添加到组织的 Azure Active Directory 登录页面
使用组织的徽标和自定义颜色方案，在 Azure Active Directory (Azure AD) 登录页面上提供一致外观和体验。 用户登录到组织的基于 Web 的应用（例如 Office 365，它使用 Azure AD 作为标识提供者时）时，将显示登录页面。

>[!Note]
>添加自定义品牌需要使用 Azure Active Directory Premium 1、Premium 2 或 Basic 版，或拥有 Office 365 许可证。 有关许可和版本的详细信息，请参阅[注册 Azure AD Premium](active-directory-get-started-premium.md)。<br><br>在中国，使用 Azure Active Directory 全球实例的客户可以使用 Azure AD 高级和基本版。 中国区 21Vianet 运营的 Azure 服务目前不支持 Azure AD Premium 和 Azure AD Basic 版本。 有关详细信息，请通过 [Azure Active Directory 论坛](https://feedback.azure.com/forums/169401-azure-active-directory/)与我们联系。

## <a name="customize-your-azure-ad-sign-in-page"></a>自定义 Azure AD 登录页面
可以自定义 Azure AD 登录页面，这些页面会在用户登录到组织的特定于租户的应用时显示（例如 [*https://outlook.com/contoso.com*](https://outlook.com/contoso.com)），或者在传递域变量时显示（例如 [*https://passwordreset.microsoftonline.com/?whr=contoso.com*](https://passwordreset.microsoftonline.com/?whr=contoso.com)）。

当用户转到如 www 站点时你的自定义品牌不会立即显示\.office.com。 相反，用户必须先登录才会显示自定义品牌。

> [!NOTE]
> 所有品牌元素都是可选的。 例如，如果指定没有背景图像的横幅徽标，则登录页面将显示带有目标网站（例如 Office 365）默认背景图像的徽标。<br><br>此外，登录页面品牌不会带到个人 Microsoft 帐户。 如果用户或企业访客使用个人 Microsoft 帐户登录，则其登录页面不会显示组织的品牌。

### <a name="to-customize-your-branding"></a>自定义品牌
1. 使用目录的全局管理员帐户登录到 [Azure 门户](https://portal.azure.com/)。

2. 选择“Azure Active Directory”，然后依次选择“公司品牌”、“配置”。

    ![Contoso - 公司品牌页面，突出显示了“配置”选项](media/customize-branding/company-branding-configure-button.png)

3. 在“配置公司品牌”页面上，提供以下任意或全部信息。

    >[!Important]
    >在此页面上添加的所有自定义图像都具有图像尺寸（像素）限制，可能有文件大小 (KB) 限制。 由于这些限制，很可能需要使用照片编辑器来创建合适尺寸的图像。

    - **常规设置**

        ![配置公司品牌页面，其中常规设置已完成](media/customize-branding/configure-company-branding-general-settings.png)

        - **语言。** 语言自动设置为默认语言，无法更改。
        
        - **登录页面背景图像。** 选择要显示为登录页面背景的 .png 或 .jpg 图像文件。 
        
            图像大小不能超过 1920 x 1080 像素，并且其中的文件的大小必须小于 300 KB。

        - **横幅徽标。** 用户在“我的应用”门户页面上输入用户名后，选择要显示在登录页面上的徽标的 .png 或 .jpg 版本。
            
            图像不能高于 36 像素或宽于 245 像素。 我们建议使用透明图像，因为背景可能与徽标背景不匹配。 我们还建议不要在图像周围添加填充，否则可能会使徽标看起来很小。

        - **用户名提示。** 键入提示文本，如果用户忘记用户名将会显示此提示文本。 此文本必须是 Unicode，不带链接或代码，且不能超过 64 个字符。 如果访客登录到应用，我们建议不添加此提示。

        - **登录页面文本。** 键入显示在登录页面底部的文本。 可使用此文本传达其他信息，例如技术支持的电话号码或法律声明。 此文本必须为 Unicode，并且不能超过 256 个字符。 我们还建议不包括链接或 HTML 标记。

    - **高级设置**
            
        ![配置公司品牌页面，其中高级设置已完成](media/customize-branding/configure-company-branding-advanced-settings.png)   

        - **登录页背景色。** 指定在低带宽连接的情况下替代背景图像显示的十六进制颜色（例如，白色为 #FFFFFF）。 我们建议使用横幅徽标的主颜色或组织的颜色。

        - **方形徽标图像。** 选择要在新 Windows 10 企业版设备的安装过程中向用户显示的组织徽标的 .png（首选）或 .jpg 图像。 此图像仅用于 Windows 身份验证，并且仅显示在使用 [Windows Autopilot]( https://docs.microsoft.com/windows/deployment/windows-autopilot/windows-10-autopilot) 进行部署的租户上；或者在其他 Windows 10 体验中用于密码输入页面。 在某些情况下它也可能在同意对话框出现。
        
            图像大小不能超过 240 x 240 像素，并且其中的文件的大小必须小于 10 KB。 我们建议使用透明图像，因为背景可能与徽标背景不匹配。 我们还建议不要在图像周围添加填充，否则可能会使徽标看起来很小。
    
        - **方形徽标，深色主题。** 与上面的方形徽标图像相同。 当在深色背景下使用（例如在开箱即用体验 (OOBE) 中使用已加入 Windows 10 Azure AD 的屏幕）时，此徽标图像取代方形徽标图像。  如果徽标在白色、深蓝色和黑色背景上看起来不错，则无需添加此图像。 
        
        - **显示保持登录状态的选项。** 你可以选择让用户保持一直登录到 Azure AD 的状态，直到明确注销。如果选择“否”，则此选项将被隐藏，用户每次关闭并重新打开浏览器时必须登录。
        
            >[!Note]
            >SharePoint Online 和 Office 2010 的某些功能取决于用户能否选择保持登录状态。 如果将此选项设置为“否”，用户会在登录时可能看到其他意外的提示。
   

3. 已完成添加品牌后，选择“保存”。

    如果此过程创建了首个自定义品牌配置，则它将成为租户的默认配置。 如果有其他配置，你可以选择默认配置。
    
    >[!Important]
    >要向租户添加更多公司品牌配置，你必须在“Contoso - 公司品牌”页面上选择“新语言”。 这将打开“配置公司品牌”页面，你可以按照上述相同步骤进行操作。

## <a name="update-your-custom-branding"></a>更新自定义品牌
创建自定义品牌后，可以返回并更改任何所需内容。

### <a name="to-edit-your-custom-branding"></a>编辑自定义品牌
1. 使用目录的全局管理员帐户登录到 [Azure 门户](https://portal.azure.com/)。

2. 选择“Azure Active Directory”，然后依次选择“公司品牌”、“配置”。

    ![“Contoso - 公司品牌”页面，其中显示默认配置](media/customize-branding/company-branding-default-config.png)

3. 在“配置公司品牌”页面上，根据本文的[自定义 Azure AD 登录页面](#customize-your-azure-ad-sign-in-page)部分中的说明添加、删除或更改任何信息。

4. 选择“保存”。

   最长可能需要一个小时才能显示用户对登录页的品牌元素所做的任何更改。

## <a name="add-language-specific-company-branding-to-your-directory"></a>将特定语言的公司品牌元素添加到目录
无法将更改原始配置的语言从默认语言更改为其他语言。 但是，如果需要其他语言的配置，则可以创建新的配置。

### <a name="to-add-a-language-specific-branding-configuration"></a>添加特定于语言的品牌配置

1. 使用目录的全局管理员帐户登录到 [Azure 门户](https://portal.azure.com/)。

2. 选择“Azure Active Directory”，然后依次选择“公司品牌”、“新语言”。

    ![“Contoso - 公司品牌”页面，突出显示了“新语言”选项](media/customize-branding/company-branding-new-language.png)

3. 在“配置公司品牌”页面上，选择语言（例如法语），然后根据本文的[自定义 Azure AD 登录页面](#customize-your-azure-ad-sign-in-page)部分中的说明添加已翻译的信息。

4. 选择“保存”。

    “Contoso - 公司品牌”页面更新以显示新的法语配置。

    ![“Contoso - 公司品牌”页面，其中显示默认配置](media/customize-branding/company-branding-french-config.png)

## <a name="add-your-custom-branding-to-pages"></a>将自定义品牌添加到页面
通过使用文本 `?whr=yourdomainname` 修改 URL 的结尾，将自定义品牌添加到页面。 此修改适用于多个页面，包括多重身份验证 (MFA) 设置页面、自助服务密码重置 (SSPR) 设置页面和登录页面。

**示例：**

**原始 URL：**https://aka.ms/MFASetup<br>
**自定义 URL：**https://account.activedirectory.windowsazure.com/proofup.aspx?whr=contoso.com

**原始 URL：**https://aka.ms/SSPR<br>
**自定义 URL：**https://passwordreset.microsoftonline.com/?whr=contoso.com

 
