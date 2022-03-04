# 规范约束

## 引言

强行要求的规范，往往带来的实际效果不好，甚至起到负面效果。所以，本篇仅推荐给对自己有要求，希望能够学习到什么简洁有力的项目结构，本篇内容不存在最终版本一说，就像本文存在缺陷会不断更新，我愿同你一起成长，也欢迎你为本文提交新的内容。

## 业务说明

indicator 名为指标，在项目的数据表设计中，该名称对应的数据表存储的是指标树的一个一个节点以及对应所属关系（pid），存在树型的层级结构，树型的每一层都是和业务紧密关联的：根 → 类别 → 维度 → 指标 → 变量，主要有这几个层次级别，其中 `指标` 这两个字就会经常出现歧义，如 指标id 可以表示指标表任意一条数据的id，也可以表示指定层级的指标的id；所以在项目的源码注释或是沟通上经常可以看见这样的描述，`指标 - 指标`、`指标（狭义）` 来特指指标层面的指标节点

项目数据表中 indicator、ranking_type 存储的数据结构是树状结构，树状结构的节点数据结构有如下特征

- pid

- path

- level

## 代码风格

go 中不推荐空格的源码编排风格，go fmt 命令会将所有的空格的编排风格转换为 tab

- 注释 `\\` 和注释内容之间都应该有一个空格
- `conf` 配置文件中支持的注释语法有两种 `#` `;`，约定前者为大模块注释，后者为小模块注释

### 注释（GoDoc）

[参照]: https://go.dev/blog/godoc

开发者应遵从，Go 文档规范，即所有方法头注释都应该遵从这样格式的注释：

`// 方法名 注释内容`

**过期**

当某些方法需要更新，负责更新的开发人员应当在旧的方法实现上添加这样格式的注释：

`// Deprecated: 过期原因 新的实现位置`

## 类型规范

### 枚举

在项目结构中了解到，静态概念都放在 `models/const` 包下，但是 `const` 同时也是 go 的关键字，综合考虑，决定采用包名表示枚举含义，所有具体的枚举类型都叫 `Enum`

**拓展方法**

```go
var text = map[Enum]string {
    Xxx1: "xxx1",
    ...
}

type (e Enum) String() string {
	return text[e]
}

type (e Enum) Valid() bool {
    switch e {
    case Xxx1, Xxx2, ...:
        return true
    default:
        return false
    }
}

type (e Enum) Value() <trueType> {
    return trueType(e)
}
```

**例一**（基础）

```go
// gender/enum.go
package gender

type Enum int

const (
    Female Enum = 0
    Male   Enum = 1
)
```

**例二**（复合）

```go
// fruit/enum.go
package fruit

type Enum struct {
  Code int
  Name string
}

var (
  Apple      = Enum{code: 1, name: "苹果"}
  WaterMelon = Enum{code: 2, name: "香蕉"}
  // ...
)
```

**例三**（多模块，单文件）

```go
// indicator/enum.go
package indicator

type LevelEnum int
type IdEnum int64
...
```

**例四**（多模块，多文件）

```go
// indicator/level.go
package indicator

type LevelEnum int
...
```

```go
// indicator/id.go
package indicator

type IdEnum int64
...
```

## 命名规范

### 包名

**慎用 `test`**

用于代表测试代码的源码文件 `xxx_test.go` 中定义的公共变量、常量，其他源码文件无法获取引用

`beego` controller/test 中的 Controller 会被忽略

**善用 `internal`**

不希望，也不应该对外暴露的包，应该将其命名为 `internal`

### 文件命名

不应该有如：`包名_文件名.go` 或 `文件名_包名.go` 的命名风格。

如希望代表 `xxx` 模块的接口定义文件，应该命名为 `i_xxx.go`。

模块包的入口或核心源码文件，应该以模块名命名，而不是譬如 `core.go`。

### 通道变量名

仅用作信号通道，传输的数据没有实际含义时，命名为：`xxxSig`

通道传输的数据有实际含义时，命名为：`xxxChan`

### 三层方法命名

#### DAO

- 查询的是单条还是列表，从特殊标识和表约束看出 → 不在方法名中体现。

- 查询的是完整实体、还是部分字段，因为命名的复杂性，目前还是以偏概全，你希望复用某个查询，如果代价不大，只要添加字段就行，不会创建多个方法 → 不在方法名中体现（从具体的实现看出）。

- 查询条件，需要在方法名中体现，但是条件名只需要体现概念即可，如通过院校编号查询 `ByUniv`，因为一般查询某类事物的条件是相同的，这里通过编号，那里通过 id，这样的情况较为罕见，假如真的出现了，再完善的条件名 `ByUnivCode`、`ByUnivId`。另外，特殊的业务表，必要的字段可以不用写，如查询某张表的数据是，必定带有版本，版本就不用体现出来（从具体的方法参数类型看出）。

- 此外，查询数据或者逻辑较为复杂（同时查询出最大值 或者 包含存储过程、CTE 等），方法命名只要遵从标识业务模块的效果就行

#### Service

- 如果是DAO 方法封装（List → Map、包装了错误信息），一般命名同封装的方法名
- 除此之外（拼装业务数据，调用其他多个模块的数据查询），命名同业务名

