## <a name="microsoft-open-source-code-of-conduct"></a>Microsoft 开放源代码行为准则

本项目采用 [Microsoft 开源行为准则](https://opensource.microsoft.com/codeofconduct/)。
有关详细信息，请参阅[行为准则常见问题](https://opensource.microsoft.com/codeofconduct/faq/)；若要其他任何问题或意见，请联系 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## <a name="contribute-to-azure-technical-documentation"></a>Azure 技术文档供稿
我们欢迎社区（用户、客户、合作伙伴、核心 Azure 产品单位外部的 MSFT 员工外部等）以及在核心 Azure 产品单位工作的员工供稿。 供稿的方式取决于供稿人的身份：

* **社区 - 次要更新**：如果供稿人是出于善意提供次要更，可在此存储库中找到相应的文章，或者在 [https://docs.microsoft.com/azure](https://docs.microsoft.com/azure) 上访问该文章，并单击文章中的“编辑”链接转到该文章的 GitHub 源。 然后，只需使用 GitHub UI 进行更新。 我们也欢迎创建存储库的分叉，然后从分叉提交更新。

* **社区 - 新文章**：如果你属于 Azure 社区并想要创建新文章，则需要求助于某个员工合作，以通过合并公共与专用存储库中的工作来发表新内容。

* **员工**：如果你是技术编写员、程序经理或者 Azure 服务产品团队的开发人员，并且职责是技术文章供稿或创作，则应使用专用存储库 (https://github.com/MicrosoftDocs/azure-docs-pr)。 如果要对现有文章进行重大更改、添加或更改图像，或者要补充新文章，则需要派生此存储库，安装 Git Bash（Markdown 编辑器）并了解一些 git 命令。 请参阅[内部供稿人指南](https://review.docs.microsoft.com/en-us/help/contribute/?branch=master)了解详细信息。


## <a name="about-your-contributions-to-azure-content"></a>有关在 Azure 中供稿
### <a name="minor-corrections"></a>轻微更正
[docs.microsoft.com 使用条款](https://docs.microsoft.com/legal/termsofuse)涵盖你在此存储库中提交的、对文档和代码示例所做的轻微更正和澄清。

### <a name="larger-submissions"></a>大型提交件
如果你不是 Microsoft 的员工，当你提交有关新文件或对文档和代码示例进行的重大更改的提取请求时，我们会在 GitHub 中发送一条意见，要求你提交一份在线供稿许可协议 (CLA)。 在我们接受你的提取请求之前，需要你填写这份在线表单。

## <a name="tools-and-setup"></a>工具和设置
社区供稿人可以使用 GitHub UI 或创建存储库的分叉来供稿。 员工应访问[内部供稿人指南](https://review.docs.microsoft.com/en-us/help/contribute/?branch=master)，详细了解如何为技术文档集供稿。

## <a name="repository-organization"></a>存储库组织
azure-docs 存储库中的内容遵循 https://docs.microsoft.com/azure 上的文档组织方式。 此存储库包含两个根文件夹：

### <a name="articles"></a>\articles
*\articles* 文件夹包含采用标记文件格式、扩展名为 *.md* 的文档文章。 文章通常按 Azure 服务分组。

*\articles* 文件夹包含用于保存根目录文章媒体文件的 *\media* 文件夹，其中的子文件夹保存每篇文章的图像。  服务文件夹包含每个服务文件夹中的文章的独立媒体文件夹。 文章图像文件夹的命名与文章文件相同，不过没有 *.md* 扩展名。

### <a name="includes"></a>\includes
可以创建要包含在一篇或多篇文章中的、可重复使用的内容部分。 

## <a name="how-to-use-markdown-to-format-your-topic"></a>如何使用标记来设置主题格式
此存储库中的所有文章使用 GitHub 风格的标记。  资源列表如下。

* [标记基础知识](https://help.github.com/articles/markdown-basics/)
* [可打印标记速查表](https://guides.github.com/pdfs/markdown-cheatsheet-online.pdf)


## <a name="labels"></a>标签
在公共 azure-docs 存储库中，会将自动化标签分配到拉取请求，帮助我们管理拉取请求工作流，并告知拉取请求的处理进度：

* 相关的供稿许可协议
  * cla-not-required：更改相对较为次要，不需要你签署 CLA。
  * cla-required：更改范围相对较大，要求你签署 CLA。
  * cla-signed：供稿人已签署 CLA，因此提取请求现在可以转到审核环节。
* 更改已发送到作者：已通知作者正在等待处理提取请求。
* ready-to-merge：已准备好供拉取请求审阅团队审阅。


