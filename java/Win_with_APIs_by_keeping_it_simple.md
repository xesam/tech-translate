#Win with APIs by keeping it simple

原文：[Win with APIs by keeping it simple](http://www.javaworld.com/article/2877163/java-app-dev/win-with-apis-by-keeping-it-simple.html)

少即是多

##简单易懂

就像你不需要阅读使用手册就知道如何使用 iPhone 一样，一个好的 API 也不需要你先花上数天时间来摸清如何使用 API，然后才能展开工作。

作为开发者，不论你对 Facebook 本身或褒或贬，有一点不可否认，它的 API 设计非常直观和简单：一旦你通过了认证，接下里的事情就水到渠成了。

在浏览器里面输入 [https://graph.facebook.com/534822875](https://graph.facebook.com/534822875) ，就可以得到 ID 为 534822875 的用户信息，JSON 片段如下【译者注：这个可能与实际结果有出入，一切以原文为准】：

    {
       "id": "534822875",
       "location": {
           "id": "114952118516947",
           "name": "San Francisco, California"
       }
    }

Facebook 还创建了一个简写形式：using /me 来表示当前已经登陆的用户，也就是说，[https://graph.facebook.com/me](https://graph.facebook.com/me) 可以得到和上面一样的结果。【译者注：这个需要登陆之后才能有结果，所以，实验结果根据登陆状态和用户 ID 的不同而不同】

事实上，我通过自己在使用 Facebook 的功能（friends, photos, likes）时了解到的信息，几乎就可以猜到各个功能对应 API 的功能和方法名称了。

比如，[https://graph.facebook.com/me/likes](https://graph.facebook.com/me/likes) 返回我在 Facebook 上的喜好列表：

    {
       "data": [
           {
               "category": "Media/news/publishing",
               "name": "The Guardian",
               "created_time": "2014-08-01T01:32:17+0000",
               "id": "10513336322"
           },
           {
               "category": "Movie",
               "name": "The Lives of Others Movie",
               "created_time": "2014-07-31T15:47:14+0000",
               "id": "586512998126448"
           }
       ],...
    }

这个 API 非常直观，基本上就能猜到里面的有效资源字段名称。

那如何获得我的照片列表呢？

没错，就是：[https://graph.facebook.com/me/photos](https://graph.facebook.com/me/photos)

同样是获取照片列表，Flickr 的 API 是这样的：

    https://api.flickr.com/services/rest/?method=flickr.people.getPhotos

人们可以有无数对 Flickr API 的吐槽：比如“不是 RESTful 风格”等等。但是，最重要的问题还是：不够简单。
既不直观，也不容易记忆，更不容易理解。作为一个使用者，我总是需要回过头去对照 API 文档来使用，这一点阻碍了他的使用体验。

对照来看，Facebook 的 API 就简单直观得多，可以快速上手并立即开工。这也是为什么会有超过 2 万的开发者使用 Facebook 的 API 创建了超过 1百万 app 的原因。

##简单易用

使用 RESTful 的方式来构建API，可以帮助你遵循“理解简单”的原则，也意味着你可以像浏览器一样来处理 http 资源。
这会帮助你将 API 构建在实际的项目和业务关系上，当你在谈论 API 的时候，可以紧密的关联到员工，客户和用户所谈论的业务上，这将是一个巨大的好处。

下面的 RESTful 风格 API，通过它可以获取账户列表或者某个特定的账户信息：

    # 列出所有账户
    GET /accounts

    # 显示账户信息
    GET /accounts/{account-number}

下面的 API 构建在一个真实的生活场景之上，其返回一个特定账户的信息：

    GET /accounts/6242525/balance

你可以想象这样的场景，某个客户打电话过来问：我的账户号码是6242525，你能帮我查一下我的账户余额吗？

通过这种方式，就相当于用一种通用而富有表现力的语言在你的 API 和业务之间建立了一种映射，使得用户更容易理解。
使用 RESTful 的另一个好处是，可以方便的在浏览器中使用，实验验证也相对简单。

#数据格式简单

这么些年，开发者们已经厌倦了冗长的XML等格式。他们不再愿意编写自定义的解析器，转而寻求更轻量级的选择。JSON解决了这些问题，并且 JSON 正成为一个事实上的数据交换格式。
另外，像许多 API 用户一样，如果你在使用JavaScript，那么就能够很容易的集成JSON。

JSON是简单的键值对表。看看前面的例子，你可以看到，我喜欢“Guardian newspaper”，点赞时间是“2014年8月1日”

    {
       "category": "Media/news/publishing",
       "name": "The Guardian",
       "created_time": "2014-08-01T01:32:17+0000",
       "id": "10513336322"
    }

##API 导航

通过提供分页链接，导航 API 就很容易实现了，API可以直接告诉用户下一步的目标在哪，而无需用户去手动构建通往下一步的 URL，这一点在处理大型数据集方面显得特别重要。
以下是来自  Facebook’s Open Graph API 的优秀例子：其中所有响应都含有一个“paging”属性，它告诉用户在哪里可以得到上一个或下一个数据集。另一个好处就是，机器人或爬虫可以通过同样的方式来浏览你的API。

    {
       "paging": {
           "previous": "https://graph.facebook.com/me/albums?limit=5&before=NITQw",
           "next": "https://graph.facebook.com/me/albums?limit=5&after=MAwDE"
       }
    }

##扩展用法

让用户能够控制 API 的行为是一个强大的特性。有很多方法可以做到这一点，包括使用各种排序规则，搜索和过滤。
“字段过滤”就可以用来指定响应中应该包含哪些字段。这种特性在带宽或性能受限，或者其中用户只关心响应中的部分字段的情况下特别有用。
过滤的另一个好处是，将选择交给用户，让用户告诉 API 什么是他们关心的，从而使得 API 的创建者不必针对每一种需求来开发不同的 API。

    https://graph.facebook.com/me/likes?fields=category,name

上面的 API 调用告诉 API 在响应中只需要返回  “category” 和 “name” 字段即可。

    {
       "category": "Media/news/publishing",
       "name": "The Guardian",
    }

##维护简单

所有的API需要支持向后兼容，同时它也应该给用户提供一种只使用某个特定的版本的方法。要做到这一点，最简单的方法是在基础 URI 中包含版本号，然后默认不包含版本号的 URI 作为最新的API版本。这样做就给 API 用户一个使用特定版本或者最新版本 API 的选择余地。

# versioning （选择版本）
/api/v1.0/accounts/12345
/api/v1.1/accounts/12345
/api/v2.0/accounts/12345

# alias no-version to the latest version（最新版本）
/api/accounts/12345

API 的创建者需要小心的维护 API 使用规则，任何更改都需要经过认真的检查审核，并与用户沟通以防止惹怒用户。

##SSL

安全是一个关键因素。使用 HTTPS，通过加密和验证，以保证用户和服务器之间的通信的完整性，并防止中间人攻击， 从而提供一个安全的 API 访问环境。
甚至可以考虑不提供通过非安全HTTP 的 API访问。

##操作简单

当你的 API 有数百到数千用户的时候，你需要考虑一整套的高级特性支持。

##访问控制

您需要为开发人员提供 key 访问您的API，阻挡黑客入侵。你也会希望限制API用户的流量，防止用户的行为导致API崩溃——不论他是有意或无意的。

##监控

你需要知道你的 API 何时失效，何时必须重启。你还需要知道你的 API 使用情况：是谁在何时何地使用 API？。

##集成

您可能需要向用户提供 API 使用情况的分析和报告，也可能需要集成一个计费系统，为 API 使用收费。

其中一些特性是不容易实现的。事实上，实现这些特性可能是整个 API 开发周期的更复杂的方面之一。但是万变不离其宗，再次强调，核心还是要保持它的简单。
不要试图推到重建自己的方案，相反，使用现成的 API 方案来管理您的API。将精力投入到业务相关的部分，从而与你的竞争对手产生差异化。

创建一套成功的API，使得架构和业务更加灵活，甚至将推动业务和收入增长，并激励创新。
要记得——保持简单！


