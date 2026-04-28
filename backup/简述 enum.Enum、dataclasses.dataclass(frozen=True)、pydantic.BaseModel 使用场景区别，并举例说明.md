```python
import datetime
from dataclasses import dataclass
from datetime import datetime
from enum import Enum

from pydantic import BaseModel


# enum.Enum
# 使用场景：
#     当你需要定义一组具有唯一性的命名常量时，例如状态码、选项值或具有特定意义的符号。
#     用于表示一组互斥的选项或类型。
class StatusCode(Enum):
    SUCCESS: int = 200
    NOT_FOUND: int = 404
    SERVER_ERROR: int = 500


# 使用
print(StatusCode.SUCCESS.name)  # SUCCESS
print(StatusCode.SUCCESS.value)  # 200
# StatusCode.SUCCESS = 2333  # AttributeError: cannot reassign member 'SUCCESS'

status = StatusCode.SUCCESS
print(status.name)  # SUCCESS
print(status.value)  # 200

# status.value = 2333  # AttributeError: <enum 'Enum'> cannot set attribute 'value'
# print(status.value)


# dataclasses.dataclass(frozen=True)
# 使用场景：
#     当你需要创建轻量级的数据容器，特别是当类的主要目的是存储数据而不是提供方法时。
#     用于定义记录或实体，其中数据在创建后不应被修改。
@dataclass(frozen=True)
class Point:
    x: int = 0
    y: int = 0


# 使用
print(Point.x)  # 0
print(Point.y)  # 0
Point.x = (
    2333  # 这里你试图修改类字段，这是可以的，因为类字段是静态的，不会影响实例的状态。
)
print(Point.x)  # 2333

p1 = Point()
print(p1.x)  # 0 因为实例字段没有创建副本，所以它们是静态的
print(p1.y)  # 0

p2 = Point(x=1, y=2)
print(p2.x)  # 1
print(p2.y)  # 2
# p2.x = 3  # dataclasses.FrozenInstanceError: cannot assign to field 'x'
# print(p2.x)

p3 = Point(
    x="not a number", y="not a number"
)  # dataclass 没有动态类型检查，所以这里不会报错
print(p3.x)  # not a number
print(p3.y)  # not a number

# pydantic.BaseModel
# 使用场景：
#     用于数据验证和设置类型注解，特别是在Web应用程序和服务中，需要确保输入数据的有效性和类型安全。
#     当你需要自动将输入数据转换为正确的类型，并提供详细的错误信息时。


class User(BaseModel, frozen=True):
    name: str = "name"
    age: int = 0


# 使用
# print(User.name)  # raise AttributeError(item)
# print(User.age)  # raise AttributeError(item)


user = User(name="Alice", age=30)
print(user.name)
print(user.age)

# 尝试修改age字段，这将引发异常，因为模型是不可变的
# user.age = "thirty"  # raise pydantic_core.ValidationError.from_exception_data(self.__class__.__name__, [error])  pydantic_core._pydantic_core.ValidationError: 1 validation error for User age
# print(user.age)

# user = User(name="Bob", age="not a number")  # pydantic_core._pydantic_core.ValidationError: 1 validation error for User age  Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='not a number', input_type=str]    For further information visit https://errors.pydantic.dev/2.8/v/int_parsing
# print(user.age)
# print(user.name)


# 同时使用enum.Enum、dataclasses.dataclass和pydantic.BaseModel
# 使用enum.Enum定义文章状态
import enum
from dataclasses import dataclass, field

from pydantic import BaseModel


# 使用enum.Enum定义文章状态
class ArticleStatus(enum.Enum):
    DRAFT = "draft"
    PUBLISHED = "published"
    ARCHIVED = "archived"


# 使用pydantic.BaseModel定义文章元数据
class ArticleMetadata(BaseModel):
    title: str = "title"
    content: str = "content"
    author: str = "author"
    status: ArticleStatus = ArticleStatus.DRAFT


# 使用dataclasses.dataclass定义文章
@dataclass(frozen=True)
class Article:
    id: int = 0
    metadata: ArticleMetadata = field(default_factory=lambda: ArticleMetadata())
    create_datetime: datetime = datetime.now()


# 使用
# 创建文章元数据实例
metadata = ArticleMetadata(
    title="Python学习笔记",
    content="笔记内容",
    author="user0",
    status=ArticleStatus.PUBLISHED,
)

# metadata2 = ArticleMetadata(
#     title=2333,  # ❗ not a string
#     content="笔记内容",
#     author="user0",
#     status=ArticleStatus.PUBLISHED,
# )  # 🤢 pydantic_core._pydantic_core.ValidationError: 1 validation error for ArticleMetadata

# 创建文章实例
article = Article(id=1, metadata=metadata)

# 打印文章信息
print(f"Article ID: {article.id}")
print(f"Article Title: {article.metadata.title}")
print(f"Article Content: {article.metadata.content}")
print(f"Article Author: {article.metadata.author}")
print(f"Article Status: {article.metadata.status.value}")
print(f"Article Create Time: {article.create_datetime}")

# article.id = 2333  # 🤢 dataclasses.FrozenInstanceError: cannot assign to field 'id'
article.metadata.title = "JAVA学习笔记"  # 这里可以修改，因为BaseModel未设置frozen=True
article.metadata.content = (
    "笔记内容笔记内容笔记内容"  # 这里可以修改，因为BaseModel未设置frozen=True
)
article.metadata.author = "user1"  # 这里可以修改，因为BaseModel未设置frozen=True
article.metadata.status = (
    ArticleStatus.ARCHIVED
)  # 这里可以修改，因为BaseModel未设置frozen=True
# article.metadata.status.value = "archived"  # 🤢 raise AttributeError(AttributeError: <enum 'Enum'> cannot set attribute 'value'
# article.create_datetime = datetime(
#     2024, 1, 1, 0, 0, 0
# )  # 🤢 dataclasses.FrozenInstanceError: cannot assign to field 'create_datetime'

# 打印文章信息
print(f"Article ID (frozen dataclass, 不能修改): {article.id}")
print(
    f"Article Title (pydantic.BaseModel 未设置frozen=True, 可以修改): {article.metadata.title}"
)
print(
    f"Article Content (pydantic.BaseModel 未设置frozen=True, 可以修改): {article.metadata.content}"
)
print(
    f"Article Author (pydantic.BaseModel 未设置frozen=True, 可以修改): {article.metadata.author}"
)
print(
    f"Article Status (enum.Enum status.value不能修改, 但是status可以改变): {article.metadata.status.value}"
)
print(f"Article Create Time (frozen dataclass, 不能修改): {article.create_datetime}")
```