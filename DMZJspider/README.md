 

# DMZJspider   

一个正经漫画的下载爬虫，漫画源自<漫画之家>.
> **声明**：
>* 本爬虫纯属学习讨论，如有侵权行为，请联系删除。


## 环境
> **开发环境**
>* python 3 
>* windows 7 - 10 
>* PyCharm

> **安装依赖库**
>* requests
>* lxml
>* execjs
>* bs4


## 实现过程

###  1. 获取全站检索的api
进入漫画之家的检索页面(https://manhua.dmzj.com/tags/category_search/0-0-0-all-0-0-0-1.shtml#category_nav_anchor)
打开浏览器F12可以发现漫画之家并不是通过传统的XHR引入漫画数据，也不是写死在检索页的Source中，而是通过使用js脚本中动态为检索页加js
标签来进行数据请求。

> 1. 打开开发者工具查看网络请求，发现资源请求下有较长的疑似api链接出现，点击发现返回结果的json数据，确定为要找的api:
![api][7]
> 2. 分析该请求结构可以得到检索api的结构:
```python
#野生api
api = 'https://sacg.dmzj.com/mh/index.php?c=category&m=doSearch&status={status}&reader_group={category}&zone={region}&initial={letter}&type={theme}{order}&p={page}&callback=NNN'
```
> 3. 这个api请求从哪里来的?从js脚本中找到关键脚本csearch.js，点击发现其内容中出现我们要找的动态标签添加，此时的便签src正是数据检索api:
![api0][8]![api1][9]

获取到了全站检索数据的api，此时我们就可以轻易的编写python脚本进行数据请求，那么api的各个参数对应的值是怎样的呢?
### 2. 获取api对应参数值
针对漫画的检索分类，在[检索页](https://manhua.dmzj.com/tags/category_search/0-0-0-all-0-0-0-1.shtml#category_nav_anchor)中设有具体的分类，根据分类不同，检索生成的查询首页链接也会不同。
![分类][10]
根据这个分类检索方式，把各个对应的检索项目进行编号，方便我们组成检索命令:
```python
def show_tips():
    print(u"""
    \r\n **********************************************
    \r\n  动漫之家(manhua.dmzj.com) 漫画下载爬虫，搜索下载功能使用介绍:
    \r\n  * 状态 : 全部[0]  连载中[1]  已完结[2]
    \r\n  * 分类 : 全部[0]  少年漫画[1]  少女漫画[2]  青年漫画[3] 女青漫画[4] 
    \r\n  * 地域 : 全部[0]  日本[1]  欧美[2]  韩国[3]  港台[4]  内地[5]  其他[6] 
    \r\n  * 首字母 : A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 0-9
    \r\n          (其中字母开头的直接输入大写字母即可，0-9 开头的请输入9,输入0表示全部)
    \r\n  * 题材 : 全部[0]  欢乐向[1]  格斗[2]  科幻[3]  爱情[4]  侦探[5]  竞技[6]  
    \r\n          魔法[7]  神鬼[8]  校园[9]   恐怖[1-0]  四格[1-1]  生活[1-2]
    \r\n          百合[1-3]  伪娘[1-4]   悬疑[1-5]   耽美[1-6]  热血[1-7]  
    \r\n          后宫[1-8]  历史[1-9]  战争[2-0]  萌系[2-1]  宅系[2-2]  
    \r\n          治愈[2-3]  励志[2-4]  武侠[2-5]  机战[2-6]  音乐舞蹈[2-7]  
    \r\n          美食[2-8]  职场[2-9]  西方魔幻[3-0]  高清单行[3-1]  性转换[3-2]  
    \r\n          东方[3-3]  扶她[3-4]  魔幻[3-5]  奇幻[3-6]  节操[3-7]  轻小说[3-8]
    \r\n          颜艺[3-9]  搞笑[4-0]  仙侠[4-1]  舰娘[4-2]  冒险[4-3]  其他[4-4]    
    \r\n  * 排序:  默认[0]  按照更新排序[1]   按照点击排序[2] 
    \r\n  * 示例:
    \r\n > 输入:001000 则表示搜索下载全部 '日本' 标签下按照 '默认' 排序方式排序的漫画
    \r\n > 输入:101Y1-82 则表示搜索下载全部'连载中' '日本'的,名称拼音'Y'开头,按照 '点击' 排序方式排序的漫画
    \r\n  * 提示 : 搜索功能的命令字符串必须包含上面每一个项目，或者使用下面的快捷命令
    \r\n  * 快捷命令:
    \r\n   > 1*       (表示按照默认配置的选项进行搜索下载)
    \r\n   > all      (下载所有类型的所有漫画)
    \r\n   > all-2    (按照点击排行下载所有漫画)
    \r\n   > all-3    (按照更新时间下载所有漫画)
    \r\n **********************************************
    """)
```
右键审查元素可以发现各个分类对应的api参数值:
![00][11]
这样，我们便可以找到全部的api参数值（用脚本写也可以）。如下:
```python
#检索分类对应的api参数值
CMD = {
    'status':{'0':'0','1':'2309','2':'2310'},
    'category':{'0':'0','1':'3262','2':'3263','3':'3264','4':'13626'},
    'region':{'0':'0','1':'2304','2':'2306','3':'2305','4':'2307','5':'2308','6':'8435'},
    'letter':{'0':'all','9':'9','A':'A','B':'B','C':'C','D':'D','E':'E','F':'F','G':'G',
              'H':'H','I':'I','J':'J','K':'K','L':'L','M':'M','N':'N','O':'O','P':'P',
              'Q':'Q','R':'R','S':'S','T':'T','U':'U','V':'V','W':'W','X':'X','Y':'Y','Z':'Z'},
    'theme':{'0':'0','1':'5','2':'6','3':'7','4':'8','5':'9','6':'10','7':'11','8':'12','9':'13',
             '1-0':'14','1-1':'17','1-2':'3242','1-3':'3243','1-4':'3244','1-5':'3245','1-6':'3246',
             '1-7':'3248','1-8':'3249','1-9':'3250','2-0':'3251','2-1':'3252','2-2':'3253','2-3':'3254','2-4':'3255',
             '2-5':'3324','2-6':'3325','2-7':'3326','2-8':'3327','2-9':'3328','3-0':'3365','3-1':'4459','3-2':'4518',
             '3-3':'5077','3-4':'5345','3-5':'5806','3-6':'5848','3-7':'6219','3-8':'6316','3-9':'6437',
             '4-0': '7568', '4-1': '7900', '4-2': '13627', '4-3': '4', '4-4': '16',},
    'sort':{'0':['0','0'],'1':['1','0'],'2':['0','1']},
}

```
### 3. 构建检索api
这样，有了api，有了编号对应的api参数值，我们便可以构建我们的检索命令字符串，每一个命令字符串便是一个查询api链接。比如命令:001000
则表示 状态-全部，分类-全部，地域-日本，首字母-全部，题材-全部，排序-默认 这些选择项构成的检索命令，其api链接为:
```javascript
url = 'https://sacg.dmzj.com/mh/index.php?c=category&m=doSearch&status=0&reader_group=0&zone=2304&initial=all&type=0&p=1&callback=NNN'
```
其中的排序sort字段需要进行单独判断，更新时间排序对应t,点击数排序对应h，默认则不加进api

### 4. 获取漫画名称检索api
按照全站检索api的获取步骤，我们可以点击进入单本漫画检索的首页，输入关键词“一拳”作为示例(https://manhua.dmzj.com/tags/search.shtml?s=%E4%B8%80%E6%8B%B3)
可以看到检索返回的json数据完成的传递回来：
![one][12]
可以获得检索单本漫画的api为:
```python
url = 'https://sacg.dmzj.com/comicsum/search.php?s={key}'
```
返回的json结果为一个相似漫画列表，以搜索关键词“阴阳师”为例，列表的第一个结果如下:
```json
[
	{
        "id":"423",
        "name":"少年阴阳师",
        "alias_name":"",
        "real_name":"",
        "publish":null,
        "type":null,
        "zone":"日本",
        "zone_tag_id":"2304",
        "language":"中文",
        "status":"[完</span>]",
        "status_tag_id":"2310",
        "last_update_chapter_name":"第05话",
        "last_update_chapter_id":"1677",
        "last_updatetime":"1178197740",
        "check":"0",
        "chapters_tbl":null,
        "description":"安倍昌浩——十三岁的半吊子阴阳师，到13岁才行元服之礼。在晴明的子孙中，是最浓厚继承了天狐之血的，作为晴明的唯一的继承者，其实力得到了众神的认可，但其本人却茫然不知。个性好强，讨厌的话语是“那个晴明的孙子！” 为了成为“最伟大的阴阳师”，昌浩及其同伴小怪一同展开了修行……
　　某日，在大内发生了火灾的同时，在左大臣的东三条殿也遭到异形的袭击。迄今为止从未察觉到的妖气，不断释放出危险气息逼近的正是来自遥远的西方大陆的妖怪，他们为了给负伤的主人——躬奇治疗，瞄上了拥有极其强大灵力的左大臣之女——彰子姬……
　　而后打败了穷奇，救下了彰子的昌浩，在忍受着误以为昌浩不认真工作的先辈阴阳生——藤原敏次所带来的不快的同时，继续工作。不久，平时颇为照顾昌浩的藤原行成突然遭到恶灵袭击，身染重病，生命垂危。此后，京都内发生了一连串怨灵作乱事件。彷徨四窜的灵，突然兴起的百鬼夜行。这一切事件的发生，都是由不断袭击晴明的女术师——风音所操纵……
　　用自己的生命换回红莲的昌浩，靠其去世的祖母——若菜的祈愿。奇迹般地得以重返人世。但是，作为复活的代价，他的见鬼之力就此丧失。在失去阴阳师重要的灵力的昌浩前，新的敌人出现了……在代替彰子入宫的中宫章子身边，有神秘的黑影悄悄接近。此时，晴明天命之时也开始临近……",
        "hidden":"1",
        "cover":"//images.dmzj.com/webpic/3/snyys.jpg",
        "sum_chapters":"7",
        "sum_source":"194",
        "hot_search":"1",
        "hot_hits":"33604",
        "first_letter":"s",
        "keywords":"",
        "comic_py":"snyys",
        "introduction":"安倍昌浩——十三岁的半吊子阴阳师，到13岁才行元服之礼。在晴明的子孙中，是最浓厚继承了天狐之血的，作为晴明的唯一的继承者，其实力得到了众神的认可，但其本人却茫然不知。个性好强，讨厌的话语是“那个晴明的孙子！” 为了成为“最伟大的阴阳师”，昌浩及其同伴小怪一同展开了修行……
　　某日，在大内发生了火灾的同时，在左大臣的东三条殿也遭到异形的袭击。迄今为止从未察觉到的妖气，不断释放出危险气息逼近的正是来自遥远的西方大陆的妖怪，他们为了给负伤的主人——躬奇治疗，瞄上了拥有极其强大灵力的左大臣之女——彰子姬……
　　而后打败了穷奇，救下了彰子的昌浩，在忍受着误以为昌浩不认真工作的先辈阴阳生——藤原敏次所带来的不快的同时，继续工作。不久，平时颇为照顾昌浩的藤原行成突然遭到恶灵袭击，身染重病，生命垂危。此后，京都内发生了一连串怨灵作乱事件。彷徨四窜的灵，突然兴起的百鬼夜行。这一切事件的发生，都是由不断袭击晴明的女术师——风音所操纵……
　　用自己的生命换回红莲的昌浩，靠其去世的祖母——若菜的祈愿。奇迹般地得以重返人世。但是，作为复活的代价，他的见鬼之力就此丧失。在失去阴阳师重要的灵力的昌浩前，新的敌人出现了……在代替彰子入宫的中宫章子身边，有神秘的黑影悄悄接近。此时，晴明天命之时也开始临近……",
        "addtime":"1174809780",
        "authors":"濑田日名子",
        "types":"冒险/神鬼/轻小说",
        "series":"",
        "need_update":"1",
        "update_notice":"",
        "readergroup":"少年漫画",
        "readergroup_tag_id":"3262",
        "has_comment_id":"1",
        "comment_key":"1c535365e94845756c0706801c9ef110",
        "day_click_count":"11",
        "week_click_count":"24",
        "month_click_count":"100",
        "page_show_flag":"0",
        "token":"AFFFVt",
        "source":"转载",
        "grade":"0",
        "copyright":"0",
        "direction":"1",
        "token32":"1c535365e94845756c0706801c9ef110",
        "url":"",
        "mobile":"1",
        "w_link":"2",
        "app_day_click_count":"1",
        "app_week_click_count":"1",
        "app_month_click_count":"1",
        "app_click_count":"65",
        "islong":"2",
        "alading":"2",
        "uid":null,
        "week_add_num":"1",
        "month_add_num":"1",
        "total_add_num":"0",
        "sogou":"2",
        "baidu_assistant":"0",
        "is_checked":"0",
        "quality":"1",
        "is_show_animation_list":"0",
        "zone_link":"",
        "is_dmzj":"0",
        "device_show":"7",
        "myfcomic_comic_id":"0",
        "comic_name":"少年阴阳师",
        "comic_author":"濑田日名子",
        "comic_cover":"//images.dmzj.com/webpic/3/snyys.jpg",
        "comic_url_raw":"//manhua.dmzj.com/snyys",
        "comic_url":"//manhua.dmzj.com/snyys",
        "chapter_url_raw":"//manhua.dmzj.com/snyys",
        "chapter_url":"//manhua.dmzj.com/snyys"
    },
    ...
 ]
```
获取到了api，又能通过api获取到了数据，那么爬虫就已经成功了一大半。


## 使用
>* 安装Python3，在Windows 7-10的环境下
>* 安装好所有依赖第三方库
>* 点击run.bat 或进入DOS，切换进项目目录下
```python
python main.py
```
>* 按照说明使用

## ToDo
* 增加采集源，后续将会把其他漫画网站加进检索中
* 增加采集记录
* 引入数据库(有些漫友并不喜欢太复杂，只想文件夹下有漫画。。。)
* ...

## 效果
> 单本搜索下载模式:
![model1][2]
> 多本全站搜索下载模式:
![more][3]
>下载成功:
![0][4]![1][5]![2][6]









[2]:\images\2018-10-30\02.png
[3]:\images\2018-10-30\04.png
[4]:\images\2018-10-30\00.png
[5]:\images\2018-10-30\01.png
[6]:\images\2018-10-30\03.png
[7]:\images\2018-10-30\05.png
[8]:\images\2018-10-30\06.png
[9]:\images\2018-10-30\07.png
[10]:\images\2018-10-30\08.png
[11]:\images\2018-10-30\09.png
[12]:\images\2018-10-30\10.png
