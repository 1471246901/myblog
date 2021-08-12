### proxy 主页

#### /api/v2/init

应用冷启动时调用一次

##### 参数

```
InitializeRequest
location 位置
platform 编号+渠道 AppId+Channel
tags 用户标签 
appId 客户端appId  Le123Plat001 
market 渠道      ?????
version 客户端版本号   值得是客户端版本号
terminalId 终端类型   指的是iphone android atv 这类的终端
uuid 设备标识
uid 用户标识
auid  auid ?????    ios设置2.4.6之前版本没有auid字段,这时使用uuid
nuid  nuid 新的auid ????
ldid  ldid  ????
tags 用户标签
clientip 客户端IP
city 城市编号
screen 屏幕尺寸
model 型号
idfa idfa ?????
webp 是否支持webp 1支持,0不支持
gender2 用户性别
oaid  pr加入联盟会有oaid ????
personalized 是否开启个性化推荐 1开启,0关闭
traceId  traceId 统一请求编码
age  年龄
partner 合作方 ????
pages  pages ??????


其他参数
novelToastId [ntid] 小说xID
sdid 
ts  时间戳
lasttime [lastShowTime] 
k-market [kMarket]
```

##### 行为 

**`解析请求数据[buildRequestParams]`**  设置请求参数,黑白名单,用户级别

1 对黑名单用户设置城市为背景,对oppo预装包用户设置城市为北京

2 对没有auid 字段的IOS 用户使用uuid(设备标识) 代替

**`限速[downloadRate]`** appSettingService.getDownloadSpeedLimitNow  根据 ProductId (产品ID) ,TerminalId(OSCode) 获取客户端限速 (限速具有缓存, )  没有限速(数据)返回-1

**`启动图片[splashPic]`** appSettingService.getSplashPic  根据 ProductId (产品ID) ,TerminalId(OSCode) 获取启动图片 启动图片为url格式

**`获取启动图片显示时间 [splashInterval] `** appSettingService.getSplashInterval 获取图片显示时间

**`app审核状态[appCheck]`** appSettingService.getAppCheck 通过 platform(编号+渠道),TerminalId(终端类型),Version(版本) 获取app审核状态 审核状态可以决定是否需要审核,是否需要反馈

**`获取客户端私密协议链接[ppocy]`** ,**`获取客户端法律条款链接[lawPolicy]`** , **`获取客户端隐私协议链接 [privacyPolicy]`** , **`获取友盟隐私政策链接 [umengPrivacyPolicy]`**  等需要显示的内容

**`生成 finalTags`**   =  `用户Tags`  + `suidtag` + `ts(时间戳)` + `sdid` + `?openlist`

- **`获取用户Tags [tagsArgumentsMap]`**  appSettingService.getTagsArguments 根据用户的 ProductId(产品id),terminalId(终端id), group(通过 Auid userService.getUserGroup 获得) ,Version(版本) 获取用户所属的tags内容 在增加 `usertag wuser_0_0_0` 和 `newusertag nuser_0_0_0` 两个tag
- **`生成 suidtag [suidtag]`**  genExtraTag 获取 nuid 和 ldid ("nuid_" + nuid + ",ldid_" + ldid ) 否则生成
- **`ts 时间戳`**
- **`sdid ???`**
- **`openlist`** 根据Auid 获取是否具有openlist

**`分组推送参数[arguments]`**  getArguments  根据 用户Tags 获取推送参数 

**`启动图片展示次数[splashLimit]`** 

**`优酷短视频上报参数[youkuArguments]`**   ,  **`好兔短视频上报参数[haotuArguments]`**

 **`是否弹出画像选择页[UserProfile]`** userProfileDialogService.isPopDialog(requestParams, lastShowTime) ? 1 : 0

 

##### 响应

```
userProfileDialog 是否弹出画像选择页
splash  默认启动图
splashInterval  重新展示启动图页面的时间间隔，单位为秒，-1表示不重新展示
splashScreen  黑屏再亮屏后，增加启动图展示，单位为秒，-1表示不重新展示
splashLimit 启动图展示次数
downloadRate 下载限速
check  审核状态 来自appCheck
report 反馈开关 来自appCheck
ppocy 隐私访问地址
applist  applist 淘宝,快手,gifmaker , 拼多多 ,美团 ,京东
lawPolicy  法律条款访问地址(新版)
privacyPolicy 隐私协议访问地址(新版)
umengPrivacyPolicy  友盟一键登录隐私政策
tags  用户标签  finalTags
yiDianTag 一点资讯tag  
tagArgs  分组推送参数 arguments 
youku , haotu 优酷好兔上报参数
novelToast 点阅导量位数据 非新用户
	id , type 类型，0图文 1视频 , jump_type 跳转类型 ,landing_type h5页 ,material_url 素材链接 ,video_url 视频url ,text 文案, recid 配置ID
```