**举例**

> `beego orm` 默认提供的方法：`Insert`、`Update`、`Delete`，为了避免冲突，可以按照如下方式命名

保存或更新：`Upsert`

保存：`InsertOne`

更新：`UpdateOne`

### Receiver 命名

**三层**

`type XxxController struct`、`（c *XxxController）`

`type xxxService struct`、`（s *xxxService）`、`type XxxService interface`、`NewXxxService XxxService`

`type xxxDao struct`、`（d *xxxDao）`、`NewXxxDao *xxxDao`

> xxx 后面的层可以省略？最好不要省略，因为枚举定义规范中，包名就是其业务概念名，即 `xxx.Enum` 会和结构体 `type xxx struct` 发生名称冲突，只能在引用的地方给包名取别名

**接口参数实体**

`Xxx struct（f *Xxx）`

**SQL**

`q := 'SELECT ...'`

`b := strings.Builder{}`

`p := make([]interface{}, 0, n)`

## 项目结构

### Beego

|                       | beego 生成 | 说明                                                         |
| --------------------- | ---------- | ------------------------------------------------------------ |
| conf                  | true       |                                                              |
| routers               | true       |                                                              |
| docs                  | true       |                                                              |
| tests                 | true       |                                                              |
|                       |            |                                                              |
| controllers           | true       |                                                              |
| controllers/internals | false      | 一些内部的，不提供给前端开发人员的接口。如果数据维护、刷新缓存等 |
|                       |            |                                                              |
| models                | true       | 最简单纯净的数据表实体                                       |
| models/internal       | false      | 项目中定义的能够根据数据表自动生成 Go 数据实体源码的 Groove 脚本，生成的数据实体 |
| models/const          | false      | 常量、枚举；数据表中特定业务含义数据的指定字段值等           |
| models/bo             | false      | 带有复杂业务逻辑处理方法的结果集实体，即使会直接作为结果集返回，也不应该放到 dto 中 |
| models/dto            | false      | 接口结果集实体（包的循环依赖：不定义实体转换方法）           |
| models/form           | false      | 接口参数实体                                                 |
|                       |            |                                                              |
| repo                  | false      | DAO                                                          |
|                       |            |                                                              |
| usercase              | false      | 业务处理                                                     |
|                       |            |                                                              |
| module                | false      | 独立模块（不含 dao 的数据交互操作，如果含有，整个模块应该放到 usercase 下） |
| filter                | false      | 拦截器                                                       |
| help                  | false      | 工具类                                                       |

### Gin

|            | 必须  | 说明                                                         |
| ---------- | ----- | ------------------------------------------------------------ |
| base       | true  | 和业务无关的基础模块父目录                                   |
| - config   | true  | 配置文件到  Go struct 实例映射                               |
| - constant | true  | 枚举、常量                                                   |
| - dao      | true  | MySQL 数据交互（基于 sqlx 封装了 事务、日志、自定义错误处理、动态 sql 拓展） |
| - errs     | true  | 通用错误定义、处理                                           |
| - filter   | true  | 拦截器（鉴权、IP 白名单、请求日志打印）                      |
| - log      | true  | 基于 zap 的日志模块（包含请求日志的打印方法）                |
| - server   | true  | 项目启动（grace close）                                      |
| - validate | true  | 对于有繁多的数据校验需求，可以使用该模块来简化开发流程       |
| cache      | false | 内存缓存；某些数据在逻辑上相当重要，使用频繁，作为单独的数据模块进行管理 |
| controller | true  |                                                              |
| service    | true  |                                                              |
| dao        | true  |                                                              |
| model      | true  |                                                              |
| - do       | false | 数据表实体；一定有 db 标签，可以没有 json 标签               |
| - bo       | false | 业务实体；可以有 json 标签                                   |
| - dto      | false | 接口响应实体；一定有 json 标签（当 dto 需要被多处引用时，应该将其提升为 bo） |
| - form     | false | 接口参数实体；有 uri 或 json 标签（不推荐 form，即表单，除非是文件上传场景） |
| - const    | false | 之所以使用关键字作为包名是希望子级都单独创建包               |
| module     | false | 业务模块；service 不允许引用 module，避免循环依赖问题        |
| router     | true  | controller 的注册中心                                        |
| util       | true  | 和业务无关的工具类                                           |

**改进方向**

`config` 含有业务概念

`errs`  含有业务概念

`filter` 含有业务概念

`validate` 属于工具

### Go-Zero（TODO）

## 接口规范

### Restful

- GET

  QUERY

- POST

  JSON（不推荐 FormData）

  >  前端的 axios 库、各大厂商的开放 api，接口参数默认都是 JSON 格式

  > 与其约定好的数据格式（逗号隔开的字符串），就不如使用共识的 JSON 格式

- PATCH

  JSON - 更新部分字段

- PUT

  JSON - 如果 创建 和 更新 接口是分开的，POST 用于创建、PUT 用于更新

- DELETE

  PATH

### 参数校验

在简单非业务的参数校验上去编写重复的代码，本身就是低效，无意义的做法，此时应该使用相关的校验类库去做这些事情

