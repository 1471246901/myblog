## 代理层 详情服务

### 下游服务

| 服务           | 解释           | url  |
| -------------- | -------------- | :--: |
| detail.service | 详情服务       |      |
| stream.service | 第三方视频服务 |      |
| cloud.service  | 自有视频服务   |      |
| rec.service    | 推荐服务       |      |
| geo.service    | 位置服务       |      |
| data.report    | 数据上报       |      |
| book.detail    | 点阅书籍服务   |      |

---



### 接口

| 接口url                          | 接口简介                                                     | 涉及到的服务 | 对应的接口                    |
| -------------------------------- | ------------------------------------------------------------ | ------------ | ----------------------------- |
| /api/v2/detail/website           | 请求各网站对应视频的元信息                                   | detail       | detail /allInfoV2             |
| /kuaikan/apisimpledetail_json.so | 获取视频元信息                                               | detail       | ？                            |
| /api/v2/rec/book                 | 相关图书推荐 老版本客户端在用 新版本已经整合进/api/v3/related/rec接口 |              |                               |
| /api/v2/rec/book2                | 播放器图书推荐接口新版本已经整合进/api/v3/related/rec接口    |              |                               |
| /api/v2/related_album            | 相关视频推荐 老版本客户端在用 新版本已经整合进/api/v3/related/rec接口中 |              |                               |
| /api/v2/rec/related              | 相关推荐                                                     |              |                               |
| /api/v2/stream/cloud             | 自有存储视频请求云盘服务获取播放地址,具体逻辑见/kuaikan/apijiexi_json.so | cloud        | cloud/openapi/openapi/playurl |
| /api/v2/stream/get               | 请求流服务播放地址                                           | stream       | stream/stream_message_v2      |
| /api/v3/stream/get               | 请求流服务播放地址                                           | stream       | stream/stream_message_v3      |
| /api/v3/related/rec              | 相关推荐数据  图书（小说）、短视频（看过这部分的人都在看）、长视频（相关正片） |              |                               |

##### /api/v2/detail/website

请求detail.server 服务

剔除siteList中的  一些数据并保留   tag category isEnd 属性

过滤版本 和宝宝巴士的Site

构建。pageWebsite  会根据用户信息，专辑排序

对特殊源使用 云盘播放逻辑

##### /api/v2/rec/book

相关图书推荐 老版本客户端在用 新版本已经整合进/api/v3/related/rec接口

##### /api/v2/rec/book2

播放器图书推荐接口

##### /api/v2/rec/related

相关推荐,不含有人工配置的视频数据

get

##### /api/v2/related_album

相关视频推荐 老版本客户端在用 新版本已经整合进/api/v3/related/rec接口中

##### /api/v2/stream/cloud

自有存储视频请求云盘服务获取播放地址

具体逻辑见/kuaikan/apijiexi_json.so

##### /kuaikan/apijiexi_json.so

自有存储视频请求云盘服务获取播放地址  ，已经整合进 /api/v2/stream/cloud 接口中  主要逻辑还在此

###### 逻辑

cloudHost服务/openapi/openapi/playurl 接口获得playurl

处理下载链接和播放链接. 加密、构建response

##### /api/v2/stream/get

请求流服务播放地址

###### 参数

device   设备信息

url  url播放地址

formats  格式

eid 

###### 逻辑

请求 streamHost流服务/stream_message_v2  获取并返回

##### /api/v3/related/rec

相关推荐数据  图书（小说）、短视频（看过这部分的人都在看）、长视频（相关正片）含有人工推荐的数据

###### 图书推荐relatedBooks

relatedBookService.getRelatedBooks   获取有关的图书 （从cms和推荐系统中来）

###### 视频推荐manualRecService.related  包含短视频和长视频

##### /api/v3/stream/get

请求流服务播放地址

###### 参数

device	设备参数	

url	待解析播放地址
formats	
eid	
extra_data	
os_type	操作系统类型
js_version	流服务解析js版本

请求 stream服务/stream_message_v3  获取播放地址

---



### Repository

#### *老cms*

##### app_check_socket

表 app_check_socket      

dao AppCheckSocketDao

缓存 appCheckSocketCache  key = "#platform+'#'+#playType+'#'+#version"