#### /api/v2/index/header

应用冷启动获取频道数据

##### 参数

```
appId 客户端appId  Le123Plat001 
market 渠道      ?????
version 客户端版本号   值得是客户端版本号
terminalId 终端类型   指的是iphone android atv 这类的终端
uuid 设备标识
uid 用户标识
auid  auid ?????    ios设置2.4.6之前版本没有auid字段,这时使用uuid
nuid  nuid 新的auid ????
ldid  ldid  ????
tags 用户标签
clientip 客户端IP
city 城市编号
screen 屏幕尺寸
model 型号
idfa idfa ?????
webp 是否支持webp 1支持,0不支持
gender2 用户性别
oaid  pr加入联盟会有oaid ????
personalized 是否开启个性化推荐 1开启,0关闭
traceId  traceId 统一请求编码
age  年龄
partner 合作方 ????
pages  pages ??????

platform   应用编码+渠道 AppId+Channel
sqtype   查询词类型
```

##### 行为 

**`app审核状态[appCheck]`** appSettingService.getAppCheck 通过 platform(编号+渠道),TerminalId(终端类型),Version(版本) 获取app审核状态 审核状态可以决定是否需要审核,是否需要反馈

**`获取导航栏数据[navbarjson]`** homePageService.getNavigationBarsForJson 通过appid ,version,isNewFeed,  获取json 格式的navbar 数据  

- 获取缓存中的navbarlist 
- 进行过滤
  - 匹配产品版本及屏蔽渠道   RuleUtil.matchProductVersionAndExChannel
    - 生效产品过滤  appIdVersionRuleMap 用于过滤不在此产品线上生效的导航栏
    - 渠道屏蔽过滤  appIdVersionRuleMap.get(appName).getChannelRule  
    - 版本version过滤   过滤生效产品中指定version 的情况
  - 匹配是否是否审核数据    获取审核的数据或者正式的数据
  - 匹配新分发规则  RuleUtil.isMatchDisplayRule
    - exCity 过滤 所在的城市
    - 匹配用户规则 displayRule
  - oppo预装包过滤短视频 OPPOPreUtil.oppopreFiltedNavbar 
- 如果没有数据,设置默认数据
- android  NewFeed 新feed流 做数据调整 将激素看片放到第一位 
  - 是新feed  将 jskp(极速看片频道放到第一位,及默认显示)
  - 不是新 feed  根据点阅usertags  和 params 中的tags 做数据调整
    - 点阅tags   将极速看片放到第一位
    - params 中 js_new  tag  将 极速看片放到第一位
- 获取默认搜索词 通过**推荐服务**获取搜索词 ,并按照类型进行分类
- 构建返回JSON 
  - name(名称) ,corner_icon(角标图标),icon(导航栏图标),

**`推广app[mfishTopic]`**    当应用为 Le123Plat202 时 , 发送kafka 消息到 mfishTopic 频道 推广app

##### 响应 

```
id  NavigationBarID
name 导航栏名称
isPublished 是否发布
loadPage 是否默认页
videoType 视频分类
pageId 统计字段
pullType 刷新规则 1 下拉刷新 , 2 下拉加载
orderNum  排序编号
corner_icon 角标图片
icon 导航栏图片
is_default 是否默认页
page 统计字段
vt 视频类型
search_query 搜索词  直播导航栏不需要搜索词
image 替换文字的图片
page_type  导航类型
channel_id 小说导航id
capsule 胶囊 直播 (type:LIVE,value:capsulename,code:id)
```

#### /api/v2/index/page

获取导航栏下的内容

##### 参数