> beego

因为 beego 不再是后续项目开发的基础框架选行，故不展开说了

需要配置翻译、以及，结合 `beego` 的 `validation.ValidFormer`

> gin

基于 gin 现有的基础模块 `base/controller.go` 已经封装好了，直接可以使用的接口参数校验方法；集成中文和英文两种翻译规则的翻译，以及自定义校验标签的集成拓展

## 接口文档

### 大纲结构

- 页面列表

  页面 1：需要调用的接口链接列表

  页面 2：需要调用的接口链接列表

  ...

- 接口列表

- 接口 A

  业务场景说明：描述清楚业务背景、注意事项等

  接口地址：首先，标注清楚请求方式；然后，如知道具体部署环境的接口地址，就把每一项列举清楚；如不知道，给出相应的路由地址即可，如：`[GET] /v1/user`

  参数说明（名称、类型、位置、是否必须、备注）

  参数样例

  响应说明（名称、类型、描述）

  响应样例

- 接口 B

  ...

- ...

- 其他

  如业务中有进行三方对接，就贴出所有三方相关的文档地址链接

### 业务迭代

当产品提出新的需求，业务进行迭代，除了更新主接口文档，还应该创建编写一份变更文档来进行说明

### 其他

暂无严格要求，文档旨在将业务和每个接口设计说清楚，如果文档能以一定的结构阐述清楚，以上规则可无视

部门现在正在尝试 `ApiFox`、`ApiPost` 等接口管理的三方解决方案

## Service

**树型节点**

关于树型结构数据节点的通过 id 查询一条数据的方法（service 层），`FindById` 方法可以添加一个可变参数的形参，当参数不为空时，就应该在方法中查询后，进行校验

`FindById(id int64, expectLevel ...int) (*models.Xxx, error)`

## Dao

### 定义实体（models）

传统项目的开发流程是：了解业务需求 → 表设计 → 编写接口文档 → 前后端并行开发，而在后端开发的时候，无可避免的需要为每一张表创建对应的 Go 结构体，即表实体。

这里提及一个 `GROOVY` 脚本文件的使用，来避免重复枯燥的工作，`Copy as Go struct.groovy`，可以直接生成数据表对应的 Go 中的实体定义源码。

将脚本文件放置到 Goland 项目的 `(a)` 目录下，然后通过 Goland 自带的 DB 连接工具，连接到指定的数据库，通过 `(b)` 操作应用运行脚本。

> (a) /Scratches and Consoles/Extensions/Database Tools and SQL/schema/Copy as Go struct.groovy

> (b) 框选想要操作的表 → 右键 → Scripted Extensions → Copy as Go struct.groovy → 剪切板中就存有了实体定义

[beego]: https://github.com/shanghairanking/docs/blob/main/3.%E6%95%B0%E6%8D%AE%E5%BA%93/Copy%20as%20Go%20struct%20beego.groovy
[sqlx]: https://github.com/shanghairanking/docs/blob/main/3.%E6%95%B0%E6%8D%AE%E5%BA%93/Copy%20as%20Go%20struct%20sqlx.groovy

**业务规则**

|          | 概念                                                         |
| -------- | ------------------------------------------------------------ |
| 整型主键 | GO 类型：int64、MySQL 类型：int                              |
| 枚举     | 得到枚举名称 → SELECT xxx<br />得到枚举序号 → SELECT xxx + 0 AS xxx |
| 是否为空 | GO 语言中所有基础数据类型都有对应类型的零值，所以当要表示无意义的空值时，应将字段类型改为特定类型对应的指针类型；在 dao 框架的映射行为上，sqlx 要求强类型对应，而 beego 默认零值转换兼容。 |

### 基础

**查询一条（QueryRow）**

- 注意：需要为接收结果的实体初始化

  ```
  var m = new(models.M)
  err := r.Raw(q, id).QueryRow(m)
  ...
  ```

- 注意：需要考虑没有查询到的情况

  `beego orm` 查询一条数据（`QueryRow`），如果 没有找到 或 找到多条，都会返回 error

  一个，这样的错误是不能直接返回给前端的，需要处理，可以直接使用 `help.OneRowErr(err, string)` 包装错误后直接返回返回

  还提供了辅助方法：`help.IsOneRow(err error) bool` 和 `help.IsMultiRows(err error) bool`

**查询多条（QueryRows）**

`beego orm` 查询多条数据（`QueryRows`）；结果集容器并不需要初始化；在没有查询到数据时，并不会报错

### 事务

文档作者，在大半年的开发实践中，经常会使用在方法局部开头声明接下来要使用变量的方式，使得代码结构更为工整，提高代码的可读性，其中最常定义（几乎是必声明）的就是 `error` 类型的变量，于是便将原来的基于一个 `bool` 类型事务提交标识值，改为 `error` 类型；实际调用：

```go
// 早期
var err error
...
o = orm.NewOrm()
_ = o.Begin()
defer help.HandlerDBTransaction(o, &err)
...
```

```go
// 后来
var err error
....
d := dao.NewXxx(nil)
d.MustBegin()
defer d.HandleTx(&err)
...
```

### JSON 类型

**查询映射（早期）**

