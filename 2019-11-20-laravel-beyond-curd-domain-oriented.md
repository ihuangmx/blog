# Laravel 大型项目架构（1） - 领域驱动设计

《领域驱动设计》一书对领域 (Domain) 的定义

> 每个软件程序是为了执行用户的某项活动，或是满足用户的某种需求。这些用户应用软件的问题区域就是软件的领域。

我们从一个简单的例子说起，假如有一个酒店管理应用，包括了客户管理、预定管理、库存管理、发票管理等模块，现在要考虑如何对项目进行架构。

最常见的方案就是使用 Laravel 的目录结构

```
Broadcasting
Console
Events
Exceptions
Http
-- Controllers
-- Middlewares
Jobs
Listeners
Mail
Notifications
Policies
Providers
Rules
```

这样的架构方式最大的不足就在于无法反映出应用的业务逻辑。无论你是开发者、项目经理、甚至是客户，都无法从目录中快速获取业务信息。因此，这种设计对于大型项目来说并不适用，因为随着业务复杂度不断增加，代码会越来越难以维护。

所以，大型项目应该基于领域来进行架构。将相关的业务需求组织在一起，构成一个个的领域模型。这样做的最大的好处在于，**让代码反映出你的业务需求**。

如何基于领域来设计 Laravel 项目，可以对项目进行简单的划分

* 领域层 - 业务需求的实现
* 应用层 - 不包括业务规则，只是负责领域层与用户层的通信

领域层

```
app/Domain/Invoices/
    ├── Actions
    ├── QueryBuilders
    ├── Collections
    ├── DataTransferObjects
    ├── Events
    ├── Exceptions
    ├── Listeners
    ├── Models
    ├── Rules
    └── States

app/Domain/Customers/
    // …
```

应用层

```
app/App/Admin/
    ├── Controllers
    ├── Middlewares
    ├── Requests
    ├── Resources
    └── ViewModels

app/App/Api/
    ├── Controllers
    ├── Middlewares
    ├── Requests
    └── Resources


app/App/Console/
    └── Commands
```

这样做会破坏 Laravel 单 App 入口的约定，如果想保持单一 App，可将领域层以模块的方式进行组织。

```
|- app/
   |- Http/
      |- Controllers/
      |- Middleware/
   |- Providers/
   |- Account/
      |- Console/
      |- Exceptions/
      |- Events/
      |- Jobs/
      |- Listeners/
      |- Models/
         |- User.php
         |- Role.php
         |- Permission.php
      |- Repositories/
      |- Presenters/
      |- Transformers/
      |- Validators/
      |- Auth.php
      |- Acl.php
   |- Merchant/
   |- Payment/
   |- Invoice/
|- resources/
|- routes/
```

无论是使用哪一种方式，都是基于领域来进行组织代码。

参考链接：

* [01. Domain oriented Laravel - stitcher.io](https://stitcher.io/blog/laravel-beyond-crud-01-domain-oriented-laravel)