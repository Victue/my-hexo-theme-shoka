---
title: selenium收集知乎用户回答
author: Li Pie
categories: 摸鱼
tags: selenium,知乎
---

知乎对selenium有检测，不显示用户回答

更改window.navigator.webdriver属性绕过检测，即可正常显示用户回答。

```python
profile = webdriver.FirefoxProfile()
profile.set_preference("dom.webdriver.enabled", False)
```



用户页面

```python
driver.get('https://www.zhihu.com/people/Victue/answers')
```



问题及回答列表

```python
QAs = driver.find_elements_by_class_name('List-item')
```



问题

```python
QA.find_element_by_css_selector('h2.ContentItem-title>div>a').text
```



阅读全文按钮

```python
QA.find_element_by_css_selector('div.RichContent-inner')
```



回答内容（粗暴的显示所有文字，后续考虑保存为富文本）

```python
QA.find_element_by_css_selector('div.RichContent-inner').text
```



赞同数量

```python
QA.find_element_by_css_selector('button.VoteButton').text
```



评论数量

```python
QA.find_element_by_css_selector('button.ContentItem-action').text
```



下一页

```python
next_page = driver.find_element_by_css_selector('button.PaginationButton-next')
```

