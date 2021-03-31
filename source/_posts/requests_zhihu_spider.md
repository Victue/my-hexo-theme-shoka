---
title: 再爬一次知乎，x-zse-86 2.0
author: Li Pie
categories: 摸鱼
tags: requests
date: 2021-03-30
---

上一次使用selenium爬知乎，太慢。并且不知道为啥，上次使用的方法已经失效，用户回答无法显示，可能是由于知乎升级了反爬机制。

这一次使用requests，正经的爬一次。

显然可知，用户回答api为

```
https://www.zhihu.com/api/v4/members/people/answers?include=data[*].is_normal,admin_closed_comment,reward_info,is_collapsed,annotation_action,annotation_detail,collapse_reason,collapsed_by,suggest_edit,comment_count,can_comment,content,editable_content,attachment,voteup_count,reshipment_settings,comment_permission,mark_infos,created_time,updated_time,review_info,excerpt,is_labeled,label_info,relationship.is_authorized,voting,is_author,is_thanked,is_nothelp,is_recognized;data[*].vessay_info;data[*].author.badge[?(type=best_answerer)].topics;data[*].question.has_publishing_draft,relationship&offset=0&limit=20&sort_by=created
```

json格式

![image-20210331111420437](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210331111420437.png)

三个变量，people，offset，limit

直接获取这个json即可

requests(api)出现403

![image-20210330192118529](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210330192118529.png)

好家伙，header改了很多都是403

分析分析

关键在于这个x-zse-86

![image-20210331111511511](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210331111511511.png)

![image-20210318162945024](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210318162945024.png)

y.set('x-zse-86', '2.0_' + j)

![image-20210318163027298](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210318163027298.png)

b.set('x-zse-86', '2.0_' + E)

![image-20210318163329235](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210318163329235.png)

![image-20210318163419758](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210318163419758.png)

signature: (0, o.default) ((0, r.default) (p))

signature: (0, o.default) ((0, r.default) (d))

打个断点看看d是啥

![image-20210318164344172](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210318164344172.png)

![image-20210318164411666](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210318164411666.png)

d值为

"3_2.0+

/api/v4/members/MarryMea/answers?include=data%5B*%5D.is_normal%2Cadmin_closed_comment%2Creward_info%2Cis_collapsed%2Cannotation_action%2Cannotation_detail%2Ccollapse_reason%2Ccollapsed_by%2Csuggest_edit%2Ccomment_count%2Ccan_comment%2Ccontent%2Ceditable_content%2Cattachment%2Cvoteup_count%2Creshipment_settings%2Ccomment_permission%2Cmark_infos%2Ccreated_time%2Cupdated_time%2Creview_info%2Cexcerpt%2Cis_labeled%2Clabel_info%2Crelationship.is_authorized%2Cvoting%2Cis_author%2Cis_thanked%2Cis_nothelp%2Cis_recognized%3Bdata%5B*%5D.vessay_info%3Bdata%5B*%5D.author.badge%5B%3F(type%3Dbest_answerer)%5D.topics%3Bdata%5B*%5D.question.has_publishing_draft%2Crelationship&**offset=0**&**limit=20**&sort_by=created+

"ALAd3ohjFRKPTjNZeG5_W8KUwFVpuCzndOk=|1603453023""

最后字符串为cookie中d_c0的值

简单思索了一下，使用md5加密这一长串试试，巧了，正好一样

![image-20210330161212333](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210330161212333.png)

第一步加密

![image-20210330161224328](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210330161224328.png)



第二步知乎js加密

https://github.com/hellokuls/python-course/blob/master/zhihu/g_encrypt.js



大功告成，可以去写爬虫收集你心爱用户的回答了。



```
# 加密函数
def encrypt(d):
    d_md5 = hashlib.new('md5', d.encode()).hexdigest()
    with open('zhihu1.js', 'r') as f:
        ctx = execjs.compile(f.read(), 'C:/Users/hasee/node_modules')
    encrypt_d = ctx.call('b', d_md5)
    return encrypt_d
```

```
# api地址以及未加密文本
def address(offset, limit, hash_url, main_url):
    hash_url = hash_url.format(offset, limit)
    url = ''.join([main_url, hash_url])
    d = '+'.join(['3_2.0', hash_url, d_c0])
    return url, d
```

```
# 获取页面内容
def get(offset, limit, hash_url, main_url):
    url, d = address(offset, limit, hash_url, main_url)
    encrypt_d = encrypt(d)
    headers = {
        'user-agent': User_Agent,
        'cookie': cookie,
        'x-zse-83': '3_2.0',
        'x-zse-86': '2.0_%s' % encrypt_d
    }

    r = requests.get(url=url, headers=headers)
    answer = json.loads(r.text)
    return answer
```

```
# 下一页，很粗暴的offset+=20
def next_page(offset):
    offset += 20
    return offset
```



answer就是一个json文件，从中可以很简单找到需要的内容

```
# 问题内容
answer['data'][i]['question']['title']
# 问题id
answer['data'][i]['question']['id']

# 回答内容
answer['data'][i]['content']
# 回答id
answer['data'][i]['id']

# 创建时间
answer['data'][i]['created_time']
# 修改时间
answer['data'][i]['updated_time']

# 点赞数
answer['data'][i]['voteup_count']
# 评论数
answer['data'][i]['comment_count']
```

结束。拿着数据去分析吧