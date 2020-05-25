二、插件
========

一、插件入门
------------

​
插件可以通过添加其他命令和定位器，在测试运行之前和之后引导安装程序以及影响录
制过程来扩展 Selenium IDE 的默认行为。

​ Selenium IDE 正在使用 WebExtension
标准在现代浏览器中运行(要了解更多信息， 请查看 Mozilla 的\ `Your Your
Extension <https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Your_first_WebExtension>`__
文章)。扩展之间的通信是通过\ `外部消息传递协议 <https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/runtime/sendMessage>`__\ 处理的，您可以\ `在此处 <https://github.com/SeleniumHQ/selenium-ide/tree/v3/packages/extension-boilerplate>`__\ 查看示例。

​ 本文假定您具有 WebExtension 开发方面的知识，并且仅讨论 Selenium IDE
特定的功能。

1.1 调用API
~~~~~~~~~~~

​ 可以使用命令 browser.runtime.sendMessage 来调用 Selenium IDE API。

​ browser.runtime.sendMessage(SIDE_ID,request)是一个签名的例子，其中
SIDE_ID 指的是 IDE 的官方推广 ID，这部分内容可以查看 1.2 节 Selenium IDE
拓展 ID。

1.2 请求
~~~~~~~~

​ 该请求是命令 browser.runtime.sendMessage 的第二个参数，其思想与 HTTP
类 似。

::

   {
   uri: "/register",
   verb: "post",
   payload: {
   name: "Selenium IDE plugin",
   version: "1.0.0" }
   }

-  uri -IDE 功能的资源定位器(例如，记录命令，解析定位器)
-  verb-修饰符功能(例如，get 获取内容，post 添加新内容，就像在 http
   中一样)

​ Selenium IDE 会回复有效的响应，如果出现错误，可以通过打开 IDE 窗口的
DevTools进行查看。

::

   browser.runtime.sendMessage(SIDE_ID, request).then(response => { console.log("it worked!");
   });

1.3 清单
~~~~~~~~

​ 插件为 Selenium IDE 提供了清单，该清单声明了它们对 Selenium IDE
功能的更 改和添加。

::

   {
     name: "New Plugin",
     version: "1.0.0",
     commands: [
       {
         id: "newCommand",
         name: "new command",
         type: "locator",
         docs: {
           description: "command description",
           target: { name: "command target", value: "command target description" },
           value: { name: "command value", value: "command value description" }
         }
       },
       {
         id: "anotherCommand",
         name: "another command",
         type: "locator",
         docs: {
           description: "another command description",
           target: "locator",
           value: "pattern"
         }
       }
     ],
     locators: [
       {
         id: "locator"
       }
     ],
     dependencies: {
       "selenium-webdriver": "3.6.0"
     }
   }

1.4 一般信息
~~~~~~~~~~~~

-  name -必需，插件名称。
-  version -必需，插件版本。

1.5 命令
~~~~~~~~

​ 要添加到 Selenium IDE 的新命令的列表，每个命令带有一些参数:

-  id -必需，该命令的 camelCase 唯一标识符。
-  name -必需，命令的自然语言名称，用户将看到此名称。
-  type-可选，可以是 locator 或 region，用于启用 find 和 select 按钮。
   (注意: type 仍处于 beta 中，将来可能会更改)。
-  docs-可选，命令描述，目标和值的元数据集合。或者，可以通过将现有命令
   目标或值的名称指定为字符串(而不是子集合)来使用现有命令目标或值(即
   ArgType)。参阅 ArgTypes 中 ArgTypes.js 的完整列表。

1.6 定位器
~~~~~~~~~~

​ 要添加到 Selenium IDE 中新定位器的列表，每个定位器只需使用 id。

-  id-需要，用于定位符的唯一标识符，将被显示给用户(例如 name，css)。

1.7 依存关系
~~~~~~~~~~~~

​ 使用命令行(请参见 1.2 节命令行运行器)运行程序运行时要下载和使用的其他
Node.js 依赖项。

​ 依赖项是 key:value 诸如此类的字典 name:version，其中 name 是 npm
上的发布 名称，version 是发布到 npm 的有效 semver。

1.8 注册插件
~~~~~~~~~~~~

​ 要向 Selenium IDE 注册插件，请确保 Selenium IDE
窗口已打开，并且您使用的 是正确的 Selenium IDE 扩展 ID(参见 2.2 节)。

发送以下消息:

::

   browser.runtime.sendMessage(process.env.SIDE_ID, {
     uri: "/register",
     verb: "post",
     payload: {
       name: "Selenium IDE plugin",
       version: "1.0.0",
       commands: [
         {
           id: "successfulCommand",
           name: "successful command"
         },
         {
           id: "failCommand",
           name: "failed command"
         }
       ]
     }
   }).catch(console.error);