使用 `/api/v2/detail/website` `/api/v2/rec/related`

dao CloudCodeDao



CloudCodeDao

使用 `/api/v2/stream/cloud`

---



#### *cms*

##### applet_share

小程序分享微信小程序id

表  applet_share

dao  AppletShareDao   

缓存 全量缓存定时刷新。 （0 0/5 * * * *） 每五分

##### book_rec_bind

分发推荐-小说池的人工绑定的小说

表   book_rec_bind

dao  BookBindRecDao

缓存 全量缓存定时刷新。 （15 0/5 * * * *） 每第五分15秒刷新

并缓存 根据aid分类并排序 的小说池

使用    `/api/v3/related/rec` ~~`book` `book2`~~

##### book_rec_pool

推荐-小说池

表   book_rec_pool

dao  BookPoolDao

缓存 （book_id, config_id, filter）缓存 定时刷新。 （20 */15 * * * *）

并缓存 根据(BookSlot config_id)分类并排序 的小说池

使用    `/api/v3/related/rec` `/api/v2/rec/related` ~~`book` `book2`~~有使用

##### dict_city

城市-字典

表   dict_city

dao   CityDao

缓存 （book_id, config_id, filter）缓存 定时刷新。 （0 0/5 * * * *）

并缓存 根据(name，code)分类

使用    `/api/v2/detail/website` 

##### nets_play_url

nets渠道 播放地址

表   nets_play_url

dao  CloudThirdPlayDao

缓存 缓存状态正常（state =1 ）的全量数据（aid 视频id，episodes 集，playtype 0云盘1截流2sdk ，real_url 截流地址，stream_src 源站点 ）   （0 0/5 * * * *）

并根据aid 视频id 和 episodes 集 使用 caffeine 缓存

使用  `/api/v2/detail/website ` 

##### display_strategy

播控可见级别。（播控策略）

表   display_strategy

dao  DisplayStrategyDao

缓存 全量缓存（id 级别id，data 播控可见数据）（0 0/5 * * * *）

并根据id分类

使用  `短视频 /api/v3/related/rec ` 

##### long_video

长视频

表   long_video

dao   LongVideoDao

缓存 缓存开启的视频 （state =1）缓存（id 主键，aid 视频id，name 视频名称，vt 视频类型 ）（0 0/5 * * * *）

并根据aid 进行分类

使用 `/api/v3/related/rec` 

##### book_rec_config

短视频推荐配置

表  book_rec_config 

dao    RecBookConfigDao

缓存  缓存 开启的小说推荐配置（id 逐渐，bind 是否有绑定关系【1有2没有对应book_rec_bind关联字段config_id】，num 推荐个数，strategy 策略，strategy_data 策略数据，target 1搜索2播放相关图书3播放相关推荐图书）  （15 0/5 * * * *）

根据渠道 （BookSlot   target ）进行分类

使用 `/api/v2/rec/related` `/api/v3/related/rec` 

##### short_video

短视频

表   short_video

dao   ShortVideoDao

缓存     使用 caffeine 缓存key `svl_longVideoId` 的数据

使用 `/api/v3/related/rec`

##### source_pack

Site欺骗

表   source_pack

dao   SiteCamouflageDao

缓存     缓存开启的（state =1） 的全部数据。 （0 0/5 * * * *）

使用 `/api/v2/detail/website`

##### site_replace

解决VideoSite争论

表   site_replace

dao   SiteReplaceDao

##### video_top

排名

表 video_top

dao TopAlbumDao

缓存   缓存（album_id 视频id）（7 30 * * * *）

使用   `/api/v3/related/rec` `/api/v2/rec/related`

##### video_tag

视频标签

表   video_tag

dao   VideoLabelDao

缓存  缓存 （is_show 展示1，gruop 类别层面，num>10该tag下的视频条目数）（0 0/5 * * * *）

使用  无（用于通过视频aid 获取视频tag）





---

### <span style="color:red">视频详情接口</span> 

#### /api/v2/detail/website   站点源视频详情  各个网站对该视频的源信息

##### 参数

`device	设备参数`	`aid	视频aid`	`site	网站`	`pn	页码`	`ps	页大小`	

##### 响应数据



##### 逻辑

