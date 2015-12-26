常用命令及说明
======================

* 初始化站点
    nikola init
* 编译并启动测试服务器
    nikola auto
* 构建
    nikola build
* 新page
    nokola new_page
* 写post
    nokola new_post -e

    tags: draft/private

    type: text(plain text)/micro(small posts)

写作相关
===========
参考 `The Nikola Handbook <https://getnikola.com/handbook.html>`_

::

   .. TEASER_END

   .. nocomments: True/False
   .. media:: video_url
   .. code:: python
   .. listing:: file python
   .. gist:: github_gist_id
   .. thumbnail:: image_url
   .. slides:: image_slide_shows
   .. chart:: Line, StackedLine, Bar, StackedBar, HorizontalBar, XY, DateY, Pie, Radar, Dot, Funnel, Gauge, Pyramid

   .. post-list::
       start : integer
           The index of the first post to show. A negative value like -3 will show the last three posts in the post-list. Defaults to None.
       stop : integer
           The index of the last post to show. A value negative value like -1 will show every post, but not the last in the post-list. Defaults to None.
       reverse : flag
           Reverse the order of the post-list. Defaults is to not reverse the order of posts.
       sort: string
           Sort post list by one of each post's attributes, usually title or a custom priority. Defaults to None (chronological sorting).
       tags : string [, string...]
           Filter posts to show only posts having at least one of the tags. Defaults to None.
       categories : string [, string...]
           Filter posts to show only posts having one of the categories. Defaults to None.
       slugs : string [, string...]
           Filter posts to show only posts having at least one of the slugs. Defaults to None.
       all : flag
           Shows all posts and pages in the post list. Defaults to show only posts with set use_in_feeds.
       lang : string
           The language of post titles and links. Defaults to default language.
       template : string
           The name of an alternative template to render the post-list. Defaults to post_list_directive.tmpl
       id : string
           A manual id for the post list. Defaults to a random name composed by 'post_list_' + uuid.uuid4().hex.

   link to other posts :doc:`creating-a-theme`