查询 MySQL JSON 类型的数据，对于 Go 中大多数的 ORM 框架，都可以使用 string 类型的字段来接收，实际上，也只有 string 类型能够接收 JSON 的结果，JSON 字符串嘛。

但是实际都是希望能直接映射到对应结构体类型中。早期只知道手动处理；定义一个冗余字段，存储中间结果，实现数据类型转换。

> beego 实际上要求不严格，所以 orm 标签可以省略

```go
XxxJSON string `orm:"column(xxx)"`
Xxx     *Xxx   `orm:"-"`
```

> sqlx

```go
XxxJSON string `db:"xxx"`
Xxx     *Xxx   `db:"-"`
```

**反序列化**（帮助方法）

```go
Xxx *Xxx `db:"xxx"`

// sql.Scanner 接口
func (x *Xxx) Scan(src interface{}) error {
    // 通用逻辑
    if src == nil {
		return nil
	}

    switch v := data.(type) {
	case []byte:
		return json.Unmarshal(v, x)
	case string:
		return json.Unmarshal([]byte(v), x)
	default:
		return fmt.Errorf("data type is valid, is %+v", data)
	}
}
```

**序列化**

数据表字段类型是 JSON，Go 中只能通过 string 类型的字段去接收。早期，就像上面一样需要定义辅助字段，需要手动处理；后面发现和上面对应的自定义序列化操作。

```go
Xxx *Xxx `db:"xxx"`

// Value for driver.Valuer helper
// 假如表字段类型是 JSON，那么这里具体返回 []byte 或 string 类型都可以
func Value(def string, data interface{}) (interface{}, error) {
	vi := reflect.ValueOf(data)
	if vi.IsZero() {
		return []byte(def), nil
	}
	return json.Marshal(data)
}

func ValueNil(data interface{}) (interface{}, error) {
	vi := reflect.ValueOf(data)
	if vi.IsZero() {
		return nil, nil
	}
	return json.Marshal(data)
}

func ValueEmpty(data interface{}) (interface{}, error) {
	return Value("", data)
}

func ValueObj(data interface{}) (interface{}, error) {
	return Value("{}", data)
}

func ValueArr(data interface{}) (interface{}, error) {
	return Value("[]", data)
}
```

**总结**

和三方的框架，dao 层框架都没有关系，驱动相关的包已经定义好了机制和实现拓展。实现了 `sql.Scanner` 和 `driver.Value` 的结构体类型能够 `直接` 作为方法的参数，以及查询出 JSON 能够自动反序列化到结构体中，无需再定义额外的字段、额外的手动处理了。

**题外话**（早期）

> SQL 需要体现出其类型，才能得到理想的结果

```mysql
# 表字段 json 类型
CAST(? AS JSON)
JSON_OBJECT('name', ?, "age", ?)

# json 类型表字段中的 bool 类型
如果希望设置进一个布尔值而不是，0 或 1，需要在参数占位符后边加上 `IS TRUE` 来表示布尔类型的值
```

```mysql
# 更新操作 SQL 层面（不推荐）
UPDATE xxx
SET xxx_info = JSON_SET(IFNULL(xxx_info, JSON_OBJECT(), '$.name', ?, '$.age', ?)
WHERE ...

# 更新操作 代码层面
UPDATE xxx
SET xxx_info = CAST(? AS JSON)
WHERE ...
```

**最佳实践**（嵌套复杂样例）

如果对着点来说，太复杂了，直接上实例

[表]

```mysql
CREATE TABLE xxx (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    name        VARCHAR(50) NOT NULL COMMENT '名称',
    detail      JSON DEFAULT '{}' COMMENT '详情',
    field       JSON NOT NULL COMMENT '属性',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP NOT NULL COMMENT '创建时间',  
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP NOT NULL COMMENT '创建时间',
);
```

[实体]