通过`compositeDetailService`获取detailHost/allinfo_v2的专辑详情

```
tag", json.get("tag"));
category", json.get("category"));
isEnd", json.get("isEnd"));
siteList", siteListArray);
```

通过`siteFilterProduce` 对sitelist 进行过滤，目前的过滤器有

- `BabybusSiteFilter`  过滤版本小于 3.5.6 的android版本
- `FunshionUnionSiteFilter` 过滤小雨3.9.3的android版本
- `PPTVSiteFilter` 过滤android版本或 tag含有 pptvios可展示 

···

通过`pageWebsite` 方法获取 并过滤以下规则的site

- ​	`appCheckSocketService` 过滤 渠道、版本、平台

- ​	`siteOfflineService`  普通用户 则 过滤 OfflineSites 信息

- ​	`appReviewSite` 过滤appReviewSite

  对site（源于sitelist）进行转化获取 相应的json字段

  ```
  videoId 【】
  site Site.find(siteData.getString("site"));
  sitelogo 经过siteCamouflageService site伪装的过滤
  sitename 经过siteCamouflageService site伪装的过滤
  aid 【】
  episodeStatus idEnd
  episodes 【】
  nowEpisodes 【】
  src
  subsrc
  external_id
  orderNum
  playStreams
  copyright
  orderlist 专辑类别如果是综艺，则特殊处理
  taglist 
  recReport 短视频为1 其他为0
  videoList resolveVideoList（）处理
  ```

  ​	videoList  = resolveVideoList() 中的处理

  ```
  external_id 转化为外部id fieldUtils.resolveExternalId
  porder 对专辑类别为综艺进行特殊处理 orderArray[zyIndex]，对非专辑 fieldUtils.fixPorder（porder｜aorder）
  name 【】
  url  siteCamouflageService site伪装过滤，并应用 handleStreamAndDisplayUrl规则
  displayUrl  siteCamouflageService site伪装过滤，并应用 handleStreamAndDisplayUrl规则
  mid
  pls
  globalVid siteOfflineService.fetchRedirectSites不匹配则通过globalVid字段为空触发 H5页面播放
  vid
  isdownload 1
  cloudId  site 为 Nets时handleCloudPlay方法中的规则
  playType  site 为 Nets时handleCloudPlay方法中的规则
  realUrl  site 为 Nets时handleCloudPlay方法中的规则
  streamSrc  site 为 Nets时handleCloudPlay方法中的规则
  isPay 
  deadlink
  contentType
  preview
  copyright
  copyAddress
  timePoint
  ```

  并对 preview 进行过滤 filterPreview 

···

对`pageWebsite` 的数据进行遍历

​	通过`singleAlbumPlayPolicyService` 获取（device，aid）对应的 专辑播放逻辑

​	对 site 为LETV 使用 configLetvPlayer 

```
configLetvPlayer
letvPlayCtrlService.findPlayCtrl(device); 获得letv的播控逻辑
根据默认的播控逻辑、letv播控逻辑、默认 对website设置 
primePlayType
failoverPlayType
player
download
letvDownload
```

​	对 site 为非LETV  configPlayer

```
primePlayType
failoverPlayType
player 如果是SDKplayer 则通过playerControlService.findPlayer(device, site) 获得播放控制否则使用默认的播放控制逻辑
script aqy播放控制逻辑需要添加播放脚本
videoList 检查是否禁用无external_id的剧集
```

​	对 site 的 `download` 字段进行修改 

​	如果 singleAlbumPlayPolicy 配置了下载规则则使用singleAlbumPlayPolicy 的下载控制，否则 使用 siteDownloadOffService .canDownload 的下载控制 

​	如果download 为1 开是 设置 `p2pDownload` p2p 下载开关为p2pDownloadService.supportP2pDownload

​	如果 此视频 site 是 重定向 的 （siteOfflineService.isRedirect） 设置打开方式 `openway`  2 （h5打开）否则为1 （native播放）

​	设置`sdkstream` （是否截流播放）字段为 allowStream 目前 不允许截流播放的site 为 【FUNSHION，SOHU，BESTV，IMGO，BABYBUS】

​	`defaultPlayer` 	 字段 ，由resolveDefaultPlayer 指定

···

