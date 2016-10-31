# <a name="microsoft-graph-error-responses-and-resource-types"></a>Microsoft Graph 错误响应和资源类型

<!--In this article:
  
-   [Status code](#msg_status_code)
-   [Error resource type](#msg_error_resource_type)
-   [Code property](#msg_code_property)

<a name="msg_error_response"> </a> -->

使用标准的 HTTP 状态代码以及 JSON 错误响应对象返回 Microsoft Graph 中的错误。

## <a name="http-status-codes"></a>HTTP 状态代码

下表列出并描述可以返回的 HTTP 状态代码。

| 状态代码 | 状态消息                  | 说明                                                                                                                            |
|:------------|:--------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| 400         | 错误的请求                     | 无法处理请求，因为格式有误或者不正确。                                                                       |
| 401         | 未经授权                    | 资源所需的身份验证信息缺少或无效。                                                   |
| 403         | 禁止访问                       | 对于请求的资源，访问被拒绝。用户可能没有足够的权限。                                                 |
| 404         | 未找到                       | 所请求的资源不存在。                                                                                                  |
| 405         | 方法不允许              | 请求中的 HTTP 方法在资源上不允许。                                                                         |
| 406         | 不接受                  | 该服务不支持“Accept”标头中请求的格式。                                                                |
| 409         | Conflict                        | 当前状态与请求预期的状态的冲突。例如，指定的父文件夹可能不存在。                   |
| 410         | 不存在                            | 所请求的资源在服务器不再可用。                                               |
| 411         | 需要长度                 | 请求上需要 Content-Length 标头。                                                                                    |
| 412         | 前提条件不满足             | 请求中提供的前提条件（例如“If-Match”标头）与资源的当前状态不匹配。                       |
| 413         | 请求实体太大        | 请求的大小超出最大限制。                                                                                            |
| 415         | 媒体类型不受支持          | 请求的内容类型的格式不受服务支持。                                                      |
| 416         | 请求的范围不满足 | 指定的字节范围无效或不可用。                                                                                    |
| 422         | 实体无法处理            | 无法处理请求，因为语义上不正确。                                                                       |
| 429         | 请求过多               | 客户端应用程序已被限制，经过一段时间之后再尝试重复的请求。                |
| 500         | 内部服务器错误           | 处理请求时出现内部服务器错误。                                                                       |
| 501         | 未实现                 | 所请求的功能未实现。                                                                                               |
| 503         | 服务不可用             | 该服务暂时不可用。可以过段时间之后再重复该请求。可能会有 Retry-After 标头。                   |
| 507         | 存储不足            | 已达到最大存储配额。                                                                                            |
| 509         | 超出带宽限制        | 您的应用因超出最大带宽上限而被限制。应用可以在等待一段时间之后再重试该请求。 |

错误响应是单个 JSON 对象，包含名为“**错误**”的单个属性。此对象包括错误的所有详细信息。除了 HTTP 状态代码外，还可以使用此处返回的信息，或者使用此信息替代 HTTP 状态代码。以下是完整的 JSON 错误正文示例。

<!-- { "blockType": "example", "@odata.type": "sample.error", "expectError": true, "name": "example-error-response"} -->
```json
{
  "error": {
    "code": "invalidRange",
    "message": "Uploaded fragment overlaps with existing data.",
    "innerError": {
      "requestId": "request-id",
      "date": "date-time"
    }
  }
}
```

<!--<a name="msg_error_resource_type"> </a> -->

## <a name="error-resource-type"></a>错误的资源类型

只要处理请求的过程中出现错误，就会返回错误资源。

错误响应遵循错误响应 [OData v4](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Toc372793091) 规范中的定义。

### <a name="json-representation"></a>JSON 表示形式

错误资源包括以下资源：

<!-- { "blockType": "resource", "@odata.type": "sample.error" } -->
```json
{
  "error": { "@odata.type": "odata.error" }  
}
```

#### <a name="odata.error-resource-type"></a>odata.error 资源类型

错误响应内是错误资源，其中包括以下属性：

<!-- { "blockType": "resource", "@odata.type": "odata.error", "optionalProperties": [ "target", "details", "innererror"] } -->
```json
{
  "code": "string",
  "message": "string",
  "innererror": { "@odata.type": "odata.error" }
}
```

| 属性名称  | 值                  | 说明\                                                                                               |
|:---------------|:-----------------------|:-----------------------------------------------------------------------------------------------------------|
| **code**       | string                 | 发生的错误的错误代码字符串                                                            |
| **message**    | string                 | 关于发生的错误的开发人员就绪消息。这不应直接显示给用户。 |
| **innererror** | error object           | 可选。可能比顶级错误更具体的其他错误对象。                     |
<!-- {
  "type": "#page.annotation",
  "description": "Understand the error format for the API and error codes.",
  "keywords": "error response, error, error codes, innererror, message, code",
  "section": "documentation",
  "tocPath": "Misc/Error Responses"
} -->

<!--<a name="msg_code_property"> </a> -->

#### <a name="code-property"></a>代码属性

`code` 属性包含下列可能值之一。您的应用应做好准备处理任意这些错误。

| 代码                      | 说明
|:--------------------------|:--------------
| **accessDenied**          | 调用方没有执行该操作的权限。 
| **activityLimitReached**  | 应用或用户已被限制。
| **generalException**      | 发生未指定错误。
| **invalidRange**          | 指定的字节范围无效或不可用。
| **invalidRequest**        | 该请求格式有误或不正确。
| **itemNotFound**          | 找不到资源。
| **malwareDetected**       | 所请求的资源中检测到恶意软件。
| **nameAlreadyExists**     | 指定的项目名称已存在。
| **notAllowed**            | 系统不允许执行此操作。
| **notSupported**          | 系统不支持该请求。
| **resourceModified**      | 正在更新的资源自上次调用方读取时已进行了更改，通常是 eTag 不匹配。
| **resyncRequired**        | 增量令牌将不再有效，并且应用必须重置同步状态。
| **serviceNotAvailable**   | 服务不可用。过段时间后再次尝试请求。可能会有 Retry-After 标头。 
| **quotaLimitReached**     | 用户已达到其配额限制。
| **unauthenticated**       | 调用方未进行身份验证。

`innererror` 对象可能以递归方式包含更多 `innererror` 对象，其中具有其他更多具体的错误代码。处理错误时，应用应遍历所有可用错误代码并使用它们认为最详细的错误代码。在本页底部列出了一些更详细的代码。

若要验证某个错误对象是否是你想要的错误，你必须遍历 `innererror` 对象，查找你想要的错误代码。例如：

```csharp
public bool IsError(string expectedErrorCode)
{
    OneDriveInnerError errorCode = this.Error;
    while (null != errorCode)
    {
        if (errorCode.Code == expectedErrorCode)
            return true;
        errorCode = errorCode.InnerError;
    }
    return false;
}
```

有关介绍如何正确处理错误的示例，请参阅 [错误代码处理](https://gist.github.com/rgregg/a1866be15e685983b441)。

根处的 `message` 属性包含供开发人员阅读的错误消息。错误消息未本地化，并且不应直接向用户显示。处理错误时，代码不应关闭 `message` 值，因为它们随时会更改，并且它们通常包含特定于失败请求的动态信息。只应针对 `code` 属性中返回的错误代码进行编码。

#### <a name="detailed-error-codes"></a>详细的错误代码
以下是你的应用可能会在嵌套的 `innererror` 对象中遇到的一些其他错误。应用不需要处理这些错误，但如果它们选择，也可以处理。服务可能会随时添加新的错误代码或者停止返回的旧代码，因此所有应用都能够处理 [基本错误代码](#code-property) 非常重要。

| 代码                               | 说明
|:-----------------------------------|:----------------------------------------------------------
| **accessRestricted**               | 访问仅限于该项目的所有者。
| **cannotSnapshotTree**             | 未能获取一致的增量快照。请稍后重试。
| **childItemCountExceeded**         | 已达到最大子项目数限制。
| **entityTagDoesNotMatch**          | ETag 与当前项目的值不匹配。
| **fragmentLengthMismatch**         | 为此片段声明的总大小与上载会话的大小不同。
| **fragmentOutOfOrder**             | 已上载的片段无序。
| **fragmentOverlap**                | 上载的片段与现有数据重叠。
| **invalidAcceptType**              | 接受类型无效。
| **invalidParameterFormat**         | 参数格式无效。
| **invalidPath**                    | 名称包含无效字符。
| **invalidQueryOption**             | 查询选项无效。
| **invalidStartIndex**              | 开始索引无效。
| **lockMismatch**                   | 锁定令牌与现有锁定不匹配。
| **lockNotFoundOrAlreadyExpired**   | 该项目上目前没有未过期的锁定。
| **lockOwnerMismatch**              | 锁定所有者 ID 与提供的 ID 不匹配。
| **malformedEntityTag**             | ETag 标头格式不正确。ETags 必须是带引号的字符串。
| **maxDocumentCountExceeded**       | 已达到最大文档数量限制。
| **maxFileSizeExceeded**            | 已超出最大文件大小。
| **maxFolderCountExceeded**         | 已达到最大文件夹数量限制。
| **maxFragmentLengthExceeded**      | 已超出最大文件大小。
| **maxItemCountExceeded**           | 已达到最大项目数量限制。
| **maxQueryLengthExceeded**         | 已超出最大查询长度。
| **maxStreamSizeExceeded**          | 已达到最大流大小。
| **parameterIsTooLong**             | 参数超出最大长度。
| **parameterIsTooSmall**            | 参数小于最小值。
| **pathIsTooLong**                  | 路径超出最大长度。
| **pathTooDeep**                    | 已达到文件夹层次结构深度限制。
| **propertyNotUpdateable**          | 属性无法更新。
| **resyncApplyDifferences**         | 需要重新同步。如果您确定上次同步时服务与您的本地更改保持同步，则请将任意本地项目替换为服务器的版本（包括删除）。上载服务器并不清楚的任意本地更改。
| **resyncRequired**                 | 需要重新同步。
| **resyncUploadDifferences**        | 需要重新同步。上载服务未返回的任意本地项目，并上载与服务器版本不同的任意文件（如果不知道哪个是最新的，就请保留两份）。
| **serviceNotAvailable**            | 服务器无法处理当前的请求。
| **serviceReadOnly**                | 资源暂时是只读的。
| **throttledRequest**               | 请求过多。
| **tooManyResultsRequested**        | 请求的结果过多。
| **tooManyTermsInQuery**            | 查询中的术语过多。
| **totalAffectedItemCountExceeded** | 因为受影响的项目数量超出阈值，所以不允许该操作。
| **truncationNotAllowed**           | 不允许数据截断。
| **uploadSessionFailed**            | 上载会话失败。
| **uploadSessionIncomplete**        | 上载会话未完成。
| **uploadSessionNotFound**          | 上载会话未找到。
| **virusSuspicious**                | 此文档可疑，可能存在病毒。
| **zeroOrFewerResultsRequested**    | 请求的结果为零或更少。

<!-- ##Additional Resources##

- [Microsoft Graph API release notes and known issues](microsoft-graph-api-release-notes-known-issues.md )
- [Hands on lab: Deep dive into the Microsoft Graph API](http://dev.office.com/hands-on-labs/4585) -->