```go
type Xxx struct {
    Id         int64     `db:"id"`
    Name       string    `db:"name"`
    Detail     *Detail   `db:"detail"`
    Field      Field     `db:"field"`
    CreateTime time.Time `db:"create_time"`
    UpdateTime time.Time `db:"update_time"`
}

// 实现 driver.Valuer 决定保存时，入库的字段参数
func (x *Xxx) Value() (interface{}, error) {
    return []interface{}{x.Name, X.Detail, x.Field}, nil
}

// 不能实现 sql.Scanner，否则会在查询一条数据时
// github.com/jmoiron/sqlx@v1.3.1/sqlx.go:759 返回错误

type (d *Detail) Scan(src interface{}) error {
    // 需要处理 nil 的情况，并且当前字段本来就可以为 nil
    // if src == nil {
    //    return nil
    // }
    // 实际上就应该认定 src 为 []byte 类型，值是一个 JSON 串
    // return json.Unmarshal(src.([]byte), d)
    
    // 但是，封装好了一个专门的方法，能够合理的应对各种各样的场景
    // 也就是上面的 Scan 方法，该方法甚至可以处理 string 类型，但是值为 JSON
    // 当然，真有这种情况，Value 方法也应该做对应的处理了
    return Scan(src, d)
}

// 注意 receiver 不应该带 *
type (d Detail) Value() (driver.Valuer, error) {
    // 同上，也有定义好的方法
    // 主要目的就是让 json 类型的值入库，对应上
    // 然后，就是说 Detail 真实的类型
    // []xxx → []
    // struct → {}
    // *struct → <nil>
    // 自定义："[0]"（各种自定义逻辑都行）
    return ValueNil(d)
}

type Field struct {
    FieldInfo FieldInfo `db:"fieldInfo" json:"fieldInfo"`
}

func (f *Field) Scan(src interface{}) error {
    return Scan(src, f)
}

func (f Field) Value() (driver.Value, error) {
    return ValueObj(f)
}

// 假如实体中的 JSON 类型的字段中并都是基础类型，还嵌套着 Object 或是 Array
// 那么就还需要继续定义实体

type FieldInfo struct{...}

// 如果有自定义数据格式逻辑 + 会有数据表实体的 field 字段查询
// 应该为 Field struct 的 FieldInfo 字段定义 json 标签
// 应该实现 json.Unmarshaler 接口，在里边定义自定义反序列化逻辑

// 如果有自定义数据格式逻辑 + 特殊的该字段查询（SELECT field ->> ‘$.fieldDetail’）
// 应该为 Field struct 的 FieldInfo 字段定义 db 标签
// 那就应该实现 sql.Scanner 接口，在里边定义自定义反序列化逻辑
```

**最佳实践**（特殊逻辑样例）

```go
// versions 在数据库中定义的数据含义是，用英文逗号隔开的年月，如 "2020-01,2021-01,2022-01"；非空，默认空串
// 而实际代码程序中，希望直接操作 []int 这样的类型，所以就需要我们自定义 序列化 和 反序列化 过程了

type Xxx struct {
    Versions CommaSlice `db:"versions"`
}

type CommaSlice []string

func (c *CommaSlice) Scan(src interface{}) error {
    *c := strings.Split(src.(string), ",")
    return nil
}

func (c CommaSlice) Value() (driver.Value, error) {
    if c == nil {
        return []byte(""), nil
    }
    return []byte(strings.Join(c, ",")), nil
}
```

**总结**

`json.Unmarshaler`、`json.Marshaler`

`sql.Scanner`、`driver.Valuer`

基层框架代码设计对这些接口的支持，我们应该通过它们，在程序代码中实现好，这些类型里包含的业务逻辑。

自定义序列化逻辑，利用好：`json.Marshaler`、`driver.Valuer`

自定义反序列化逻辑，利用好：`json.Unmarshaler`、`sql.Scanner`

（自定义接口方法实现都有两个，具体应该实现哪个，视业务的查询的背景，视实体在表实体中所处的层级）

### 多值条件

> 基于 sqlx.In 可以避免繁杂的 sql 拼写，以及参数列表拼装

```go
func (d *xxxDao) ByIds(ids []int64, status int) ([]models.Xxx, error) {
	q := `SELECT * FROM xxx WHERE id IN (?) AND status = ?`
    
    qi, ai, err := sqlx.In(q, ids, status)
    if err != nil {
        return nil, err
    }
    
    var list []models.Xxx
    err = d.Select(&list, qi, ai...)
    return list, err
}
```

### 批处理

**批量插入**

```go
// 早期（手动拼接）
func (r *xxxRepo) BatchInsert(xxxs []xxx.Xxx) error {
    const fieldSum = n
    one := `(` + help.OrmJoinRepeat(fieldSum) + `)`
    
    // sql
    b := strings.Builder{}
    b.WriteString(`
    INSERT 数据库名.表名
    (字段名1, 字段名2, 字段名3, ..., 字段名n, created_by)
    VALUES `)
    b.WriteString(help.JoinRepeat(one, ",", len(xxxs)))
   
    // 实际参数
    params := make([]interface{}, 0, fieldSum*len(xxxs))
    for _, xxx := range xxxs {
        params = append(params, xxx.Field1, ..., xxx.Fieldn, xxx.CreatdBy)
    }

    _, err := r.Raw(b.String(), params...).Exec()
    return err
}
```

```go
// 后期 sqlx
func (u *User) Value() (driver.Value, error) {
    return []interface{u.Xxx1, u.Xxx2, ..., u.Xxxn, U.createdBy}, nil
}

func (d *xxxDao) BatchInsert(users []*models.User) error {
    q := `
    INSERT INTO xxx.user
    (field1, field2, ..., fieldn, created_by)
    VALUES` + util.JoinRepeat("(?)", len(users))
    
    p := make([]interface{}, 0, len(users))
    for _, v := range users {
        p = append(p, v)
    }
    
    query, args, err := sqlx.In(q, p...)
    if err != nil {
        return err
    }
    
    _, err := d.Exec(query, args...)
    return err
}

func (d *xxxDao) NamedBatchInsert(users []*models.User) error {
    q := `
    INSERT INTO xxx.user
    	(field1, field2, ..., fieldn, created_by)
    VALUES
    	(:field1, :field2, ..., :fieldn, :created_by)`
    
    _, err := d.NamedExec(query, users)
    return err
}
```

**批量 插入或更新**

> 为了凸显 SQL 语句的写法

