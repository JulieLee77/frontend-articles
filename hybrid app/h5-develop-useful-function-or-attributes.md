#h5开发实用功能／属性
一些常用的功能，可根据需求应用。
##meta属性
- apple-mobile-web-app-capable

	```
<meta name="apple-mobile-web-app-capable" content="yes">
	```	
	yes删除apple默认的工具栏和菜单
- apple-mobile-web-app-status-bar-style

	```
	<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
	```
	default/black/black-translucent 设置apple状态栏样式
	另会配合apple-touch-fullscreen
	
	```
	<meta name="apple-touch-fullscreen" content="yes">
	```
- formata-detection
	
	```
	<meta name="format-detection" content="telephone=no">
	```
	telephone=no/email=no,adress=no 取消apple的默认格式识别
	
		
##-webkit-min-device-pixel-ratio
```
@media and screen(-webkit-device-pixel-ratio:2){
	...
}
```	
`-webkit-max(min)-device-pixel-ratio` 设备最大／最小像素密度

`deviePixelRatio`指的是 `window.devicePixelRatio`，被所有主流浏览器支持。

`devicePixelRatio=物理像素／dip`

dip或dp（**device independent pixel**）与屏幕密度有关。可以用来辅助区分视网膜设备和非视网膜设备。
##autocompelete="off"
去掉输入框的历史记录
##-webkit-tap-highlight-color

```
a,img,button,input,textarea{
	-webkit-tap-highlight-color:rgba(0,0,0,0);
}
```
取消移动端tap后的高亮显示
##去掉input，button点击时蓝色边框

```
input[type=button],button{
	outline:none;
}
```
(input一般只用得上type=button)

##-webkit-appearance:none

```
textarea,input{
	-webkit-appearance:none;
	appearance:none;
}
```
去掉`textrea`, `input`的默认样式（如ios的圆角及内阴影）。对于 `radio`，`checkbox`可解决移动端不能设置大小（扩展选取范围）。
##user-select:none

```
body{
	-webkit-user-select:none;
	user-select:none;
}
```
去掉文本可选取复制。
##pointer-events:none;
决定是否可以穿越绝对定位元素，触发遮挡元素的某些行为。
##-webkit-touch-callourt:none;
当触摸链接时禁用系统默认菜单。
设置ios的webview可设置body此属性。
	