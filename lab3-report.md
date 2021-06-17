# 实验三实验报告

**名称**：The Service Layer

**作者**：李玉盈 201831990607，黄诗雅 201831990103，徐坚苗 201831990136

**日期**：2021.6.16

## 摘要

目的：在基于lab2的基础上，将在 `services.py `中实现一个面向`EnglishPal`的`service layer`， 它提供了一个叫做 `read` 的核心`service`。`service`将选择一篇合适的文章供用户阅读。读取的函数用以下四个参数（`user`, `user_repo`,` article_repo`, `session`）作为输入，如果用户已成功分配了要读取的文章，则返回文章 ID

具体内容：补充`service.py`使得`test_services.py`能够成功完成测试

## 方法和材料

我们使用python语言进行代码的编写，采用`SQLAlchemy`的`ORM`而不是用普通的sql语句，根据 [Going to Town on the Message Bus](https://www.cosmicpython.com/book/chapter_09_all_messagebus.html)进行代码的编写

## 结果

**1.service.py**

```python
# Software Architecture and Design Patterns -- Lab 3 starter code
# An implementation of the Service Layer
# Copyright (C) 2021 Hui Lan


# word and its difficulty level
WORD_DIFFICULTY_LEVEL = {'starbucks':5, 'luckin':4, 'secondcup':4, 'costa':3, 'timhortons':3, 'frappuccino':6}


class UnknownUser(Exception):
    def __init__(self,user):
        self.user = user

    def __str__(self):
        print('不正确的用户名或密码: %s' % self.user )

class NoArticleMatched(Exception):
    def __init__(self, level):
        self.level = level

    def __str__(self):
        print('用户的词汇级别为%dlevel，没有匹配的文章' % self.level)


def read(user, user_repo, article_repo, session):
    repo = user_repo.get(user.username)
    # 判断用户名和密码是否相符
    if repo is None:
        raise UnknownUser(user.username)
    else:
        if repo.password != user.password:
            raise UnknownUser(user.password)
        else:
            # 计算用户词汇级别
            word_len = len(repo.newwords)
            user_sum = 0
            if word_len > 3:
                user_newwords_dict = {key: value for key, value in WORD_DIFFICULTY_LEVEL.items() if key in repo.newwords}
                sortedword_level = sorted(user_newwords_dict.items(),key=lambda item:item[1],reverse=True)[:3]
                level = sum(word_tuple[1] for word_tuple in sortedword_level) / 3
            else:
                for item in list(repo.newwords):
                    user_sum += WORD_DIFFICULTY_LEVEL[item.word]
                level = user_sum / word_len
                article_list = article_repo.list()#遍历
                available_article = {}
            # 匹配文章
            for article in article_list:
                if article.level > level:
                    available_article[article.article_id] = article.level
            # 如果有匹配文章，选取文章等级最低的
            if available_article:
                min_level_article = article_repo.get(min(zip(available_article.values(), available_article.keys())) [1])# La最小级
            # 没有匹配文章就抛出异常
            else:
                raise NoArticleMatched(level )
            read_record = repo.read_article(min_level_article)
            session.commit()
            session.close()
            return read_record
```

2.test_services.py

以下为test_services.py的pytest运行结果

![](https://ftp.bmp.ovh/imgs/2021/06/da48b2ecad113448.png)

## 讨论

**1.修改工作的详细解释**

我们的主要工作是补充两个异常类和`read`类

`read`类主要功能是计算用户词汇级别从而获取匹配文章

如果用户名和密码不相符，抛出`UnknownUser`异常

如果没有匹配文章，抛出`NoArticleMatched`异常

计算用户词汇时，如果新单词个数大于3就取等级为3个最难新单词的平均等级，如果新单词个数小于等于3就取等级为全部新单词的平均等级

匹配文章时：如果文章等级大于用户等级，则匹配满足条件的文章中等级最低的文章

**2.关于是否遵循单一职责原则（SRP）**

我们在`service.py`的书写中并没有遵循单一职责原则（SRP）。

不遵循单一职责原则的原因：`read`类的实现逻辑简单，需要的方法较少，可以不遵循单一职责原则，这样在写代码的时候也能很顺畅快速

## 参考

[编写计算机实验报告](https://thehackpost.com/a-brief-guide-how-to-write-a-computer-science-lab-report.html)

[SQLAlchemy 1.4.18](https://pypi.org/project/SQLAlchemy/)

[Events and the Message Bus](https://www.cosmicpython.com/book/chapter_08_events_and_message_bus.html)