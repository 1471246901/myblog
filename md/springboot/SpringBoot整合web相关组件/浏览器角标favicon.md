# 浏览器角标favicon

favicon在线制作网站 https://tool.lu/favicon/

在以上网站制作一个16*16 的.icon文件,然后放在服务器根目录下面

一般浏览器会自动默认在跟目录下面查找

将名为`favicon.ico` 放入 `resource/static`

或者在html的head表头中加

```html
<link rel="shortcut icon" href="favicon.ico" type="image/x-icon" />
```

如果icon是gif动态图需要修改type属性

```html
<link rel="icon" href="gif_favicon.gif" type="image/gif" >
```