```
_f	043115214405
nt	wifi
lc  设备标识
ts	时间戳 1628134067694
sig	40d3887304dab9d12bada587dd5fc09a
age2	-1
city	城市 CN_15_187_2
code	346e5b9d1bd97036
ipid	chn-bj-bj
ldid	5e83db7553837c8fbb404002fbef65ef
page	当前频道  page_index_variety
tags	用户tags djbmd,zy_reader,sug,pagehz,cmsadtestdq,hotsearch,js_new,feed_b,speed,autoplay,p2p_d,aB_js,AB_test4,AB_test5,AB_proxy,AB_test2,265DXVA,hsplay,toutiao,order,wuser_0_0_0,nuser_0_0_0,nuid_5ec515b5f13ac12d,ldid_5e83db7553837c8fbb404002fbef65ef
uuid	c3740461-6d70-31b4-be6e-71128c7a8ca2
webp	1
pages	各个频道page_index, page_index_episode, page_index_film,page_index_novel, page_index_jisu_YX,page_index_variety,page_index_cartoon,page_index_children,page_index_haiwai,page_index_jisu_CW,page_index_jisu_GX,page_index_jisu_SS,page_index_Documentary
times	访问次数(会根据访问次数动态更新3屏) 1
version	4.0.8
guestid	5ec51589633bb829350040022dd3a42c
apiversion	2
capsule	    胶囊 指用户选择的胶囊
pageindex	第几页  1
pagesize    分页大小 4
platform	应用编码+渠道 Le123Plat002360
plattype	aphone
recid	模块id
```

##### 行为

**`构建请求参数[requestParams]`** buildRequestParams 

**`app审核状态[appCheck]`** appSettingService.getAppCheck 通过 platform(编号+渠道),TerminalId(终端类型),Version(版本) 获取app审核状态 审核状态可以决定是否需要审核,是否需要反馈

**`获取页面数据[data]`** homePageService.getPageDataForJson 获取json形式的页面数据

- 获取选中的导航栏  根据导航栏id(page)  
- 根据导航栏获取导航栏所属的所有模块
  - 过滤 oppo预装包短视频模块
  - 除推荐页外其他气泡不允许插入协同过滤数据
  - isMatchRules  版本和渠道 , 地域 , 时间,tag ,gender, 用户 过滤
  - matchAlgorithm 配置算法过滤
  - matchIsNewUser  新用户过滤
  - abTest 过滤 userTags().contains( moduleTestTagName )
  - matchCapsule 胶囊过滤 过滤掉需要胶囊的模块 matchCapsule
  - 焦点图放在首页
- 所有模块获取(**推荐**)人工配置的数据,获取已经展示过的(专辑)aid
- 所有模块获取(**推荐**)推荐的数据,经由人工配置的数据去重
- 根据 model 样式对数据进行处理
- 短视频流插入广告
- 构建json消息并返回

检查获取到的数据,如果没有取到数据返回默认数据

**响应**

```json
"rec":[模块list],
"模块" :{
    "recreport":0,
    "is_pull":"0",
    "data":[专辑list],
    "book_module_id":"",
    "bucket":"",
    "refresh_name":"刷新名称",
    "unlimit":0,
    "ad_control":"{广告控制}",
    "style":"2",
    "reid":"",
    "reccode":"",
    "recname":"重磅推荐 王牌综艺",
    "recid":"CHANNEL_SHOW_NEW",
    "continue_play":"{续播}"
},
"total":3,
"icon":[分类list],
分类:{
    "jumpurl":"跳转链接",
    "name":"跳转名称",
    "icon":"图标",
    "sort":-1
}
"focus":[焦点图list],
焦点图:{
    "site":"站点",
    "display":"类型 ,album",
    "subname":"",
    "name":"名称",
    "jumplink":"跳转链接",
    "pic":"图片链接",
    "aid":"专辑id",
    "vt":"视频类型"
},
"has_more":1
```

#### /api/v2/index/module

##### he 

#### /api/v2/index/related

##### 参数

```json
[{"key":"personalized","value":"1","equals":true,"description":"个性化","enabled":true},{"key":"gender2","value":"0","equals":true,"description":"性别","enabled":true},{"key":"_f","value":"465771445500","equals":true,"description":"","enabled":true},{"key":"nt","value":"wifi","equals":true,"description":"","enabled":true},{"key":"ts","value":"1628146934841","equals":true,"description":"","enabled":true},{"key":"sig","value":"b21b32364cb7efac284e9348a8c733b8","equals":true,"description":"","enabled":true}]
```