```go
# 注意：如果表中有其他非空字段，下面这种方式就不好使了（除非手动填一些值）
# Error 1364: Field 'xxx' doesn't have a default value
func (r *xxxRepo) BatchUpsert(userId int64, xxxs []xxx.Xxx) error {
    const fieldSum = n
    one := `(` + help.OrmJoinRepeat(fieldSum) + `)`
    
    b := strings.Builder{}
    b.WriteString(`
    INSERT 数据库名.表名
    (id, 字段名1, 字段名2, ..., 字段名n-1)
    VALUES `)
    b.WriteString(help.JoinRepeat(one, ",", len(xxxs)))
    b.WriteString(`
    ON DUPLICATE KEY
    UPDATE xxx1 = VALUES(xxx1), ..., xxxn-1 = VALUES(xxxn-1), updated_by = ?`)
   
    params := make([]interface{}, 0, fieldSum*len(xxxs))
    for _, xxx := range xxxs {
        params = append(params, xxx.Id, xxx.Field1, ..., xxx.Fieldn-1)
    }

    _, err := r.Raw(b.String(), params, userId).Exec()
    return err
}
```

```go
func (r *xxx) BatchUpsert(userId int64, xxxs []xxx.Xxx) error {
	const fieldSum = n
	one := `
     UNION ALL SELECT ? AS id, ? AS xxx1, ..., ? AS xxxn-1`

	b := strings.Builder{}
	b.WriteString(`
	WITH args AS (
     SELECT
		? AS id, ? AS xxx1, ..., ? AS xxxn-1`)
	b.WriteString(help.JoinRepeat(one, "", len(xxxs)-1))
	b.WriteString(`
    )
	UPDATE 库名.表名 a JOIN args USING(id)
    SET a.xxx1 = args.xxx1, a.updated_by = ?`)

	params := make([]interface{}, 0, fieldSum*len(xxxs))
	for _, f := range xxxs {
		params = append(params, f.Id, f.Xxx1)
	}

	_, err := r.Raw(b.String(), params, userId).Exec()
	return err
}
```

### 业务注意

**查询**

- 所有查询：一定要注意 `deleted_at IS NULL`

- 列表查询：要注意，业务场景下，是否要求排序 `ORDER BY ord_no`

- 列表查询：模糊查询条件，`LIKE ?`，在具体的参数指定 `"%" + xxx + "%"`

  注意，SQL 语法中， 除非模糊查询的条件是数字，否则条件的具体值要加上单引号

**创建**（`created_by`、`updated_by`）

- created_at 和 updated_at 在表字段定义时，设置了默认值

**更新**（`updated_by`）

- updated_at 在表字段定义时，设置了自动更新

- `UPDATE` 更新多个字段值，`SET xxx1 = ?` 后面接第二个字段，拼接的不是 `AND`，而是 `逗号`

- `AND 条件1 OR 条件2`，注意 `条件1 OR 条件2` 是否要用括号括起来

**软删除**（`updated_at`、`deleted_by`、`deleted_at`）

- updated_at 会自动更新，但 updated_by 不会

- 带有 `含有唯一约束的字段` 和 `deleted_by、deleted_at 字段` 的数据：

  ```mysql
  UPDATE
  	xxx
  SET
  	name = CONCAT(id, '~', name),
  	phone_num = CONCAT(id, '~', phone_num),
  	email = CONCAT(id, '~', email),
  	deleted_at = ?,
  	deleted_by = ?
  WHERE
  	id = ?
  ```

### 主库

如果项目中有用到主库的数据，要注意和项目关联较大库的查询表名前边一定要带有库名

目前项目的主库有两个：`univ_a` 和 `univ_b`，这两个库是一模一样的，只是说哪个库导入了新的数据，就应该被切换为线上实际使用的，所以是轮番切换使用的；这样做也是避免直接更新线上数据库带来的种种弊端且能实现线上应用的快速切库，和备份数据的效果。

对于应用来说，切换数据库就是修改配置文件中配置的数据库名（默认这些数据库都是存放在一个 MySQL 服务端程序中的），因为查询中没有指定数据库名的表查询，默认就是应用这个数据库，为了实现综上所述，在进行和项目应用本身相关程度比较大的那个数据库的查询的表名前面应该都带上这个数据库的名称，具体表现在 `FROM` 和 `JOIN` 关键字后边的表名

> 现采用了 公共资源服务实例 和 动态表名 机制来处理这样的数据变更、切换机制了

## 日志规范

后端开发，经常需要处理各种异常和错误情况：

业务逻辑：数据状态不符合特定的接口调用场景，代码中主动报出的错误

常见类库：`json.Marshal`、`strconv.ParseInt`

中间件依赖：`sql.DB.Exec`（MySQL）、`redis.Bool`（Redis）

当发现这些错误，健全的后端程序，应该将上下文环境，以及具体的错误信息以日志的形式打印出来，并将包装后的错误信息返回给前端（不能将真实错误情况返回，保证后台安全），错误信息有对应业务场景的、也有后端程序代码无法处理的，如 MySQL 挂了，无法处理的情况下，接口应当返回，类似 `请联系管理员` 的信息。

**健全的日志样例**

