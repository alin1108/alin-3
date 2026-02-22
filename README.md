# 第一个FastAPI程序
### 总结：
- 为什么要创建虚拟环境？
1. 隔离项目运行环境 避免依赖冲突，保持全局环境的干净和稳定
- 怎么运行FastAPI项目？
1. run项目
2. uvicorn main:app --reload(--reload:更改代码后自动重启服务器)
- 怎么访问FastAPI交互式文档？
http://127.0.0.1:8000/docs
## FastAPI路由
### 总结：
- 什么是路由？
1. 路由是URL地址和处理函数之间的映射关系
- 说出下方关键代码的含义？
```bash
@app.get("/")
async def root():
    return {"message": "Hello World"}
```
1. 访问 /根路径 响应结果为{"message": "Hello World"}
### 练习
1. 需求：访问路径/user/hello,响应结果是{"msg":"我正在学习FastAPI...."}
```bash
@app.get("/user/hello")
async def get_user_hello():
    return {"msg":"我正在学习FastAPI...."}
```
## 参数的简介和路径参数
### 参数
1. 参数就是客户端发送请求时附带的额外信息和指令
2. 参数的作用是让同一个接口能根据不同的输入，返回不同的输入，实现动态交互
### 参数的分类
1. 路径参数
- 位置：UPL路径的一部分/book/{id}
- 作用：指向唯一的、特定的资源
- HTTP方法：GET
- FastAPI允许为参数声明额外的信息和校验
- 类型注解Path函数
![img.png](img.png)
2. 查询参数
- 位置：UPL?之后k1=v1 & k2=v2
- 作用：对资源集合经行过滤、排序、分页等操作
- HTTP方法：GET
- 函数括号内声明的参数不是路径参数时，路径操作函数会把该参数自动解释为查询参数
- 类型注解Query函数
4. 请求体
- 位置：HTTP请求的消息体(body)中
- 作用：创建、更新资源携带大量数据 如：JSON
- HTTP方法：POST、PUT
- 类型注解Field函数
### 练习
1. 路径参数
- 需求:以用户id为路径设计URL，要求响应结果包含用户id和名称(普通用户id)
```bash
@app.get("/token/{id}")
async def get_id(id:str):
    return {"用户id":id,"title":f"普通用户{id}"}
```
- 需求：把两个路径参数组合成一个接口
```bash
@app.get("/book/{id}/{name}")
async def get_book_name(id:int = Path(...,gt = 0,le = 100,description="书籍id,取值范围1-100"),
                        name:str = Path(...,min_length =  2,max_length = 10,description="这本书作者的名字，长度范围：2-10")):
    return {"name":f"第{id}本书籍作者名字为{name}"}
```
2. 路径参数—类型注解
- 需求：定义两个接口，携带路径参数，并使用Path来实现类型注解
- 具体如下：接口1，以新闻分类id为参数设计URL，范围为1~100.接口2
，以新闻分类名称为参数设计URL，分类名称长度为2~10.
```bash
@app.get("/{NewsCategoryID}")
async def get_NewsCategoryID(NewsCategoryID:int = Path(...,ge = 1,le = 100,description="新闻分类id,范围为1~100")):
    return {"Introduction":f"这个新闻分类id为{NewsCategoryID}"}
@app.get("/{NewsCategoryName}")
async def get_NewsCategoryName(NewsCategoryName:str = Path(...,min_length = 2,max_length = 10,description="新闻分类名称，分类名称长度为2~10")):
    return {"Introduction":f"这个新闻分类名称为{NewsCategoryName}"} 
```
3. 查询参数
- 需求：设计接口查询图书，要求携带两个查询参数：图书分类和价格
- 具体要求如下：
图书分类，默认值为Python开发，长度限制5~255
价格,限制大小范围5~100
```bash
@app.get("/book/book_list")
async def get_book_list(
        category:str = Query("Python开发",min_length = 5,max_length = 255,description="图书分类"),
        price:int = Query(...,gt = 50,lt = 100,description="图书价格50~100")
):
    return {"category":category,"price":price }
```
4. 请求体参数
- 需求：设计接口新增图书，图书信息包含：书名、作者、出版社、价格
```bash
class Books(BaseModel):
    BookTitle: str
    BookAuthor: str
    BookPublisher: str
    BookPrice: int
@app.post("/Book_Information")
async def book_information(book:Books):
    return book
```
5. 请求体参数—类型注解
- Optional 意思是"可选的"、"可有可无的"。它表示一个变量可以是某种类型，也可以是 None（空值）。
- 需求：设计接口新增图书，图书信息包含：书名、作者、出版社、价格
- 具体要求如下：书名,不能为空；长度限制5~20,作者,长度2~10；出版社,默认值"北京大学出版社"
售价,不能为空；价格大于0元
```bash
class Books(BaseModel):
    BookTitle: str = Field(...,description="书名不能为空",min_length=5,max_length=20)
    BookAuthor: str = Field(min_length=2,max_length=10,description="长度2~10")
    BookPublisher: str = Field(default="北京大学出版社",description="出版社默认值")
    BookPrice: int = Field(...,gt = 0,description="价格不能为空,价格大于0元")
@app.post("/Book_Information")
async def book_information(book:Books):
    return book
```
### 参数的简介和路径参数总结
1. 路径参数出现在什么位置？
- URL 路径的一部分 /book/{id}
2. 如何为路径参数添加类型注解？
- Python原生注解和Path注解
3. 查询参数出现在什么位置？
- URL ? 之后 k1=v1 & k2=v2
- ![img_1.png](img_1.png)
4. 如何为查询参数添加类型注解？
- Python原生注解和Query注解
5. 请求体参数的作用是什么？
- 创建、更新资源，向服务器提供资源
6. 如何定义、使用请求体参数？
- ![img_2.png](img_2.png)
7. 如何为请求体参数添加类型注解？
- Python原生注解和Field注解
8. Path,Query,Field函数注解中的...有什么用？
- ...（Ellipsis）是 Python 的特殊常量，在 Pydantic 中专门用来标记字段为必填项—— 如果前端请求体中缺少这个字段，或字段值为None
- FastAPI 会自动返回422 Unprocessable Entity错误，提示 “字段必填”
- 也就是用来约束这个这个参数不能为空(也可以用required=True，和...等价)
## 响应类型
- 默认情况下,FastAPI会自动将路径操作函数返回的Python对象(字典，列表，Pydantic模型等),经由jsonable_encoder()函数转换为JSON兼容格式，
并包装为JSONResponse返回。这省去了手动序列化的步骤，让开发者能更专注于业务逻辑。
如果需要返回非JSON数据(如HTML,文件流)，FastAPI提供了丰富的响应类型来返回不同数据
![img_3.png](img_3.png)
### 响应类型的设置场景
![img_4.png](img_4.png)
### 响应HTML格式
- 设置响应类为HTMLResponse,当前接口即可返回HTML内容
```bash
@app.get("/html",response_class=HTMLResponse)
async def get_html():
    return "<h1>Hello World</h1>"
```
### 响应文件格式
- FileResponse 是FastAPI提供的专门用于高效返回文件内容(如图片，PDF，Excel,音频视频等)的响应类。它能够智能处理文件路径、媒体类型推断、范围请求
和缓存头部、是服务静态文件的推荐方式
```bash
@app.get("/file")
async def get_file():
    path = "./files/测试图片.jpeg"
    return FileResponse(path)
```
### 自定义响应数据格式
- response_model 是路径操作装饰器(如@app.get或@app.post)的关键参数，它通过一个Pydantic模型来严格定义和约束API端点的输出格式。
这一机制在提供自动数据验证和序列化的同时，更是保障数据安全性的第一道防线
- 约定的这一类型，响应的就必须是这个类型
```bash
class News(BaseModel):
    id:int
    title:str
    content:str
@app.get("/news/{id}",response_model= News)
async def get_news(id:int):
    return {
        "id":id
        ,"title":f"这是第{id}篇新闻"
        ,"content":"新闻内容"
    }
```
### FastAPI中,怎么设置响应类型？
- 装饰器中设置响应类和返回响应对象
![img_5.png](img_5.png)
## 异常处理
- 使用前需导包(from FastAPI import HTTPException)
- 对于客户端引发的错误(4xx，如资源未找到、认证失败)，应使用fastapi.HTTPException来中断正常处理流程，
并返回标准错误响应。
- 需求：按id 查询新闻 → 1 - 6
```bash
class News(BaseModel):
    id:int
    title:str
    content:str
@app.get("/news/{id}",response_model = News)
async def get_news(id:int):
    id_list = [1, 2, 3, 4, 5, 6]
    if id not in id_list:
        raise HTTPException(status_code = 404,detail = "新闻id不存在")
    return {
        "id":id
        ,"title":f"这是第{id}篇新闻"
        ,"content":"新闻内容"
    }
```
## 中间件
- 中间件是一个在每次请求进入FastAPI应用时都会被执行的函数
它在请求到达实际的路径操作(路由处理函数)之前运行,并且在响应返回给客户端之前再运行一次
- 中间件：函数顶部使用装饰器@app.middleware("http")
```bash
#request代表请求
#call_next代表传递请求给路径处理函数
@app.middleware("http")
async def middleware1(request,call_next):
    print("中间件1,start")
    response = await call_next(request)
    print("中间件1,end")
    return response

@app.middleware("http")
async def middleware2(request,call_next):
    print("中间件2,start")
    response = await call_next(request)
    print("中间件2,end")
    return response
@app.get("/")
#@FastAPI实例.HTTP请求方法（请求路径）
async def root():
    return {"message": "Hello World888"}
```
- 运行结果：
![img_6.png](img_6.png)
- 调换中间件位置重新运行
```bash
@app.middleware("http")
async def middleware2(request,call_next):
    print("中间件2,start")
    response = await call_next(request)
    print("中间件2,end")
    return response

@app.middleware("http")
async def middleware1(request,call_next):
    print("中间件1,start")
    response = await call_next(request)
    print("中间件1,end")
    return response
@app.get("/")
#@FastAPI实例.HTTP请求方法（请求路径）
async def root():
    return {"message": "Hello World888"}
```
- 运行结果：
![img_7.png](img_7.png)
- 理解逻辑：大致就是一个洋葱模型，自下而上执行
### 中间件作用是什么？
1. 为每个请求添加统一的处理逻辑(记录日志、身份认证、跨域、设置响应头、性能监控等)
### 中间件怎么定义?
1. 函数顶部使用装饰器@app.middleware("http")
![img_8.png](img_8.png)
### 多个中间件的执行顺序？
自下而上
## 依赖注入系统
1. 使用依赖注入系统来共享通用逻辑，减少代码重复
2. 依赖项：可重用的组件(函数/类)，负责提供某种功能或数据
3. 注入：FastAPI自动帮你调用依赖项，并将结果"注入"到路径操作函数中
### 中间件和依赖注入的差别
- 中间件是控制所有请求，每个请求都会自动运行中间件封装的代码
- 依赖注入是人作主，哪些请求需要使用这些通用的代码就注入哪些请求
### 优点：
- 代码复用：一次编写，多次使用
- 解耦：业务逻辑与基础设施代码分离
- 易于测试：轻松地用模拟依赖替换真是依赖进行测试
### FastAPI中依赖注入系统有什么用？
- 出去可复用的组件，实现代码的复用、解耦且可轻松替换依赖项进行测试
### 怎么使用依赖注入系统？
- 步骤
1. 创建依赖项
2. 导入Depends 
3. 声明依赖项
- 操作实例
```bash
#导入Depends
from fastapi import Depends
#创建依赖项
async def common_parameters(
        skip:int = Query(0,ge = 0),
        limit:int = Query(10,le = 60)
        ):
    return {"skip":skip,"limit":limit}
#声明依赖项
@app.get("/news/news_list")
async def get_news_list(commons = Depends(common_parameters)):
    return commons
@app.get("/user/user_list")
async def get_user_list(commons = Depends(common_parameters)):
    return commons
```
# ORM 简介
- ORM(对象关系映射)是一种编程技术，用于在面向对象编程语言和关系型数据库之间建立映射。
它允许开发者通过操作对象的方式与数据库进行交互，而无需直接编写复杂的SQL语句。
- 优势
1. 减少重复的SQL代码
2. 代码更简洁易读
3. 自动处理数据库链接和事务
4. 自动防止SQL注入攻击
## ORM使用流程
1. 安装
- sqlalchemy[asyncio]
- aiomysql(异步数据库驱动)
- pip install sqlalchemy[asyncio] aiomysql
2. 建库、建表
- 让数据库按照我们定义的表结构，把所有的表都建出来：run_sync(Base.metadata.create_all)
- 创建数据库引擎
- 定义模型类
- 启动应用时建表
3. 操作数据
- 增删改查
### 创建数据库引擎
1. 导入create_async_engine模块
- 先导包(from sqlalchemy.ext.asyncio import create_async_engine)
2. 数据库的链接
- 例子：
```bash
ASYNC_DATABASE_URL = "数据库+驱动://用户名:密码@localhost:端口/数据库的名字?charset=utf8mb4"
ASYNC_DATABASE_URL = "mysql+aiomysql://root:123456@localhost:3306/fastapi_test?charset=utf8mb4"
```
3. 使用create_async_engine创建异步引擎
```bash
async_engine = create_async_engine(
  ASYNC_DATABASE_URL, #数据库URL
  echo = True,  #可选：输出SQL日志
  pool_size = 10,  #设置连接池中保持的持久连接数
  max_overflow = 20 #设置连接池允许创建的额外连接数
)
```
### 定义模型类
1. 基类,继承DeclarativeBase(包含通用属性和字段映射)
- 导入DeclarativeBase模块(from sqlalchemy.orm import DeclarativeBase,Mapped,mapped_column)
- 顺便导入Mapped(约定属性类型的映射),mapped_column模块(映射字段)
- Mapped[datetime]：SQLAlchemy 2.0 的类型注解语法