siteSortService.sortSite(newSiteArray, device);  对sitelist 进行排序

albumSiteSort(albumSiteOrder, newSiteArray);  对sitelist 进行排序

···

设置字段

```
site  realSite.getSource()
subsrc  realSite.getSource
defaultPlayer  nets
speed speedPlayService.assembleSpeedPlay

```

通过ApiResponses 封装并返回

#### /kuaikan/apisimpledetail_json.so  获取视频元信息

##### 参数

`device	设备参数`	`aid	视频aid`	`download	是否下载`

##### 响应数据



##### 逻辑

compositeDetailService.albumSite获取专辑视频源信息

···

​	detail服务/allWebsite 获取aid 所对应的所有视频源信息

​	通过  cleanSiteData() 过滤返回的部分信息		

```
external_ids
videoSingleVideoImages
external_play_ids
unIds
isTv
dataType
isPc
isMobile
intro
letv_original_id
videoDeadlinkUrls
```

对返回数据中  siteList 中的每一个 site 应用 cleanSiteData() 过滤信息 并且过滤掉`external_id `字段

···

如果 DUAN_SHI_PIN.isOrigin(返回的数据.getIntValue("category"))  类别为短视频 则去掉 authors 字段否则

···

对authors （list）字段中的属性 进行过滤

仅保留 `name - name ` `avatar - avatar` `id - author_id`  属性  并对name 为空或大于 9 字符的数据进行过滤 过滤掉 author

如果avatar 为空的话 设置avatar 的值 为 `https://s.yingshidq.com.cn/avatar/default_uploader_sculpture.png`

···

fetchSimpleDetail 将除下列的其他字段过滤掉

```
aid ，src， name ,category ,vt ,categoryname,releasedata ,year,areaname,directory,directoryname,starring,starringname,isend,episodes,nowepisodes,rating,description,pls,poster,author,poster2,tag,[siteList]

```

增加字段 `shareurl` 为resolveShareUrl

短视频设置`applet_id` 来自 appletShareDao.findAppletShareId（categoryname）

对sitelist 进行操作

···

下载开启 通过`singleAlbumPlayPolicyService.getAlbumPlayPolicy` 和 `siteDownloadOffService.canDownload` 过滤不能下载的site

过滤不存在的站点site 

过滤 非白名单用户且site（siteOfflineService.fetchOfflineSites）已被下线 的site

设置字段 `openway` 为  siteOfflineService.fetchRedirectSites 重定向site服务中的规则

设置site logo 和name  为 siteCamouflageService.findCamouflageSite 如果没有伪装规则 则设置默认的logo和name

···

siteSortService.sortSite(newSiteArray, device);  对sitelist 进行排序

albumSiteSort(albumSiteOrder, newSiteArray);  对sitelist 进行排序

···

通过`siteFilterProduce` 对sitelist 进行过滤，目前的过滤器有

- `BabybusSiteFilter`  过滤版本小于 3.5.6 的android版本
- `FunshionUnionSiteFilter` 过滤小雨3.9.3的android版本
- `PPTVSiteFilter` 过滤android版本或 tag含有 pptvios可展示 

···

重新设置sitelist head 的site 中的subsrc ，sitename，sitelogo

设置status 和 errmsg 并且通过ApiResponses 包装并返回

### <span style="color:red">视频播放地址接口</span>

#### /api/v3/stream/get 	请求流服务播放地址

##### 参数

`device	设备参数`	

`url	待解析播放地址`	
`formats`	
`eid`	
`extra_data`	
`os_type	操作系统类型`
`js_version	流服务解析js版本`

##### 响应数据



##### 逻辑

构建 StreamRequest 

```
url，formats，clientIP，eid，estra，osType，uuid，version，jsversion，requestSource
```

对requestFormat 字段进行映射 `100 - ALL_FORMAT` `FORMAT_MAP.get(requestFormatid);`

请求 `stream服务/stream_message_v2 ` 并返回



#### /api/v2/stream/cloud   自有存储视频请求cloud服务获取播放地址

##### 参数

device	设备参数	

requestType	请求类型（play 播放，download 下载）	

src	视频播放源（nets）	

type	android: {0:mp4, 1:m3u8} ios: {1:m3u8, 2:mp4}	