二、Selenium IDE扩展ID
----------------------

​ 同时发布到 Chrome WebStore 和 Firefox 附加组件的官方 Selenium IDE
具有恒定的 ID。该 ID 用于通过第三方插件与 Selenium IDE 进行通信。

-  Chrome: mooikfkahbdckldjjndioackbalphokd
-  Firefox: {a6fd85ed-e919-4a43-a5af-8da18bda539f}

三、插件运行状况检查
--------------------

​ 扩展窗口打开时，插件只能在 Selenium IDE 中注册。

​ 在此之前注册将产生错误提示 Selenium IDE is not active。

​ 为了解决这个问题，您可以向系统 API 的运行状况检查发送一条消息。

3.1 运行请求
~~~~~~~~~~~~

::

   {
   uri: "health", 
   verb: "get"
   }

3.2 运行响应
~~~~~~~~~~~~

-  error -Selenium IDE-处于非活动状态，或者未安装。

-  true -您的插件已经注册，可以接受请求。

-  false-您的插件尚未注册，应发送[[registration \| Getting Started with

   Plugins#registering-the-plugin]]请求。

3.3 轮询运行状况检查
~~~~~~~~~~~~~~~~~~~~

​ 您可以使用此运行状况检查机制在 Selenium IDE
处于活动状态时引入轮询并进行注册。

​ 您甚至应该继续轮询 Selenium IDE，因为用户可以关闭 Selenium IDE
的窗口。

::

   let interval;

   export function sendMessage(payload) {
     return browser.runtime.sendMessage(SIDE_ID, payload);
   }

   export function startPolling(payload, cb) {
     interval = setInterval(() => {
       sendMessage({
         uri: "/health",
         verb: "get"
       }).catch(res => ({error: res.message})).then(res => {
         if (!res) {
           sendMessage({
             uri: "/register",
             verb: "post",
             payload
           }).then(() => {
             console.log("registered");
             cb();
           });
         } else if (res.error) {
           cb(new Error(res.error));
         }
       });
     }, 1000);
   }

   export function stopPolling() {
     clearInterval(interval);
   }

​ 这样，您可以每秒重新尝试连接到 Selenium IDE，如果 Selenium IDE
窗口关闭，则在 一秒钟之内您将收到一个回调通知。

四、发送和接收请求
------------------

​ Selenium IDE 中发送和接收请求的的灵感来自于 HTTP 的消息传递。

​ 但是，通过它进行消息传递的方法略有不同。

4.1 面向Selenium IDE的请求
~~~~~~~~~~~~~~~~~~~~~~~~~~

​ 面向 Selenium IDE 的请求是带有特定键值的 JSON 对象，这些键值决定了在
Selenium IDE 中的请求方式和内容。

::

   {
     uri: "/path/to/resource",
     verb: "get",
     payload: {
       data: "request body goes here"
     }
   }

-  uri -IDE 功能的资源定位器(例如，记录命令，解析定位器)
-  verb-修饰符功能(例如，get 获取内容，post 添加新内容，就像在 http 中一
   样)
-  payload-请求主体，执行操作所需的信息，从 uri 变为 uri

4.2 发送请求
~~~~~~~~~~~~

​ 如果出现错误，Selenium IDE 会以有效的响应进行回复，这可以通过打开 IDE
窗口的 DevTools 查看。

::

   browser.runtime.sendMessage(SIDE_ID, request).then(response => {
     console.log("it worked!");
   });

4.3 来自Selenium IDE的请求
~~~~~~~~~~~~~~~~~~~~~~~~~~

​ 来自 Selenium IDE 的请求在键和结构上有所不同，IDE
具有一个负责嵌套路由的 路由器(例如 uri:
/path/to/nested/uri)。通过采取不同的方法来避免开发人
员开发或实现他们自己的路由。

::

   {
     action || event: "an action to perform or an event to adhere",
     request keys...
   }

-  action或event-要执行的动作或要响应的事件，动作可以是执行命令或发送其代码，而事件可以是录制、回放开始或结束。
-  additional keys-通过执行 action 或 event 来确定其他键。 注意:仅会定义
   action 或 event，而不会两者都定义。

4.4 接收请求
~~~~~~~~~~~~

​ 要接收请求，您必须实现 browser.runtime.onMessageExternal 这个类。

::

   browser.runtime.onMessageExternal.addListener((message, sender, sendResponse) => {
       if (message.action === "execute" && message.command) {
         console.log("I need to execute a command");
         return sendResponse(true); // I've finished execution
       }
       if (message.event === "playbackStarted") {
         console.log("IDE notified me a playback was started"); // Responding to events is optional
       }
   });

4.5 异步请求
~~~~~~~~~~~~

​ 当我们需要等待一个 Promise 或执行一个更改 DOM
的命令时，某些请求实际上是 异步的。

​ Selenium IDE 必须被通知等待，它将等待直到 sendResponse
被调用为止。为了防止 Selenium IDE
永远等待，请确保在失败的情况下将错误(请参阅 2.5.1 节)返回 给 IDE。

​ 为了让 Selenium IDE 等待，在 onMessageExternal 事件处理程序中返回
true，并 使用 sendResponse 返回最终结果。

::

   browser.runtime.onMessageExternal.addListener((message, sender, sendResponse) => {
       if (message.action === "execute" && message.command && message.command.command === "myAsyncCommand") {
         executingSomeAsyncFunctionality(message.command).then(() => {
           return sendResponse(true);
         });
         return true;
       }
   });

五、错误处理
------------

​
如果在执行命令时遇到故障，并且由于该原因而导致需要使用的测试用例失败，则必须
对 IDE 做出响应。

​ 由于 JSON 不支持错误序列化，因此 Selenium IDE 制定了一个标准。

5.1 错误
~~~~~~~~

​ 错误对象是普通的 JavaScript 对象，将其发送到 Seleni IDE
时，会被解析为错误。

::

   {
     status: "fatal",
     error: "This command can't be run individually, please run the test case."
   }

-  status-选项，可以是 undefined 或 fatal，致命错误将使测试失败，非致命
   错误将继续执行，对 verify 命令很有用。
-  error -必需，消息要打印给用户。

5.2 发送错误
~~~~~~~~~~~~

​ 在执行过程中遇到错误时，您可以使用 sendResponse 对错误对象进行应答。

::

   browser.runtime.onMessageExternal.addListener((message, sender, sendResponse) => {
       if (message.action === "execute" && message.command && message.command.command === "myFailingCommand") {
         executingSomeFunctionalityThatWillEventuallyFail(message.command).catch((error) => {
           return sendResponse({ error: error.message, status: "fatal" });
         });
         return true;
       }
   });

六、触发代码
------------

​ Selenium IDE 有两个主要组件，在浏览器内回放，由 actions 和 events
提供支持。 并使用命令行运行器以命令行模式回放。

6.1 Selenium SIDE Runner 环境
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

​ 运行程序基于 Node，您可以利用环境发布更好的代码。

-  使用 npm 的 Node.js 8 或更高版本
-  Jest
-  Jest-environment-selenium
-  Selenium-webdriver

6.2 触发代码
~~~~~~~~~~~~

​
触发代码时，必须注意某些要点，因为您的插件不是唯一触发代码的插件，也不是控制
触发代码的插件。

​ 为了确保插件不会干扰彼此的执行，必须采取某些预防措施。

6.3 永远不使用return
~~~~~~~~~~~~~~~~~~~~

​ 关键字 return 意味着您之后的代码将永远无法访问，您可能会阻止其他插件。

::

   return somePromise();
   plugin2Func(); // unreachable

​ 相反，由于我们使用的是 Node 8 或更高版本，因此我们可以利用异步功能。

::

   await somePromise();
   plugin2Func(); //works

6.4 不要在全局范围内定义变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

​ 在全局范围内定义变量意味着，如果您和另一个插件或 Selenium IDE
本身定义了相
同的变量，则可能会发生错误或未知的副作用，这将使调试变得困难。

​ 例如，使用以下测试用例:

-  store\| button|element
-  plugin click \| button
-  assert element present \| css=${element}

如果定义变量，代码将如下所示

::

   let element = "button";
   let element = await driver.findElement();
   await element.click();
   expect(element).toBePresent(); // different button!

为避免在全局范围内定义变量，请使用 Promise 的 then 函数。

::

   let element = "button";
   await driver.findElement().then(element => {
     return element.click();
   });
   expect(element).toBePresent(); // the store defined button

6.5 总结
~~~~~~~~

​ 一般来说，尽量避免弄乱全局作用域，如果您需要定义，可以始终使用 Promise
的
then函数，更糟糕的情况下可以使用\ `iife <https://developer.mozilla.org/en-US/docs/Glossary/IIFE>`__
。

七、代码导出
------------

​ 插件 API 允许任何 Selenium IDE 插件导出以下任一代码:

​ 1.现有语言

​ 2.一种新的语言

​ 注意:如果添加新语言，请查看[@ seleniumhq /
side-utils](https://www.npmjs.com/package/@seleniumhq/side-utils)。您可以看到在现有语言中用于代码导出的示例(例如\ `java-junit <https://github.com/SeleniumHQ/selenium-ide/tree/v3/packages/code-export-java-junit>`__)。

7.1 导出配置
~~~~~~~~~~~~

​ 在您的插件清单中，您需要指定其导出语言。

7.1.1 添加到现有语言
^^^^^^^^^^^^^^^^^^^^

​ 要扩充现有语言，请使用 languages 键并指定数组中的语言。

::

   "exports": {
       "languages": ["java-junit"]
     }

​ 目 前 可 用 的 语 言 ID 为 “java-junit” ， “javascript-mocha” ，
“python-pytest”，和“csharp-xunit”。

7.1.2 新增语言
^^^^^^^^^^^^^^

​ 要将新语言添加到代码导出中，请使用 vendor 键并在对象数组中指定语言。

::

    "exports": {
       "vendor": [{"your-language": "Your language"}]
     }

​ 关键字是将在导出事件中使用的 ID。该值是将在 UI
的代码导出菜单中使用的显示名称。

7.2 导出事件
~~~~~~~~~~~~

​ Selenium IDE
为每种实体类型发送以下事件，您的插件可以响应以下事件以进行代码导出。

::

   {
     action: "export",
     entity,
     language,
     options,
   }

-  action- export，指示需要代码导出的操作
-  entity，在导出语言时，要导出的实体可以是
   command，下一节提到的任意钩子，或者 vendor
-  options
   -元数据可帮助您的插件对导出内容做出更明智的决定(例如，项目，测试用例名称，测试用例集名称等)

7.3 钩子
~~~~~~~~

​
代码导出是围绕钩子的概念构建的，它提供了进入要导出的测试代码各个部分的入口点。

-  afterAll (所有测试完成后)
-  afterEach(完成每个测试后-在 afterAll 之前)
-  beforeAll (在运行所有测试之前)
-  beforeEach(在运行每个测试之前-在 beforeAll 之后)
-  command (为插件添加的新命令送代码)
-  dependency (添加其他语言依赖性)
-  inEachBegin (在每个测试的开始阶段)
-  inEachEnd (在每个测试的末尾)
-  variable (声明将在整个测试用例集中使用的新变量)

7.4 响应
~~~~~~~~

.. _添加到现有语言-1:

7.4.1 添加到现有语言
^^^^^^^^^^^^^^^^^^^^

​ 要响应导出事件，将 sendResponse
与您要导出的字符串一起调用。如果您的字符串
有多行，请用换行符(例如:raw-latex:`\n`)将它们分开。或者，您可以研究使用在代码导出入门中提
到的命令对象结构。

::

   sendResponse(`const myLibrary = require("my-library");`);

.. _新增语言-1:

7.4.2 新增语言
^^^^^^^^^^^^^^

​ 为了您的新语言的导出事件响应 vendor，您的插件需要使用包含键 filename

和 body 的对象进行响应。

::

   const payload = {
     filename: 'test.js',
     body: '// your final exported code\n// goes here\n// etc.'
   }
   sendResponse(payload)

八、添加命令
------------

​ 首先要向 Selenium IDE 添加命令，请确保在清单(请参见 1.3 节)中声明它。

声明该命令后，Selenium IDE 将等待您响应其执行和发出命令的请求。

8.1 执行命令
~~~~~~~~~~~~

​ 一旦您的回放命令提交后，您将收到执行命令的请求。

::

   {
     action: "execute",
     command: {
       command: "commandId",
       target: "target specified",
       value: "value specified"
     },
     options: {
       runId: "unique identifier",
       testId: "test identifier",
       frameId: "the frame context",
       tabId: "the tab context",
       windowId: "the window context"
     }
   }

-  action-execute,此操作需要您执行命令。

.. _命令-1:

8.2 命令
~~~~~~~~

​ 用户在表中输入的命令详细信息:

-  command -清单中的命令 ID。
-  target -已填充目标(如果您的命令没有目标，则可以忽略它)。
-  value -填充的值(如果您的命令不使用值，则可以忽略它)。

8.3 选项
~~~~~~~~

​ 回放选项，详细说明执行环境:

-  runId
   -可选的，当前测试用例运行的唯一标识符，可用于推断在同一运行期间执行的某些命令。
-  testId
   -测试用例的永久唯一标识符，即使测试用例的内容可能更改，也会始终发送相同的标识符。
-  frameId- integer 或 0 \|\| undefined 指示应在哪个框架下执行命令，0 或
   undefined 指示应在 window 级别上执行命令。
-  tabId -用于在其中执行命令的标签。
-  windowId -包含受测试标签的窗口。

​ 注意:如果用户双击一个命令以单独执行该命令，则不会生成一个
runId，这可用于确定它 是运行单个命令还是运行一个完整的测试用例。

8.4 将结果发送回 IDE
~~~~~~~~~~~~~~~~~~~~

​ 要将有效的响应发送回 Selenium IDE，首先必须确定希望 IDE 显示的 4
种可能结果中 的哪一种。

结果一：通过

​ 您的命令已通过。

::

    sendResponse(true);

结果二：未确定

​ 如果您的命令已完成，但无法可靠地确定它是否已通过。

​ 要在以后的阶段更新命令状态，请使用 Playback API。

::

   sendResponse({ status: "undetermined" });

结果三：软失效

​ 您的命令已失败，但是测试用例可以在此之后继续执行(例如，verify 命令)。

::

   sendResponse({ error: "detailed error to show the user" });

结果四：硬失效

​ 您的命令失败，导致测试无法继续(例如 click)。

::

   sendResponse({ error: "detailed error to show the user", status: "fatal" });

8.5 触发命令
~~~~~~~~~~~~

​ 为了使用命令行运行器运行测试用例，命令还必须提供 JavaScript
代码片段来执行 它。不提供它会导致用户在尝试保存测试用例时出错。

​ 在实现命令的代码触发之前，了解触发如何工作的一般概念将很有用。

​
本文仅与命令的测试代码有关，如果您的插件需要添加设置和拆解代码，请参阅触发设
置和拆解代码。

​ Selenium IDE 将发送一个请求，以发出您已在清单中注册的命令。

::

   {
     action: "emit",
     entity: "command",
     command: {
       command: "commandId",
       target: "target specified",
       value: "value specified"
     }
   }

-  action- emit，此操作指示必须发出代码。
-  entity- command，此实体指示要发出命令。

.. _命令-2:

8.6 命令
~~~~~~~~

​ 用户在表中输入的命令详细信息:

-  command -清单中的命令 ID(不是友好名称)。
-  target -已填充目标(如果您的命令没有目标，则可以忽略它)。
-  value -填充的值(如果您的命令不使用值，则可以忽略它)。

8.7 返回触发的代码
~~~~~~~~~~~~~~~~~~

​ 完成创建 JavaScript 字符串以执行命令后，将其发送回 IDE。

​ 不要使用 return 关键字，如果您需要等待 Promise 使用 await，return
将导致测 试提前退出。

​ 注意:不要美化代码或插入新行，IDE 会美化它。

::

   sendResponse(`driver.findElement().then((element) => {element.perform...});`);

​ 您可以在 Selenium IDE 的内部触发模块中查看大量示例。

九、Selenium IDE 事件
---------------------

​ Selenium IDE
会在整个使用过程中发送事件，以通知插件回放状态或录制状态。
事件请求(请参见 第四节)与动作请求非常相似。

9.1 事件请求
~~~~~~~~~~~~

::

   {
     action: "event",
     event: "recordingStarted",
     options: {
       event specific keys...
     }
   }

-  action- event 表示要采取的措施是事件。
-  event -该事件的唯一标识符。
-  options -包含有关该事件信息的 JavaScript 对象。

9.2 响应事件
~~~~~~~~~~~~

​ 有些事件仅是通知，这意味着 Selenium IDE
不会让您有机会停止它来进行计算，而 在某些 Selenium IDE
可以等待时，请参考事件列表(请参见 9.3 节)。

​ 响应事件就像任何请求一样，请查看”接收请求”(请参见 4.4 节)。

​ 在应对事件时要注意一些陷阱。有关详细信息，请参阅响应事件时的边缘情况。

9.3 事件列表
~~~~~~~~~~~~

​ Selenium IDE 发出的事件列表。

9.3.1 系统事件
^^^^^^^^^^^^^^

projectLoaded

每次用户加载新项目文件时弹出的事件。

注意: Selenium IDE 不会等待此事件。

选项:

-  projectName -加载的项目的名称
-  projectId -加载的项目的 ID

9.3.2 记录事件
^^^^^^^^^^^^^^

recordingStarted\ **,**\ recordingStopped

每次用户开始或结束记录其动作时都会弹出的事件。

注意: Selenium IDE 不会等待此事件。

选项:

-  testName -记录命令的测试。

commandRecorded

记录命令时弹出的事件。

注意: Selenium IDE 将等待此事件。

选项:

-  tabId -记录命令的选项卡的选项卡 ID。
-  command -记录的命令。
-  target -记录的目标。
-  targets -所有记录的目标及其策略的可选列表。
-  value -记录的值。

9.3.3 回放事件
^^^^^^^^^^^^^^

playbackStarted,playbackStopped

测试用例开始或完成执行时弹出的事件。

注意: Selenium IDE 将等待这些事件。

选项:

-  runId -此测试运行的唯一标识符。
-  testId -此测试用例的唯一标识符(在不同的运行之间持续存在)。
-  testName -运行测试的名称。
-  suiteName -可选，运行套件的名称(仅当作为套件的一部分运行时定义)。
-  projectName -当前项目的名称。

suitePlaybackStarted

当测试用例集开始执行时弹出的事件。

注意: Selenium IDE 将等待此事件。

选项：

-  runId -此测试运行的唯一标识符。
-  suiteName -正在运行的测试用例集的名称。
-  projectName -当前项目的名称。

suitePlaybackStopped

测试用例集执行完毕时弹出的事件。

注意: Selenium IDE 不会等待此事件。

选项：

-  runId -此测试用例运行的唯一标识符。
-  suiteName - 正在运行的测试用例集的名称。
-  projectName -当前项目的名称。

​ Selenium IDE 不会等待 stop 事件(不同于 start
事件)，这是为了防止用户感觉 IDE
被冻结，您仍然可以运行您的拆卸代码，因为后续的测试运行会有所不同 runId。

​ 注意:除了正常的测试用例事件外，套件事件还将弹出。

9.4 应对事件时的边缘案例
~~~~~~~~~~~~~~~~~~~~~~~~

9.4.1 空响应事件
^^^^^^^^^^^^^^^^

​ 如果您的插件响应事件，并且代码中没有操作，请确保
sendResponse(undefined) 在这种情况下进行操作。

9.4.2 用异步代码响应事件
^^^^^^^^^^^^^^^^^^^^^^^^

​ 如果您的插件响应事件并执行异步代码，请确保 return true
不在异步闭包之外。

​ 有关详细信息，请参见4.5节异步请求。

十、触发设置和拆解代码
----------------------

​ 如果使用某些命令，则可能需要其他设置和拆解。

​ 例如，用于计算重定向次数并在测试结束时输出金额的命令。

​ 在浏览器中运行时，可以使用回放事件(请参见
9.3.3节)，但是在使用运行器执行时，需要额外的步骤来实现。

​ 在实施设置和拆解代码触发之前，了解触发工作原理是一个好主意(请参见 第六
节)。

10.1 测试架构
~~~~~~~~~~~~~

​ 运行器在底层使用 jest 进行测试。

​ 测试一般看起来像这样

::

   // config emission
   describe("suite name", () => {
     beforeAll(async () => {
       // suite before all emission
     });
     beforeEach(async () => {
       // suite before each emission
     });
     afterEach(async () => {
       // suite after each emission
     });
     afterAll(async () => {
       // suite after all emission
     });
     it("test name", async () => {
       // test setup
       driver.doStuff();
       ....
       expect();
       // test teardown
     });
   });

在上述每个注释处，都可以插入代码作为设置和拆解的一部分。

10.2 触发配置
~~~~~~~~~~~~~

​
使用配置发出请求将代码发送到测试文件的顶部，用于对不同模块进行全局变量或
require 的设置。

::

   {
     action: "emit",
     entity: "config",
     project: {
       name: "project name"
       tests: []
     }
   }

-  action- emit，指示需要发出代码的动作。
-  entity- config，要发送的实体，配置代码位于测试文件的顶部.
-  Project -导出项目的数据。

   响应:

::

   sendResponse(`const myLibrary = require("my-library");`);

10.3 测试用例触发
~~~~~~~~~~~~~~~~~

​ 为了始终使用测试用例触发，它使用了内置的 jest 方法，包括
beforeAll，beforeEach， afterAll 和 afterEach。

::

   {
     action: "emit",
     entity: "suite",
     suite: {
       name: "suite name"
       tests: []
     }
   }

-  action- emit，指示需要触发代码的动作。
-  entity- suite 触发代码的实体，测试用例代码可以在每次测试之前或之后进
   行，也可以在所有测试之前和之后进行一次。
-  suite -导出测试用例集的数据。

响应：

::

   sendResponse({
     beforeAll: "this will run once before all tests",
     beforeEach: "this will run before every test",
     afterEach: "this will run after every test",
     afterAll: "this will run after all tests"
   });

10.4 测试触发
~~~~~~~~~~~~~

​ 不建议使用此方法，因为它将在测试用例 it 函数内部触发代码。

​
如果在所有可能的情况下使用该测试用例集的如果在所有可能的情况下使用该套件的
beforeEach 和 afterEach 级别，那么测试指标(测试性能等)也会受到影响。

::

   {
     action: "emit",
     entity: "test",
     test: {
       id: "unique test identifier",
       name: "test name",
       commands: [
         command: "commandId",
         target: "specified target",
         value: "specified value"
       ]
     }
   }

-  action- emit，指示需要发送代码的动作。
-  entity- suite
   触发代码的实体，测试用例集代码可以在每次测试之前或之后进
   行，也可以在所有测试之前和之后进行一次。
-  test -导出的测试数据，标识符，名称和命令列表。

响应：

::

   sendResponse({
     setup: "this will run at the beginning of the test",
     teardown: "this will run at the end of the test"
   });

十一、插件API简介
-----------------

​ 尽管 Selenium IDE
可以向您发送执行任务的请求(执行命令或发出命令)，但您也可 以要求 Selenium
IDE 执行任务。

​ Selenium IDE 实现了类似 HTTP
的消息传递协议。有关其工作原理的一般概述，请参 阅 1.1 节调用 API。

11.1 API 的结构
~~~~~~~~~~~~~~~

​ 该 API 按功能域(例如，回放，记录等)构造。

11.1.1 版本控制
^^^^^^^^^^^^^^^

​ uriAPI 中的 s 始于/，并已进行版本控制，当前有效版本为 1，调用 uri
无版本的表示最 新版本。

-  /register -注册功能的最新版本。
-  /v1/register -注册 v1

11.1.2 verbs
^^^^^^^^^^^^

​ API 支持 HTTP 等动词(get，post，delete，put)。

​ 每个人确定资源上的不同功能。

-  get -获取资源或有关它的信息。
-  post -创建新资源。
-  put -更新资源。
-  delete -删除资源。

11.1.3 Errors
^^^^^^^^^^^^^

​ 如果在发送请求时关闭了窗口，则 Selenium IDE 将不会响应，并且 Promise
将拒绝。

​ 或者，如果 Selenium IDE 已打开，则它可以成功将解决
Promise，或者将“用户区” 错误传回给您，因为错误无法序列化。

11.1.4 Connection error
^^^^^^^^^^^^^^^^^^^^^^^

​ 关闭 Selenium IDE 窗口时将发生连接错误，因此 promise 将被拒绝。

::

   browser.runtime.sendMessage(SIDE_ID, payload).catch((error) => {
     console.error(error); // connection error
   });

11.1.5 Request error
^^^^^^^^^^^^^^^^^^^^

​ 当请求无效时(例如，对不存在的资源的请求，例如
fetch)，将发生请求错误。这些 请求解决附有他们的错误的 promise。

::

   browser.runtime.sendMessage(SIDE_ID, payload).then((response) => {
     if (response.error) {
       console.error(response.error); // request error
     }
   });

11.1.6 Successful Request
^^^^^^^^^^^^^^^^^^^^^^^^^

​ 成功的请求是在响应对象上未定义 error
的请求。每个端点都有自己的响应，如果成功， 大多数 POST 请求都会以 TRUE
响应。

::

   browser.runtime.sendMessage(SIDE_ID, payload).then((response) => {
     if (!response.error) {
       console.error(response); // true
     }
   });

十二、插件系统API
-----------------

​ 系统 API 是 Selenium IDE 提供的最基本的 API。它没有前缀，可以用/调用。

12.1 打开 Selenium IDE
~~~~~~~~~~~~~~~~~~~~~~

​ 如果安装了扩展，则插件可能会发出打开 Selenium IDE 的请求。

::

   {
   openSeleniumIDEIfClosed: true 
   }

12.2 GET /health
~~~~~~~~~~~~~~~~

​ 用于插件运行状况检查，请参阅插件运行状况检查。

12.3 POST /register
~~~~~~~~~~~~~~~~~~~

​ 用于在 Selenium IDE 中注册您的插件，通过这种方式，IDE
可以知道您插件的存在， 请参阅注册插件。

12.4 POST /log
~~~~~~~~~~~~~~

​ 用于 system 日志，表示用户可以按系统日志组进行过滤的时间。

​ 解释插件用法或状态的日志应记录在此处。

::

   {
     uri: "/log",
     verb: "post"
     type: "log type", // error, warn, undefined
     message: "your log message goes here"
   }

-  type-日志类型，undefined 是信息日志，同时 error 会显示为红色，并 warn
   显示为橙色。
-  message- string 消息，任何链接将被自动链接。

12.5 Returns
~~~~~~~~~~~~

​ 如果添加了日志则返回 True。

12.6 GET /project
~~~~~~~~~~~~~~~~~

​ 获取的 id 和 name 当前加载的项目。

::

   {
   id: "auto-generated-project-id",
   name: "your-project-name" 
   }

12.7 POST /project
~~~~~~~~~~~~~~~~~~

​ 将项目加载到 Selenium IDE
中，就像用户打开它一样，如果用户尚未保存更改，则
对话框将在执行之前询问他。

::

   {
        project: JSON parsed side file
   }

12.8 POST /control
~~~~~~~~~~~~~~~~~~

​ 从另一个 Chrome 扩展程序开始连接。当用户接受此连接时，Selenium IDE
将重新 启动并注册调用方，并且该扩展将对 Selenium IDE
进行独占控制，直到用户关闭 Selenium IDE
或接受另一个连接为止。启用此模式后，通过将辅助文件发送到控制 Selenium
IDE 的扩展名，保存到计算机的功能将被覆盖。

​ 该调用的有效负载与 POST /register 调用的有效负载相同。

12.9 POST /close
~~~~~~~~~~~~~~~~

​ 当 Selenium IDE 由另一个 chrome
扩展程序控制时，控制器扩展程序可以使用此 API 关闭 IDE
窗口。如果用户有未保存的更改，它将提示用户放弃更改还是忽略关闭。无
需有效负载。

十三、回放API
-------------

​ 回放 API 用于 Selenium IDE 的回放功能。

​ 以/playback 开头。

13.1 GET /playback/location
~~~~~~~~~~~~~~~~~~~~~~~~~~~

​ 用于使用 Selenium IDE 解析定位器。

::

   {
     uri: "/playback/location",
     verb: "get",
     location: "valid IDE locator"
   }

-  location-有效的 Selenium IDE 定位器(例如 css=input.submit)。

.. _returns-1:

13.2 returns
~~~~~~~~~~~~

​ 如果找到元素，则返回 xpath;如果没有找到，则返回错误.

13.3 POST /playback/command
~~~~~~~~~~~~~~~~~~~~~~~~~~~

​ 用于更改命令状态(例如，通过，失败等)。

::

   {
     uri: "/playback/command",
     verb: "post",
     commandId: "the command's id",
     state: "valid state",
     message: "a message to show the user"
   }

-  commandId -当前正在运行的测试用例中出现的命令的命令 ID。
-  state-有 效 的 命 令 状 态 : ( ，failed，fatal，passed， ) 。
-  message
   -可选，显示给用户的消息，请注意，并非所有命令状态都支持显示消息。

13.4 Return
~~~~~~~~~~~

​ 命令状态如果已更改则返回 True。

13.5 POST /playback/log
~~~~~~~~~~~~~~~~~~~~~~~

​ 用于 playback 日志，表示用户可以按回放日志组过滤的时间。
记录回放进度或状态的日志应记录在此处。API 与系统日志相同。

十四、录制API
-------------

​ Record API 与 Selenium IDE 的录制功能有关。

​ 该 API 的前缀为/record。

14.1 GET /record/tab
~~~~~~~~~~~~~~~~~~~~

​ 获取记录器当前连接到的 tabId (即使当前没有记录也可以使用)。

::

   {
   uri: "/record/tab",
   verb: "get" 
   }

.. _return-1:

14.2 Return
~~~~~~~~~~~

​ 包含 id 标签的对象，如果未附加标签，则返回错误。

14.3 POST /record/command
~~~~~~~~~~~~~~~~~~~~~~~~~

​ 向当前记录的测试用例添加命令。

::

   {
   uri: "/record/command", 
   verb: "post",
   command: "the command id", 
   target: "an initial target",
   value: "an initial value", 
   select: true || false
   }

-  command -每个清单要插入的命令 ID。
-  target -命令的初始目标。
-  value -命令的初始值。
-  select- true 或 false
   添加命令后才能打开目标选择器，只能与命令类型结合使用。

.. _returns-2:

14.4 Returns
~~~~~~~~~~~~

​ 如果添加了命令则返回 true。

十五、弹出API
-------------

​ Popup API 处理用户可以看到的弹出窗口。

​ 该 API 的前缀为/popup。

​ 注意: Selenium IDE
一次显示一个弹出窗口，发送一个新的弹出窗口将取消上一个弹出窗口。

​ 因此，如果 seleniumide 返回 false (如果这意味着取消)
，则不应采取任何操作，因为弹出窗口可能已被关闭。

15.1 POST /popup/alert
~~~~~~~~~~~~~~~~~~~~~~

​ 用于在 Selenium IDE 内部显示 alerts 对话框。

​ 这可能会给用户带来烦恼和不便，因此请谨慎使用!

::

   {
     uri: "/popup/alert",
     verb: "post",
     message: "message to show the user",
     cancel: "cancel label",
     confirm: "confirm label"
   }

-  message -邮件正文。
-  cancel -可选，取消按钮文本。
-  confirm -可选，确认按钮文本。

.. _returns-3:

15.2 Returns
~~~~~~~~~~~~

​ 如果单击确认按钮，则返回 true;如果单击取消按钮，则返回 false。

十六、插件导出API
-----------------

​ 回放 API 与 Selenium IDE 的导出功能有关。

​ 该 API 的前缀为/export。

16.1 GET /export/location
~~~~~~~~~~~~~~~~~~~~~~~~~

​ 用于获取 WebDriver 代码以解析定位器。

::

   {
   uri: "/export/location", verb: "get",
   location: "valid locator"
   }

-  location-有效的 Selenium IDE 定位器(例如 css=input.text)。

.. _returns-4:

16.2 Returns
~~~~~~~~~~~~

​ 如果定位符有效，则返回解析为该元素的 JavaScript WebDriver 代码(例如
By.css (“ input.text”))。

​ 在任何其他情况下都是错误。