- mapped_column()：声明式列定义函数

- DateTime：SQLAlchemy 的列类型，映射到数据库的日期时间类型

- insert_default：INSERT 时的数据库默认值

- default：Python 对象的默认值（可调用对象）

- onupdate：UPDATE 时自动更新的值

- func.now()：数据库时间函数（带括号调用）

- func.now：Python 时间函数（函数引用，无括号)
```bash
#基类：创建时间和更新时间
class Base(DeclarativeBase):
    create_time:Mapped[datetime] = mapped_column(DateTime, insert_default=func.now(),default=func.now,comment="创建时间")
    update_time:Mapped[datetime] = mapped_column(DateTime, insert_default=func.now(),default=func.now,onupdate=func.now(),comment="修改时间")
```
2. 定义数据库表对应的模型类
- 这个模型类要继承基类: Base
- 要设置表名：__tablename__ = "表名"
- 要设置字段：字段名：Mapped[字段类型] = mapped_column(字段类型,字段属性)
- String：SQLAlchemy 的字符串类型
- (255)：长度限制，255 个字符
- 对应数据库中的 VARCHAR(255)
```bash
class Book(Base):
    #表名
    __tablename__ = "book"

    id:Mapped[int] = mapped_column(primary_key=True,comment="书籍id")
    bookname:Mapped[str] = mapped_column(String(255),comment="书籍名称")
    author:Mapped[str] = mapped_column(String(255),comment="作者")
    price:Mapped[float] = mapped_column(Float,comment="价格")
    publisher:Mapped[str] = mapped_column(String(255),comment="出版社") 
```
### ORM-创建数据库表
1. 从连接池获取异步链接，开启事务，执行ORM操作
2. FastAPI应用启动时，创建数据库表
- create_tables() 函数作用：创建表
- async with async_engine.begin() as conn:(创建一个连接对象我们称他为conn)
- await 关键字 作用：等待异步操作完成 原理：挂起当前协程，直到异步操作返回结果
- async with：确保连接在使用后正确关闭，即使发生异常
- .begin()：开启一个数据库事务，返回一个异步连接对象
- as conn：把这个连接命名为conn
- conn.run_sync(Base.metadata.create_all)意思：让数据库按照我们定义的表结构，把所有的表都建出来
- conn.run_sync()：异步连接执行同步方法的桥梁
- 因为 SQLAlchemy 的 create_all() 方法是同步的
- Base.metadata：包含所有注册到 Base 的模型元数据
- create_all()：生成并执行 CREATE TABLE 语句
- 导包(from contextlib import asynccontextmanager)
- @asynccontextmanager 作用：用于将一个异步生成器函数转换为异步上下文管理器
这样我们就可以在 FastAPI 应用启动和关闭时执行一些代码
- lifespan：用于处理应用的生命周期事件（启动和关闭）
- (app: FastAPI)告诉 PyCharm：app 是 FastAPI 实例
- 类型提示：app: FastAPI 提供 IDE 类型检查和自动补全
- yield 就像一个"开关"，它把 lifespan 函数分为：启动准备 → 运行应用 → 关闭清理 三个阶段
- .dispose() 是 SQLAlchemy 作用：它负责清理并释放所有数据库连接池中的连接资源
- app = FastAPI(lifespan=lifespan)：创建一个 FastAPI 应用实例，并传递生命周期管理器给它
- autoincrement=True(必填的意思)
```bash
async def create_tables():
    async with async_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 应用启动时执行 create_tables()这个创建表的异步函数
    await create_tables()
    # 应用运行中
    yield
    # 应用关闭时清理资源
    await async_engine.dispose()
    
app = FastAPI(lifespan=lifespan)
```
#### 练习
- 需求：使用SQLAlchemy ORM创建用户表
- 包含的字段：用户id、用户名、密码、创建时间、更新时间
```bash
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy import DateTime, func, String, Float
from sqlalchemy.ext.asyncio import create_async_engine
from sqlalchemy.orm import DeclarativeBase,Mapped,mapped_column
from datetime import datetime
app = FastAPI()
# 需求：使用SQLAlchemy ORM创建用户表
# - 包含的字段：用户id、用户名、密码、创建时间、更新时间

#1.创建数据库连接
ASYNC_DATABASE_URL = "mysql+aiomysql://root:123456@localhost:3306/fastapiproject99?charset=utf8mb4"
#2.创建异步引擎
async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    echo= True, # 打印SQL语句
    pool_size = 20, # 连接池大小
    max_overflow = 10,  #额外连接数
)
# 3.定义基类
class Base(DeclarativeBase):
    create_time:Mapped[datetime] = mapped_column(
        DateTime,insert_default=func.now(),default=func.now,comment="创建时间"
    )
    update_time:Mapped[datetime] = mapped_column(
        DateTime,insert_default=func.now(),default=func.now,onupdate=func.now(),comment="更新时间"
    )

#4.定义模型类
class Users(Base):
    #表名
    __tablename__ = "users"
    userid:Mapped[int] = mapped_column(primary_key=True,autoincrement=True,comment="用户id")
    username:Mapped[str] = mapped_column(String(255),comment="用户名")
    password:Mapped[str] = mapped_column(String(255),comment="密码")

#5.启动应用时建表
#5.1 建表函数
async def create_table():
    async with async_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

#5.2 启动应用时执行建表函数
@asynccontextmanager
async def lifespan(app: FastAPI):
    await create_table()    #启动应用前建表
    yield   #正在运行
    await async_engine.dispose()    #关闭清理

app = FastAPI(lifespan=lifespan)
```
## ORM - 路由匹配中使用ORM 
- 核心：创建依赖项，使用Depends注入到处理函数
1. 创建异步会话工厂(先导入工厂函数和指定的会话类from sqlalchemy.ext.asyncio import async_sessionmaker,AsyncSession)
2. 创建依赖项，用于获取数据库会话
3. 使用Depends将依赖项注入到处理函数
- 需求：#需求：查询功能的接口，查询图书→ 依赖注入：创建依赖项获取数据库会话+Depends 注入路由处理函数
1. 通过async_sessionmaker创建会话工厂
- AsyncSession 的特点：
- 支持 async/await 语法
- 可以配合 asyncio 使用
- 提供异步的数据库操作方法
- class_=AsyncSession：指定要创建的会话类为AsyncSession，这是SQLAlchemy提供的异步会话类
- expire_on_commit=False：提交事务后，会话中的对象不失效，下次运行会直接输出缓存的值，不重新查询
```bash
AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,  # 绑定数据库异步引擎
    class_=AsyncSession,    #指定会话类
    expire_on_commit = False    #保证提交后会话不过期，重新运行时不会重新查询数据库
)
```
2. 创建依赖项，用于获取数据库会话
- get_database()：作用：为每个HTTP请求提供标准化的会话管理
- async with AsyncSessionLocal() as session:(意思是在会话工厂获取一个会话)
- yield 实现会话的"借用"模式(保证调用完之后能够返回来执行yield之后的操作(如提交事务，资源释放等))
- session.commit()：提交事务(确认这操作都生效)
- session.rollback()：回滚事务(撤销之前执行的操作或更改,以恢复到某个已知的稳定状态)
- session.close()：关闭会话(# 注意：async with 也会调用 close()，所以这里是双重保证,但在复杂的异常处理中，显式关闭是好习惯)
```bash
async def get_database():
    async with AsyncSessionLocal() as session:
        try:
            yield session   # 返回数据库会话给路由处理函数
            # ⏸️ 在这里暂停，等待路由函数用完
            # 路由函数用完后，回到这里继续执行
            await session.commit()  #提交事务
        except Exception as e:
            await session.rollback()    #有异常,回滚事务
            raise e #抛出异常
        finally:
            await session.close()   #显式关闭,确保资源释放,不管运行成功与否关闭会话
```
3. 创建一个查询图书的接口
- db:AsyncSession - 类型注解，说明参数类型
- AsyncSession 是一个特殊的会话类，支持异步操作
- db.execute()执行SQL语句
- .execute() 是异步方法
- .scalars() 的作用：从行中提取标量值经过 
- .scalars()：去掉元组包装，变成 Book 对象
- .all() 的作用：获取所有结果并转换为列表
```bash
@app.get("/book/books")
async def get_book_list(db:AsyncSession = Depends(get_database)):
    #  查询
    result = await db.execute(select(Book))
    book = result.scalars().all()
    return book
```
### 练习
- 需求：创建一个查询用户id、用户名、密码的接口
```bash
from contextlib import asynccontextmanager
from fastapi import FastAPI,Depends
from sqlalchemy import DateTime, func, String, select
from sqlalchemy.ext.asyncio import create_async_engine,async_sessionmaker,AsyncSession
from sqlalchemy.orm import DeclarativeBase,Mapped,mapped_column
from datetime import datetime
#1.通过async_sessionmaker创建会话工厂
async_session = async_sessionmaker(
    bind = async_engine,    # 绑定引擎
    class_= AsyncSession,   # 指定的会话类：异步会话
    expire_on_commit = False    # 提交后不重置
)

#2.创建依赖项(让路由处理函数能够获取数据库会话)
async def get_db():
    async with async_session() as session:  #从数据库会话工厂中获取数据库会话
        try:
            yield session   #⏸️ 暂停！把数据库的会话传给路由处理函数
            await session.commit()  #路由处理函数用完会话后，把会话提交事务
        except Exception as e:
            await session.rollback()    #如果出现异常，回滚事务
            raise e
        finally:
            await session.close()   #显式型关闭会话

#3.创建带查询数据的路由处理函数
@app.get("/users/user")
async def get_user_list(db:AsyncSession = Depends(get_db)): #注入依赖项
    result = await db.execute(select(Users))  #await执行查询操作
    user = result.scalars().all()
    return user
```
## ORM数据库操作 - 查询
- 核心语句：await db.execute(select(模型类)),返回一个ORM对象
1. 获取所有数据
- scalars().all()
```bash
result = await db.execute(select(Book))
book = result.scalars().all()
```
2. 获取单条数据
- scalars().first()
```bash
result = await db.execute(select(Book))
book = result.scalars().first()
```
- get(模型类,主键值)
```bash
# 获取主键值(id)为4的图书(这种查询方式不需要await db.execute(select(Book)))
book = await db.get(Book,4)
```
## ORM数据库操作 - 查询条件
- 核心语句：select(模型类).where(条件,条件2)
- 条件：
1. 比较判断：==,>,<,>=,<=,!=等
- result.scalar_one_or_none():
这个方法的名字就是它的功能描述：
scalar（标量） + one（一个） + or_none（或者空）
找到了符合条件的一条记录,返回该记录，没有则返回None
```bash
#需求：路径参数是书籍的id 用户输哪本的id,就返回该id的图书信息
@app.get("/book/get_book/{book_id}")
async def get_book_list(book_id:int,db:AsyncSession = Depends(get_database)):
   result = await db.execute(select(Book).where(Book.id == book_id))
   book = result.scalar_one_or_none()
   if book is None:
       raise HTTPException(status_code = 404,detail = f"id为{id}的图书不存在")
   return book
#需求：条件 价格大于等于600
@app.get("/book/search_book")
async def get_search_book(db:AsyncSession = Depends(get_database)):
    result = await db.execute(select(Book).where(Book.price >= 600))
    books = result.scalars().all()
    return books
```
2. 模糊查询：like()
- 写法：select(模型类).where(模型类.字段名.like("查询条件"))
- %：任意个字符
```bash
@app.get("/book/search_book")
async def get_search_book(db:AsyncSession = Depends(get_database)):
    result = await db.execute(select(Book).where(Book.author.like("曹%")))
    book = result.scalars().all()
    return book
    #结果：曹植和曹雪芹
```
- _：单个字符
```bash
@app.get("/book/search_book")
async def get_search_book(db:AsyncSession = Depends(get_database)):
    result = await db.execute(select(Book).where(Book.author.like("曹_")))
    book = result.scalars().all()
    return book
    #结果：曹植
```
3. 与或非查询：&；|；~  
```bash
@app.get("/book/search_book")
async def get_search_book(db:AsyncSession = Depends(get_database)):
    result = await db.execute(select(Book).where((Book.author.like("曹%")) & (Book.price>200)))
    book = result.scalars().all()
    return book
```
```bash
@app.get("/book/search_book")
async def get_search_book(db:AsyncSession = Depends(get_database)):
    result = await db.execute(select(Book).where(~ Book.author.like("曹%")))
    book = result.scalars().all()
    return book
```
4. 包含查询：in_()
- 写法：select(模型类).where(模型类.字段名.in_(包含条件))
```bash
async def get_search_book(db:AsyncSession = Depends(get_database)):
    id_list = [1, 3, 5, 7]
    result = await db.execute(select(Book).where(Book.id.in_(id_list)))
    book = result.scalars().all()
    return book
```
5. 或者的条件连接：or_()
- or_() 是 SQLAlchemy 的一个函数，用于创建 SQL 的 OR 条件
### 练习：
- 需求：实现username中的一部分包含了username_list的元素的查询接口
```bash
@app.get("/users/user")
async def get_user_list(db:AsyncSession = Depends(get_db)): #注入依赖项
    username_list = ["三","四","五","六"]
    conditions = [] #条件列表
    for name in username_list:  #遍历关键字
        #   为每个关键字创建一个like条件
        condition = Users.username.like(f"%{name}%")#表示"用户名包含'三'"
        conditions.append(condition)
    #*conditions：Python的展开语法，相当于：
    #or_(conditions[0], conditions[1], conditions[2], conditions[3])
    result = await db.execute(select(Users).where(or_(*conditions)))
    users = result.scalars().all()
    return users
```
## ORM数据库操作 - 聚合查询
- 聚合计算：func.方法(模型类.属性)
- count:统计行数量
- avg:求平均值
- max:求最大值
- min:求最小值
- sum:求和
- 注意：聚合查询的出来的结果用scalar()方法来处理数据
- 因为聚合查询返回一个值，不是列表，所以用scalar()方法来处理数据
```bash
@app.get("/book/get_book_count")
async def get_book_count(db: AsyncSession = Depends(get_database)):
    result = await db.execute(select(func.count(Book.id)))
    book_count = result.scalar()
    return book_count
```
## ORM数据库操作 - 分页查询
- 分页查询：select().offset().limit()
- offset:跳过的记录数
- limit:每页的记录数
- offset值 = (当前页码page - 1) * 每页数量limit
- 需求：实现分页查询
```bash
@app.get("/book/get_book_list")
async def get_book_list(
        page:int = Query(1,ge = 1,le = 10,description="页码" ),
        page_size:int = Query(3,ge = 1,le = 10,description="每页的记录数量"),
        db:AsyncSession = Depends(get_database)
):
    skip = (page - 1) * page_size     #  每页跳过的记录数量
    stmt = select(Book).offset(skip).limit(page_size)   #ORM分页查询语句
    result = await db.execute(stmt)     #执行ORM查询语句
    book_list = result.scalars().all()  #处理查询到的结果
    return book_list    #   输出处理后的结果
```
## ORM - 查询 - 总结
- 核心思路：
- select()→db.execute() → 从ORM对象获取数据 → 根据数据类型选择采用scalars()或者scalar()方法处理数据 → 响应结果
- db.get(模型类,主键值)
- 查询类型：条件查询,聚合查询,分页查询
## ORM数据库操作 - 新增数据
- 核心步骤：定义ORM对象→添加对象到事务：add(对象)→commit提交数据库
- __dict__:是什么：Python 对象的属性字典；用途：查看、操作对象的属性；包含什么：实例的所有自定义属性（不包括类属性、方法）
- book_obj = Book(**book.__dict__)：将 Pydantic 模型转换为 SQLAlchemy ORM 对象
- **：展开操作符，将字典的键值对展开为多个参数
- db.add(book_obj)：将 ORM 对象添加到数据库会话
- add():db.add() 不会立即将数据保存到数据库它只是将对象标记为"待插入"
- await db.commit():提交事务，将数据真正保存到数据库
```bash
class BookBase(BaseModel):
    id:int
    bookname:str
    author:str
    price:float
    publisher:str
@app.post("/book/add_book")
async def add_book(book:BookBase,db: AsyncSession = Depends(get_database)):
    #ORM对象 → add → commit
    book_obj = Book(**book.__dict__)
    db.add(book_obj)
    await db.commit()
    return book
```
## ORM数据库操作 - 更新数据
- 核心步骤：查询get → 属性重新赋值 → commit提交到数据库
- Optional 意思是"可选的"、"可有可无的"。它表示一个变量可以是某种类型，也可以是 None（空值）。
- from typing import Optional
- 如果是post创建资源那么所有字段都是必填的,不用optional
- 如果是put修改资源那么所有字段都是可选的,必须用optional
```bash
#需求：修改图书的信息:先查再改
#设计思路：路径参数：书籍id:作用是查找；请求体参数：作用是修改图书信息(书名、作者、价格、出版社)

#请求体
class BookUpdate(BaseModel):
    bookname:Optional[str] = None   #可以只更新书名
    author:Optional[str] = None     #可以只更新作者
    price:Optional[float] = None    #可以只更新价格
    publisher:Optional[str] = None  #可以只更新出版社

#接口
@app.put("/books/update_book/{book_id}")
async def update_book(
        book_id:int,
        data:BookUpdate,
        db: AsyncSession = Depends(get_database)
):
    #查询图书
    result = await db.execute(select(Book).where(Book.id == book_id))
    db_book = result.scalar_one_or_none()
    #如果没找到 抛出异常
    if db_book is None:
        raise HTTPException(
            status_code=404,
            detail="id not found"
        )
    # 重新赋值(只更新客户端提供的字段)
    if data.bookname is not None:
        db_book.bookname = data.bookname
    if data.author is not None:
        db_book.author = data.author
    if data.price is not None:
        db_book.price = data.price
    if data.publisher is not None:
        db_book.publisher = data.publisher
    #提交事务
    await db.commit()
    return db_book
```
## ORM数据库操作 - 删除
- 核心步骤：查询get → delete删除 → commit提交到数据库
```bash
@app.delete("/books/delete_book/{book_id}")
async def delete_book(book_id:int,db:AsyncSession = Depends(get_database)):
    #查询
    result = await db.execute(select(Book).where(Book.id == book_id))
    db_book = result.scalar_one_or_none()
    if db_book is None:
        raise HTTPException(status_code=404,detail="图书不存在")
    await db.delete(db_book)  #删除方法
    await db.commit()
    return {"message": "删除成功"}
```
## ORM数据库操作 - 总结
- 安装：pip install sqlalchemy[asyncio],pip install aiomysql
- 建库：create database 数据库名;
- 建表：
1. 连接数据库：ASYNC_DATABASE_URL = "数据库+驱动://用户名:密码@localhost:端口/数据库的名字?charset=utf8mb4"
2. 创建异步引擎：create async_engine(ASYNC_DATABASE_URL)(参数：echo= True,pool_size=20,max_overflow=10)
3. 定义基类：class Base(DeclarativeBase)
4. 定义模型类：class Book(Base)
5. 建表：run.sync(Base.metadata.create_all)
6. 使用异步上下文管理器调用建表函数：@asynccontextmanager
- 操作数据：
1. 创建会话工厂：async_sessionmaker(bind = 异步引擎,class_=AsyncSession(指定会话类),expire_on_commit=False(保证提交后会话不过期，重新运行时不会重新查询数据库))
2. 创建依赖项：async with AsyncSessionLocal() as session(用来获取数据库会话)
3. 使用路由处理函数注入数据库会话依赖项(Depends)
4. 查询数据：select()
5. 增加数据：add()
6. 更新数据：先查询再重新赋值
7. 删除数据：delete()