k-uid	

k-dc	

k-old-auid	

cloudcode	

peid	

p2pType	1:玩客云 2:自有p2p	

vt	视频类型	

##### 响应数据



##### 逻辑

根据版本和平台确定p2p类型 1:玩客云 2:自有p2p	支持自有p2p 则设置为2

下载模式 且 p2p type 为 自有p2p ，类型type修改为m3u8

调用 `/kuaikan/apijiexi_json.so` 接口

构建cloudRequest

```
.cloudCode(cloudCode)
.uniqueId(oldAuid)
.city(device.getCity().getCode())
.requestType(requestType)
.type(type)
.cloudId(cloudId.getCode())
.tag(cloudId.getSrc() == 1 ? 1 : 2)
.clientIp(device.getClientIp())
.uid(uid)
.peid(peId)
.downloadType(p2p)
.nuid(device.getNuid())
.ldid(device.getLdid())
.version(device.getVersionRaw())
.platType(device.getPlatCode().getCode())
.platform(device.getPlatform())
.vt(vt)
.FCode()  = cloudId+Time/1000+rabdomInt(0-1000)+ClientIp+uid+devId
```

cloudPlayUrlService.getPlayUrl 获取cloud 播放地址

​	`cloud服务/openapi/openapi/playurl` 获取cloud 播放地址

​	newHDService.isNewHD 判断是否进行高清晰度测试

​	判断是否为isDefaultHD wifi +version +android

​	`retainLowDefinition（）` 处理播放链接为低清晰度，保留6个清晰度，并设置默认清晰度字段`default_play`

​	`cloudUtils.resolveDownloadParam`  处理下载链接 将下载链接中的playid=0 修改为playid=2

对数据进行加密

封装加密的数据 stream ：加密的数据 ，p2pType：p2p方式 ，encode：2

### <span style="color:red">视频推荐接口</span>

#### /api/v3/related/rec 相关推荐数据图书、短视频、长视频   含有人工配置的数据

对应 详情页小说推荐、看过这部片的人都在看

根据视频类型图书短视频长视频会有不同的推荐

##### 参数

`device	设备参数`

`aid	视频专辑aid`
`k-dc	设备标识`

##### 响应数据

##### 相关图书推荐

```
title  相关图书推荐
items  relatedBooks(device, aid).toJavaList())
name bookRec
style  3
```

通过`relatedBooks（）` 获取推荐的图书 大小固定 为3

​		`relatedBookService.getRelatedBooks（）` 获取详情页相关图书 `RELATED_BOOK`

···

`recBookConfigDao.recBookConfig(slot)`  获取人工配置的推荐规则

`bookPoolDao.bookIds`  获取 对应slot 且 `gender`符合的 bookid

通过人工配置推荐规则`bookMatcherMap.getBean(recBookConfig.getStrategy());` 对 bookpool 中的数据进行筛选，并且保留指定条数

通过`bookDetail` 方法获得书籍详细信息

​	具体通过bookDetailService 获取数据 `bookDetail服务/internal/v1/book/detail/multi`

​	对结果保留books 字段中 `id, name,cover,intro,authors` 字段  作为 bookinfo

​	组装 bookSlotElement 对象 

```
Id(bookInfo.getId());                                        
Authors(bookInfo.getAuthors());                              
Pic(bookInfo.getCover());                                    
Desc(bookInfo.getIntro());                                   
Title(bookInfo.getName());                                   
Original(0);                                                 
CornerText(linkType == LinkType.H5 ? "小说大全" : "小说");         
CornerColor(linkType == LinkType.H5 ? "#F0A102" : "#FF8849");
LinkType(linkType);                                          
Link(linkType.getLink(bookInfo.getId()));                    
DataType(DataType.BOOK);                                     
```

通过 recBookConfig 对图书池中的数据进行切割，并且获得人工绑定的数据`findBookManual` 

`mergeBooks` 整合图书池数据和人工绑定的数据（去重 ，并且优先展示人工数据），并且再次通过recBookConfig 对数据切割为指定大小

**视频**

manualRecService.related(device, aid) 获取aid手动配置的视频

​	如果aid 是短视频 则只获取对应的长视频 ，并且将它们排序，根据视频类型进行分组 （长短视频）