```go
func (s *xxxService) Func(param1 int, param2 string) (interface{}, error) {
    // ...
    
    // dao 查询
    if err != nil {
        return nil, log.ErrorW("查询巴拉巴拉出错|系统异常，请联系管理员", fmt.Sprintf("[param1:%d] [param2:%s]", param1, param2))
    }
    
    // ...
    
    // 业务出错
    if !exists {
        return nil, log.ErrorW("参数非法|巴拉巴拉异常", fmt.Sprintf("不存在巴拉巴拉 [param1:%d] [param2:%s]", param1, param2))
    }
}
```

**日志级别**

Debug（常见）：目前仅用于非生产模式下的 MySQL 语法打印、测试调试

Info（较少）：业务中比较重要的流程、三方接口调用的请求和响应。

Warn（极少）：当接口请求参数没有通过校验、某些数据或者业务条件出现异常情况，但是需要保证业务流程，所以会进行一些特殊处理，此时比较适合该级别的日志打印。

Error（常见）：任何预期内和预期外都应该进行该级别的日志打印。

## 构建部署

传统项目部署，有构建的二进制包体积大、构建时间长、上传时间长、项目运行环境多（不同环境的配置文件，具体参数项，差异较大）等痛点。

目前暂未有这样的问题，故项目的部署更新模式，还是以全过程手动操作为主。

服务器上的项目部署更新，要求有一个好习惯，就是将一定时间内的二进制程序包，保留下来，以应对新程序包出问题时，能够立马回滚，保证服务的可用。

```
@echo off
set proj_name=项目名
set ts=%date:~2,2%%date:~5,2%%date:~8,2%.%time:~0,2%%time:~3,2%%time:~6,2%
set ts=%ts: =0%
set exe_name=%proj_name%-%ts%.exe
set upx_name=%proj_name%-%ts%-upx.exe
go build -o %exe_name% && upx -9 -o %upx_name% %exe_name% && rm %exe_name%
```

注意，在最后构建，还做了一个压缩，`upx` 是一个开源的二进制文件压缩工具

## Git 版本管理

### commit message

```markdown
feat:     增加新功能
change:   功能变动
fix:      修复bug

style:    不影响代码含义的改动，例如去掉空格、改变缩进、增删分号、修改变量名、将魔法值抽取成常量、修改注释等
docs:     只改动了文档相关的内容

refactor: 代码优化、代码重构
perf:     提高性能的改动

build:    go.mod 中类库的版本升级
test:     添加测试或者修改现有测试
```

特殊说明，补交能紧接着遗失的提交，就不需要标签，否则标记为 Fix

### Changelist 

关闭线上的一些严格的校验规则、修改一些框架文件或日志文件等的生成目录配置等的本地开发的非跟踪的文件改动，一般会使用 IntelliJ 的 CVS 提供的 `Changelist` 功能，但是需要注意，假如非提交 `Changelist` 暂存的内容和实际开发需要提交的代码有重叠，提交时就会被忽略！！！

## 最佳实践

### 配置文件

为了业务的发展和需求的迭代，拓展和统一入口，会将一些模块组件的策略放到配置文件中。但是，为了安全，配置文件是不能放到代码仓库中管理的，所以配置文件内容如果有新增一定要注意落实

凡是提供业务数据接口服务的后端，都会需要一个映射核心配置文件的变量实例，来方便获取配置项；几乎所有主流开发语言的项目框架都会提供这样一个机制

Beego 会将从 `conf/app.conf`（注意，配置文件的路径和名字是死的，这也造成后期拓展到 `app.yaml` 遇到阻碍） 中读取到的配置，加载到 `web.AppConfig` → `web.BConfig`

虽然可以通过 `web.AppConfig.Xxx("xxx")` 获取配置文件中的任意配置项，但是没有配置项大纲的提示，总是不合适的，所以，肯定需要自定义配置项到自定义配置实体的映射，因此由项目早期负责人利用 Go 的反射机制，实现了将 `web.AppConfig` 中配置项读取到自定义实体的逻辑（逻辑放在 `init` 函数中），并在没有读取到配置实体对应的配置项时抛出异常，以提示后端开发人员核对配置项

**改进（配置文件格式）**

随着业务的持续发展迭代，项目的配置项可能越来越多，简单的 `ini` 配置形式的配置文件逐渐体现出弊端：可读性差（无法体现出配置的层级关系）、不好维护管理（配置名冗长）

因此，引出 `yaml` 格式的配置文件，使用一些现成的三方包，能够轻松的做到，将 `yaml` 的数据映射到配置实体中；但是问题也来了，Beego 默认的配置文件是写死的，所以只能在主要使用 `yaml` 格式的配置文件基础上，同时保留 Beego 的原生配置 `app.conf`

**修正（配置文件位置）**

> 前置：运行路径的影响

并且在项目中白那些某些 test 的时候，或多或少都会使用工具包下的方法，而自定义配置文件配置数据到配置实体的逻辑就在工具包下，这时，就会因为执行路径的响应，导致无法读取到配置文件报错

因此，应当将配置文件的读取逻辑放置到单独的模块目录，这样使用工具包就不会有各式各样的问题了

