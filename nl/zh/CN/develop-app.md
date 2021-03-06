---

copyright:
  years: 2015, 2017
lastupdated: "2017-07-24"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# 构建客户机应用程序

您已有了工作对话。现在，您希望开发应用程序，用来与用户进行交互并与 {{site.data.keyword.conversationfull}} 服务进行通信。
{: shortdesc}

本教程中的代码示例是通过 Node.js Watson SDK 使用 JavaScript 编写的，但您也可以使用其他语言。安装适用于您的编程语言的 Watson [ SDK ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://www.ibm.com/watson/developercloud/developer-tools.html){: new_window}，并查看 SDK 文档以获取更多信息。
{: tip}

## 设置 Conversation 服务

我们将在本部分中创建示例应用程序，该应用程序可用于实现认知个人助手的若干功能。应用程序代码将连接到 {{site.data.keyword.conversationshort}} 工作空间，在其中将执行认知处理（例如，用户意向检测）。

继续使用此示例之前，您需要先设置必需的 {{site.data.keyword.conversationshort}} 工作空间：

1.  下载工作空间 <a target="_blank" href="https://watson-developer-cloud.github.io/doc-tutorial-downloads/conversation/conversation-simple-example.json" download="conversation-simple-example.json">JSON 文件</a>。
1.  [导入工作空间](configure-workspace.html#creating-workspaces)，以将工作空间导入到 {{site.data.keyword.conversationshort}} 服务的实例中。

## 获取服务信息

要访问 {{site.data.keyword.conversationshort}} 服务 REST API，应用程序需要能够向 {{site.data.keyword.Bluemix}} 进行认证，并连接到正确的 {{site.data.keyword.conversationshort}} 工作空间。您需要复制服务凭证和工作空间标识，然后将其粘贴到应用程序代码中。

要通过工作空间访问服务凭证和工作空间标识，请选择 ![菜单](images/Menu_16.png) 菜单，选择**部署**，然后转至**凭证**选项卡。

还可以通过 Bluemix“仪表板”来访问服务凭证。

## 与 Conversation 服务通信

与 {{site.data.keyword.conversationshort}} 服务交互非常简单。请看下面的 Node.js 示例，此示例连接到服务，发送一条消息，然后将输出记录到控制台：

```javascript
// 示例 1：设置服务包装器，发送初始消息，
// 接收响应。

var ConversationV1 = require('watson-developer-cloud/conversation/v1');

// 设置 Conversation 服务包装器。
var conversation = new ConversationV1({
  username: 'USERNAME', // replace with username from service key
  password: 'PASSWORD', // replace with password from service key
  path: { workspace_id: 'WORKSPACE_ID' }, // replace with workspace ID
  version_date: '2016-07-11'
});

// 以空消息开始会话。
conversation.message({}, processResponse);

// 处理会话响应。
function processResponse(err, response) {
  if (err) {
    console.error(err); // something went wrong
    return;
  }

  // 显示对话输出（如果有的话）。
  if (response.output.text.length != 0) {
      console.log(response.output.text[0]);
  }
}
```
{: codeblock}

第一步是为 {{site.data.keyword.conversationshort}} 服务创建包装器。

包装器是一个对象，用于向服务发送输入并从服务接收输出。创建服务包装器时，请指定服务密钥中的认证凭证以及使用的 {{site.data.keyword.conversationshort}} API 的版本。

在此 Node.js 示例中，包装器是存储在变量 `conversation` 中的 `ConversationV1` 实例。针对其他语言的 Watson SDK 提供了用于实例化服务包装器的等效机制。

创建服务包装器之后，使用该包装器将消息发送到 {{site.data.keyword.conversationshort}} 服务。在此示例中，消息为空；我们只想触发对话中的 conversation_start 节点，因此不需要任何输入文本。

使用 `node <filename.js>` 命令运行示例应用程序。

**注：**确保已使用 `npm install watson-developer-cloud` 安装 Node.js Watson SDK。

假定一切正常，那么 {{site.data.keyword.conversationshort}} 服务将从对话返回输出，然后将该输出记录到控制台：

```
欢迎使用 Conversation 示例！
```
{: screen}

此输出表明，我们已成功与 {{site.data.keyword.conversationshort}} 服务通信，并收到了对话中 conversation_start 节点指定的欢迎消息。现在，我们可以添加用户界面，以便能处理用户输入。

## 处理用户输入以检测意向

为了能够处理用户输入，我们需要向应用程序添加用户界面。对于此示例，我们简化了工作，使用的是标准输入和输出。可以使用 Node.js prompt-sync 模块来执行此操作。（可以使用 `npm install prompt-sync` 来安装 prompt-sync。）

```javascript
// 示例 2：添加用户输入并检测意向。

var prompt = require('prompt-sync')();
var ConversationV1 = require('watson-developer-cloud/conversation/v1');

// 设置 Conversation 服务包装器。
var conversation = new ConversationV1({
  username: 'USERNAME', // replace with username from service key
  password: 'PASSWORD', // replace with password from service key
  path: { workspace_id: 'WORKSPACE_ID' }, // replace with workspace ID
  version_date: '2016-07-11'
});

// 以空消息开始会话。
conversation.message({}, processResponse);

// 处理会话响应。
function processResponse(err, response) {
  if (err) {
    console.error(err); // something went wrong
    return;
  }

  // 如果检测到意向，就将其记录到控制台。
  if (response.intents.length > 0) {
    console.log('Detected intent: #' + response.intents[0].intent);
  }

  // 显示对话输出（如果有的话）。
  if (response.output.text.length != 0) {
      console.log(response.output.text[0]);
  }

  // 提示下一轮输入。
  var newMessageFromUser = prompt('>> ');
  conversation.message({
    input: { text: newMessageFromUser }
    }, processResponse)
}
```
{: codeblock}

此版本应用程序的开始方式跟之前一样：向 {{site.data.keyword.conversationshort}} 服务发送空消息来启动会话。

现在，`processResponse()` 函数显示对话检测到的任何意向以及输出文本，然后提示您进行下一轮的用户输入。

但有些地方还是有问题：

```
欢迎使用 Conversation 示例！
>> hello
检测到的意向：#hello
欢迎使用 Conversation 示例！
>> what time is it?
检测到的意向：#time
欢迎使用 Conversation 示例！
>> goodbye
检测到的意向：#goodbye
欢迎使用 Conversation 示例！
>>
```
{: screen}

{{site.data.keyword.conversationshort}} 服务在检测正确的意向，但每轮会话都返回 conversation_start 节点的欢迎消息（`欢迎使用 Conversation 示例！`）。

发生这种情况的原因是 {{site.data.keyword.conversationshort}} 服务是无状态服务；维护状态信息是应用程序的责任。由于我们尚未执行任何操作来维护状态，因此 {{site.data.keyword.conversationshort}} 服务将每轮用户输入都视为新会话的第一轮，从而触发 conversation_start 节点。

## 维护状态

会话的状态信息是使用*上下文*进行维护的。上下文是应用程序与 {{site.data.keyword.conversationshort}} 服务之间来回传递的 JSON 对象。从一轮会话转至下一轮会话时，应用程序负责维护上下文。

上下文中包含每个用户会话的唯一标识以及随着每轮会话递增的计数器。我们先前版本的示例未保留上下文，这意味着每轮输入看起来都是新会话的开始。我们可以通过保存上下文并每次将其发送回 {{site.data.keyword.conversationshort}} 服务来解决此问题。

上下文不但可以帮助我们保持在会话中的位置，还可以用于存储希望在应用程序和 {{site.data.keyword.conversationshort}} 服务之间来回传递的其他任何数据。这些数据可能包括要在整个会话中维护的持久数据（例如，客户姓名或帐号），或者希望跟踪的其他任何数据（例如，选项设置的当前状态）。

```javascript
// 示例 3：维护状态。

var prompt = require('prompt-sync')();
var ConversationV1 = require('watson-developer-cloud/conversation/v1');

// 设置 Conversation 服务。
var conversation = new ConversationV1({
  username: 'USERNAME', // replace with username from service key
  password: 'PASSWORD', // replace with password from service key
  path: { workspace_id: 'WORKSPACE_ID' }, // replace with workspace ID
  version_date: '2016-07-11'
});

// 以空消息开始会话。
conversation.message({}, processResponse);

// 处理会话响应。
function processResponse(err, response) {
  if (err) {
    console.error(err); // something went wrong
    return;
  }

  // 如果检测到意向，就将其记录到控制台。
  if (response.intents.length > 0) {
    console.log('Detected intent: #' + response.intents[0].intent);
  }

  // 显示对话输出（如果有的话）。
  if (response.output.text.length != 0) {
      console.log(response.output.text[0]);
  }

  // 提示下一轮输入。
  var newMessageFromUser = prompt('>> ');
    // Send back the context to maintain state.
    conversation.message({
      input: { text: newMessageFromUser },
      context : response.context,
    }, processResponse)
}
```
{: codeblock}

与先前示例唯一不同的是，现在对于每轮会话，都会将在上一轮中接收到的 `response.context` 对象发送回去：

```javascript
    conversation.message({
      input: { text: newMessageFromUser },
      context : response.context,
    }, processResponse)
```
{: codeblock}

这样可以确保从一轮会话转至下一轮时维护上下文，这样 {{site.data.keyword.conversationshort}} 服务就不会再认为每一轮都是第一轮：

```
>> hello
检测到的意向：#hello
您好。
>> what time is it?
检测到的意向：#time
>> goodbye
检测到的意向：#goodbye
好的，再会。
>>
```
{: screen}

现在我们已经取得了进步！{{site.data.keyword.conversationshort}} 服务将正确识别我们的意向，并且对话将返回每个意向的正确输出文本（如果提供）。

但是，别的什么也没发生。我们请求返回时间，但没有回答；我们说再见，会话也没有结束。这是因为这些意向需要应用程序执行其他操作。

## 实现应用程序操作

除了要向用户显示的输出文本之外，对话还会使用响应 JSON 中的 `output` 对象，在应用程序需要执行操作时，根据检测到的意向发出信号。

这些操作标志会使用 `action` 属性发送，对话将该属性定义为响应 JSON 的一部分。对话确定应用程序需要执行某些操作时，会将 `action` 的值设置为相应的值：`display_time` 或 `end_conversation`。

请记住，`output` 只是一个 JSON 对象，您可以向其添加所需的任何有效内容。对于较复杂的应用程序，可以使用具有多个操作标志的数组。

但是，在我们的示例中，我们将使用支持单一操作标志的简单键/值对。我们的应用程序代码需要检查响应中 `action` 属性的值，然后执行任何指定的操作。（此版本也不会显示检测到的意向，因为我们确信可以正确识别这些意向。）

```javascript
// 示例 4：实现应用程序操作。

var prompt = require('prompt-sync')();
var ConversationV1 = require('watson-developer-cloud/conversation/v1');

// 设置 Conversation 服务。
var conversation = new ConversationV1({
  username: 'USERNAME', // replace with username from service key
  password: 'PASSWORD', // replace with password from service key
  path: { workspace_id: 'WORKSPACE_ID' }, // replace with workspace ID
  version_date: '2016-07-11'
});

// 以空消息开始会话。
conversation.message({}, processResponse);

// 处理会话响应。
function processResponse(err, response) {
  if (err) {
    console.error(err); // something went wrong
    return;
  }

  var endConversation = false;

  // 检查操作标志。
  if (response.output.action === 'display_time') {
    // 用户询问现在几点，所以我们输出本地系统时间。
    console.log('The current time is ' + new Date().toLocaleTimeString());
  } else if (response.output.action === 'end_conversation') {
    // 用户说再见，所以我们结束。
    console.log(response.output.text[0]);
    endConversation = true;
  } else {
    // 显示对话输出（如果有的话）。
  if (response.output.text.length != 0) {
        console.log(response.output.text[0]);
    }
  }

  // 如果未结束，那么提示进行下一轮输入。
  if (!endConversation) {
    var newMessageFromUser = prompt('>> ');
    conversation.message({
      input: { text: newMessageFromUser },
      // 将上下文发送回来以维护状态。
      context : response.context,
    }, processResponse)
  }
}
```
{: codeblock}

现在，processResponse() 函数会检查从 {{site.data.keyword.conversationshort}} 服务接收到的 `output` 对象的 `action` 属性值。如果值为 `display_time` 或 `end_conversation`，那么应用程序会执行相应的操作。

```
欢迎使用 Conversation 示例！
>> hello
您好。
>> what time is it?
当前时间是晚上 12:40:42。
>> goodbye
好的，再会。
```
{: screen}

成功！现在，应用程序可使用 {{site.data.keyword.conversationshort}} 服务来识别自然语言输入中的意向，显示相应的响应，并实现所请求的操作。

当然，现实世界应用程序将使用更复杂的用户界面，例如 Web 交谈窗口。它将实现更复杂的操作，可能是与客户数据库或其他业务系统相集成。但是，应用程序与 {{site.data.keyword.conversationshort}} 服务进行交互的基本原理保持不变。

有关更复杂的示例，请查看“导航”窗格中的“样本应用程序”。