​	通过`detail服务/albumInfo` 拿到详情视频详情并合并 

​	长视频则下发短视频相关数据，短视频则不下发相关短视频数据

​	对短视频进行处理，如果手动配置的视频小于10条 则`rec服务/bigsarrs` 填充

##### 看过这部片的人都在看 短视频

如果aid 是短视频，则为空，如果是长视频，则为10条相关短视频，手动配置有限

##### 相关正片 长视频

`manualRecService.related` 获得的长视频

#### /api/v2/rec/related   相关推荐数据，不含有人工配置的数据  

对应详情页「热播推荐」板块

调用 `rec服务/bigsarrs`  根据aid 和cid 返回推荐的数据

对返回的数据处理，保留以下字段信息。返回rec 中的 ，`status  reid  bucket`

```
title year sub_category area vt category_name pic pic2 aid score episodes now_episode duration is_end siteName type bucket author
```

对视频生成 分享链接 `shareurl`，小视频下发小程序分享ID `applet_id`  

对推荐列表加入 图书推荐 `assembleBook`   （版本要求）

构建消息并通过 ApiResponses 封装 返回

```
reid  bucket   items 来自rec并添加了图书推荐    style   
```



### 首页服务

consul ，hystrix ，ribbon，caffeine

api/v2/init  应用启动时调用

上报用户的位置，平台，渠道，启动时间等数据

/api/v2/index/header    应用启动时调用

获取首页类别和二级分类

/api/v2/index/page  选中模块，刷新

获取模块页的数据 （板块，轮播图）

/api/v2/feed/more 上滑加载

推荐视频

/api/v2/feed/more  热点模块标签请求



/api/v2/theme/list    专题列表页，或刷新上滑

专题列表

/api/v2/theme/detail   专题详情页 

专题详情

/kuaikan/apiranking.so   排行榜

排行榜

/kuaikan/apihandlerinfo_json.so  数据上报

用户数据上报 每2周

/kuaikan/apiextractjsnew_json.so   更新流服务解析js脚本信息

获取 获取extract.js文件

/api/iptable/ipid  ip定位地理位置

用于定位城市和ip



### 搜索服务

/api/v2/search  搜索

获取搜索的内容

/kuaikan/apisuggest_json.so  联想搜索

输入搜索词时的联想

/api/v3/hotsearch  热搜  搜索页

搜索页热搜信息

/api/v2/channel/list    分类页

分类页获取视频类型 （电影、电视剧。。。。）

/kuaikan/apisubcategory_json.so   分类页点击视频类型

根据视频类型获得子分类 和 关键词猜你想搜

/kuaikan/apilistnew_json.so   分类页点击子分类

根据子分类获取视频

/api/v1/author/albums   短视频作者页

获得作者相关作品

### other 服务

/kuaikan/apiaddcollection_json.so   长视频播放页登陆后 添加收藏

上报收藏

/kuaikan/apiremovecollection_json.so   长视频播放页登陆后 取消收藏

取消收藏

/kuaikan/apicheckcollection_json.so   收藏列表页

收藏过的视频

/kuaikan/v2/apiplaylist_json.so  播放历史页

播放历史

/api/v2/feedback/category  反馈分类

获取用户反馈的两级分类

/api/v2/feedback/report   提交反馈

提交反馈信息

/api/v1/video/feedback   短视频负反馈

首页短视频不感兴趣反馈

/api/v2/rec/report   视频推荐播放后上报 

上报推荐视频播放

/api/v2/hot/favorite   收藏页热门收藏

热门收藏数据

/kuaikan/apidownloadrelated_json.so   离线缓存页推荐数据

离线缓存页推荐数据

/kuaikan/apihotdownload_json.so   离线缓存页热门下载数据

离线缓存热门下载数据

/api/v2/upgrade  升级

升级

/api/messages/list  消息

消息列表

/h5/simpledetail   url分享，用户点击链接之后

html页面

/h5/related   分享出去的html页面上的相关推荐

相关推荐数据

/kuaikan/apicheckalbumlist_json.so    查看已经看过的剧集是否有更新

更新信息

/kuaikan/apiuserprofilesync_json.so   获取用户信息

获取用户年龄性别信息