### 分页

分页查询的结果一般包含总数，以及当前页的数据，这里存在简单的优化准则，先查询得到数量，如果发现查询得到的数量为 0，则不应该再进行数据库的数据查询

### 线上下路由

因为默认服务端架构都会采用一个网关中间件，网关中间件又需要区分请求是访问前端资源还是后端接口数据，因此，后端数据接口的访问地址会变为：`http://127.0.0.1:8080/v1/userinfo` → `https://xxx.xxx.xxx/api/v1/userinfo`（多了 `api`）

### 加密

> 参照 [私有仓库](#私有仓库)

目前，前后端约定好了一种对称加密的方法，所有需要涉及重要隐私权限的地方都需要使用到该加密规则

使用需要引入 `github.com/shanghairanking/helper v0.0.2`

注意，和前端对接时，`enc.NewAESCipher(密钥, openSSL)`的 `openSSL` 需要指定为 true，因为前端加密的方法默认是使用 `openSSL` 的

### 错误处理

> 错误处理是指，发生错误时，进行相关错误的日志打印，以及将给前端的错误信息作为方法的返回值

所有的错误处理都放在 `service` 层中，因为这一层是不可能避免处理错误的，就不如统一在这处理；在 `dao` 层处理，处理的错误提示也不能定制，太抽象。

反：`dao` 层处理方便复用，那些真的从数据库层报出的错误，真的没有必要一个一个去重复处理啊。

正：`service` 中可以抽取提出 `dao` 层的同名方法进行校验，调用这个方法复用就行了。

反：那岂不是所有 `dao` 方法都要在 `service` 定义一个同名的错误处理，那导致的额外工作量太大了。

正：就实际开发的经验来说，能复用的 `dao` 方法并不多。

反：......



如果默认 `service` 层中的方法都做了错误处理，`service` 之间的方法调用，都无需再进行错误处理。

## 技术点

### Graceful

当定义了任何 **异步** 的后台任务机制，都需要考虑在程序结束时的任务防丢，Grace 机制就是，当请求没有处理完程序是不会结束的

> beego

配置 `Graceful = true`

> gin

参照，官方样例

### 临时文件管理

这里对一些动态生成的文件进行生成位置上的更改说明，如果你也有将所有临时文件、生成文件放在一个统一文件夹中管理的习惯，可以像下边这样做

- **lastupdate.tmp**

  默认位置：项目目录下

  修改：修改 beego 的源码  `%GOPATH%\pkg\mod\github.com\beego\beego\v2@v2.0.1\server\web\parser.go` 中 `lastupdateFilename` 变量的值

  举例：`target/lastupdate.tmp`

- **go_build_项目名.exe**

  默认位置：项目目录下

  修改：修改项目运行配置中的 `Output directory` 项

  举例：`项目所在目录的绝对路径\target`

- **logs**

  配置位置

  ```go
  logs.SetLogger(logs.AdapterFile, fmt.Sprintf(`{"filename":"target/logs/%s.log", "maxdays":15}`
  ```

### SQL 日志

在 `Beego 2.0`，中如果希望接口服务实例记录输出所有执行的 SQL 日志，那么需要为 `github.com/beego/beego/v2/client/orm.Debug` 赋值为 `true`

特别说明，`Beego` 在更新至 2.0 后，原先许多重要模块都有发生不同程度的改变，其中原来在 1.0 中使用事务的方式（参照最佳实践），到了 2.0 中行为发生了改变，相关的 SQL 不再进行日志打印

核心代码位置

```
# 重要代码位置
github.com/beego/beego/v2@v2.0.1/client/orm/orm.go:477 (o *ormBase) Raw
github.com/beego/beego/v2@v2.0.1/client/orm/orm_raw.go:86 (o *rawSet) Exec

-- 为什么不打印日志？
因为调用了 Begin 方法后，内部，一个事务标识的 bool 值会变为 true，然后最终会导致下面执行 Exec 的实例类型不同，也就产生了打印日志上的行为差异

-- 非事务
Exec 方法 client/orm.(*dbQueryLog).Exec → ExecContext client/orm/orm_log.go:144 打印日志

-- 开启事务
Exec 方法 client/orm.(*TxDB).Exec → ExecContext client/orm/db_alias.go:250 不打印日志
```

> 基于 gin 的基础架构封装，已经思考处理好了这个

## 私有仓库

目前，项目中有打算将基础的方法、模块抽取到一个独立的二方库 `github.com/shanghairanking/helper` 中，这样，就不需要将相同的代码，四处拷贝了，方便统一管理和降低维护成本。对于该私有仓库，如果 `go get`、`go mod` 的相关命令无法拉取，可以考虑一下的解决方法

**方法一**

协议变更 HTTPS → GIT

**方法二**

设置 Go 的环境变量

```
GOPRIVATE=github.com/shanghairanking
GONOPROXY=github.com/shanghairanking
GONOSUMDB=github.com/shanghairanking
```

## 其他文档

https://github.com/uber-go/guide/blob/master/style.md

https://github.com/cristaloleg/go-advice/blob/master/README_ZH.md

https://github.com/qichengzx/gopher-reading-list-zh_CN