[https://juejin.im/post/596baf5f6fb9a06bb15a3df9](https://juejin.im/post/596baf5f6fb9a06bb15a3df9 "自定义 View 1-2 Paint 详解")
#自定义View绘制 第二期：Paint深入详解

	基础的Paint能实现基本的绘制需求，而深入Paint后，能够做出一些更加细致、炫酷的效果。

##Paint的API分为4类：

	*颜色 color
	*效果 filter
	*drawText()相关 
	*初始化
	
## 1： 颜色
		
	Canvas绘制的内容有三层对颜色的处理：
		基本颜色        

		   ↓	1，Canvas.drawColor/ARGB()-颜色参数
		   		2，Canvas.drawBitmap()-bitmap参数
				3，Canvas 图形和文字绘制-paint参数
		ColorFilter
		   ↓    Paint.setColorFileter(colorfilter)
		Xfermode  
				Paint.setXfermode(Xfermode)
###1.1，基本颜色 drawColor/RGB/ARGB是直接写在方法的参数里，通过参数来设置；drawBitmap()的颜色是由Bitmap对象来提供的；除此外，是图形和文字的绘制，它们的颜色就需要用Paint参数来额外设置了。
	
	Canvas的方法
		drawColor/RGB/ARGB  
			直接作为参数传入

		drawBitmap()
			与bitmap参的像素颜色相同

		图形和文字(drawCircle()/drawPath()/drawText()...)
			在paint参数中设置
		

####1.1.1Paint设置颜色的方法有两种：一种是直接用Paint.setColor/ARGB()来设置颜色，另一种使用Shader来指定着色方案。


		android的绘制使用Shader，是使用它的子类：
		LinearGradient
		RadialGradient
		SweepGradient
		BitmapShader
		ComposeShader

####1.1.2 setShader 设置Shader
#####1.1.2.1 LinearGradient 线性渐变
	
	构造参数：
	x,y,x1,y1：渐变的两个断点的位置
	startColor,endColor：端点的颜色
	tile：断点范围之外的着色规则，类型是TileMode
		CLAMP 会在端点之外延续端点处的颜色
		MIRROR 镜像模式
		REPEAT 重复模式

#####1.1.2.2 RadialGradient  环形渐变

	参数：
	centerX centerY：辐射中心的坐标
	radius：辐射半径
	centerColor：辐射中心的颜色
	edgeColor：辐射边缘的颜色
	tileMode：辐射范围之外的着色模式。

#####1.1.2.3 SweepGradient  扫描渐变

	参数：
	cx cy ：扫描的中心
	color0：扫描的起始颜色
	color1：扫描的终止颜色

#####1.1.2.4 BitmapShader  着色
	
	参数：
	bitmap：用来做模板的 Bitmap 对象
	tileX：横向的 TileMode
	tileY：纵向的 TileMode

#####1.1.2.5 Compoase混合着色器
	所谓混合，就是把两个Shader一起使用
	
	参数：
	shaderA, shaderB：两个相继使用的 Shader
	mode: 两个 Shader 的叠加模式，即 shaderA 和 shaderB 应该怎样共同绘制。它的类型是 PorterDuff.Mode /Xfermode

	
	PorterDuff.Mode	
		用来指定两个图像共同绘制时颜色策略的。有17种。可以分为2类
		1，Alpha合成（Alpha Compositing）
		2，混合（Blending）
		{
		第一类：Alpha
			CLEAR       所绘制源图像不会提交到画布上。
			SRC         只显示源图像。
			SRC_OVER    正常绘制显示，源图像居上显示。
			SRC_IN      取两层绘制交集中的源图像。
			SRC_ATOP    在源图像和目标图像相交的地方绘制源图像，在不相交的地方绘制目标图像。
			SRC_OUT     只在源图像和目标图像不相交的地方绘制源图像。
			DST         只显示目标图像。
			DST_OVER    上下层都显示。目标图像居上显示。
			DST_IN      取两层绘制交集中的目标图像。
			DST_ATOP    在源图像和目标图像相交的地方绘制目标图像而在不相交的地方绘制源图像。
			DST_OUT     只在源图像和目标图像不相交的地方绘制目标图像。
			XOR          异或：去除两图层交集部分
		第二类：混合
			DARKEN      取两图层全部区域，交集部分颜色加深
			LIGHTEN     取两图层全部，点亮交集部分颜色
			MULTIPLY    取两图层交集部分叠加后颜色
			SCREEN      滤色。
			OVERLAY
		}

####以下图片是Shader的着色器合成混合样式参考，并不是Xfermode样式，请注意！
![](https://user-gold-cdn.xitu.io/2017/7/17/3444dfcd8745677dc668ffd94fd26cb8?imageView2/0/w/1280/h/960/ignore-error/1)	

Alpha通道合成类型  透明度计算  
![](https://user-gold-cdn.xitu.io/2017/7/18/9d09c00d9e4a1c54de678ee5823dd5c6?imageView2/0/w/1280/h/960/ignore-error/1)
混合类型  本身不是Alpha通道合成类型 但是也被归类到ProterDuffMode里面，
![](https://user-gold-cdn.xitu.io/2017/7/17/4be41991c90e3eacf92e9378168b681d?imageView2/0/w/1280/h/960/ignore-error/1)


### 1.2，setColorFilter(ColorFilter filter) 设置颜色过滤，统一过滤的策略，然后drawXXX()会对每个像素都进行过滤再绘制出来。
	
	使用Paint.setColorFilter(ColorFilter filter)，使用它的子类：
		1，LightingColorFilter
		2，ProterDuffColorFilter
		3，ColorMatrixColorFilter
	

####1.2.1 LightingColorFilter 模拟光照效果

	构造参数：	
		mul & add ：颜色值 mul用来和目标像素相乘；add用来和目标像素相加。

	算法：
		R' = R * mul.R / 0xff + add.R
		G' = G * mul.G / 0xff + add.G
		B' = B * mul.B / 0xff + add.B
	Alpha通道A是忽略的，所以随便设置什么都不会有变化的

####1.2.2 PorterDuffColorFilter 使用一个指定的颜色和一种指定的PorduffModel来与绘制对象合成
	
	构造参数：
		int color:颜色
		PorterDuffMode：合成模式类型

####1.2.3 ColorMatrixColorFilter 一个4*5的矩阵，可以把要绘制的像素进行转换

	[
	 a, b, c, d, e,
  	 f, g, h, i, j,
  	 k, l, m, n, o,
  	 p, q, r, s, t 
	]
![](https://user-gold-cdn.xitu.io/2017/7/17/8a67051ca54932f59934b36b7df44447?imageView2/0/w/1280/h/960/ignore-error/1)


### 1.3 setXfermode(Xfermode mode) 严格来说就是“Transfer mode” ,用X来代替“Trans”是美国人的简写方式，严谨来说就是你要绘制的内容和目标位置的内容应该怎么样结合计算出最终的颜色。

	
	canvas.drawXXX();
	Xfermode xfermode = new PorterDuffMode(PorterDuff.Mode.DST_IN);
	paint.setXfermode(mode);
	canvas.drawXXX();
	paint.setXfermode(null);//用完及时清理Xfermode


#### 1.3.1 这里又发现了PorterDuff.Mode 它在Paint有3处API，工作原理都是一样，只是用途不同。
	
	    API                       用途   
	ComposeShaer                      混合两个Shader
	PorterDuffColorFilter             增加一个单色的ColorFilter
	Xfermode（PorterDuffXfermode）     设置绘制内容和View中已有内容的混合计算方式
		
#### 1.3.2 使用Xfermode注意事项
官方效果：

![官方效果](http://img.blog.csdn.net/20160119134212634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

由于和官方demo的不一样，以下是实际测试得到的效果图！
![自己实际效果](http://upload-images.jianshu.io/upload_images/1311457-4641819c69a2b5fa?imageMogr2/auto-orient/strip%7CimageView2/2/w/440)
通过阅读以下第一篇文章，发现坑不少，除了SRC_IN和DST_IN，其他模式都是需要两个Bitmap才能生效。

要达到官方的效果 总结：

	1、关闭硬件加速。
	2、两个bitmap大小尽量一样。
	3、背景色为透明色。
	4、如果两个bitmap位置不完全一样，可能也是预期效果，只不过你看到的效果和你自己脑补的预期效果不一致。
	
	PorterDuffXfermode用于实现新绘制的像素与Canvas上对应位置已有的像素按照混合规则进行颜色混合。

PorterDuffXferMode不正确的真正原因PorterDuffXferMode深入试验：
[http://m.blog.csdn.net/wingichoy/article/details/50534175](http://m.blog.csdn.net/wingichoy/article/details/50534175 "PorterDuffXferMode不正确的真正原因PorterDuffXferMode深入试验")

Android中Canvas绘图之PorterDuffXfermode使用及工作原理详解：
[http://blog.csdn.net/iispring/article/details/50472485](http://blog.csdn.net/iispring/article/details/50472485 "Android中Canvas绘图之PorterDuffXfermode使用及工作原理详解")

#### 1.3.2.1  Canvas.saveLayer()可以做短时的离屏缓冲。
	
	
	int saved = canvas.saveLayer(null, null, Canvas.ALL_SAVE_FLAG);

	canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方
	paint.setXfermode(xfermode); // 设置 Xfermode
	canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆
	paint.setXfermode(null); // 用完及时清除 Xfermode

	canvas.restoreToCount(saved);


#### 1.3.2.2 View.setLayerType(type) 是直接把整个View都绘制在离屏缓冲中。

	
	参数：
	LAYER_TYPE_HARDWARE 使用GPU来缓冲
	LAYER_TYPE_SOFTWARE 直接使用一个Bitmap来缓冲

	
	注意：
		使用Xfermode来绘制的内容，除了注意使用离屏缓冲，还应该注意控制它的透明区域不要太小，要让它足够覆盖到要和它结合绘制的内容，否则得到的结果可能不是你想要的。


用这个图来说明这个注意的问题：（透明覆盖率太小，xfermode就没有影响）
![](https://user-gold-cdn.xitu.io/2017/7/17/35fc04baf9cb8598e347a9c5cf7601a1?imageView2/0/w/1280/h/960/ignore-error/1)




## 2：效果
	
	效果类的API，指的是抗锯齿、填充、轮廓、线条等等这些

### 2.1 paint.setAntiAlias(boolean) 

	设置抗锯齿
### 2.2 paint.setStyle(Paint.Style style) 

	用来设置图形是线条风格还是填充风格，或者二者并用。

### 2.3 线条形状

	1，setStorekeWidth(int)   线条形状
	2，setStorekeCap(Paint.Cap cap) 线头形状
		BUTT平头、ROUND圆头、SQUARE方头
	3，setStrokeJoin(Paint.Join join) 拐角形状 MITER尖角、BEVEL平角、ROUND圆角
    4，setStrokeMiter(float miter) 是对strokeJoin的补充，延长线的最大值
![](https://user-gold-cdn.xitu.io/2017/7/17/8a817db61864a298bdfbca27268c77f7?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/7/17/8fa8c7cfb47fa0f6d5d3c8c18635ae82?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/7/17/6a5f1de602276ce42cc5a7313a59fc08?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/7/17/36eeed61a7a0f1dc5b2d2db97e5b0557?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/7/17/a337179c3a87afcbf342256cb2e7f01f?imageView2/0/w/1280/h/960/ignore-error/1)
### 2.4 色彩优化
	
	作用：让图片看着更加顺眼
	
	paint.setDither(boolean)  图像的抖动
	paint.setFilterBitmap(boolean)  双线性过滤
![](https://user-gold-cdn.xitu.io/2017/7/17/efd7ea8e30164dc15492bac83dcd58bc?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/7/17/944479afd3ad2b8bff581ab0992173f4?imageView2/0/w/1280/h/960/ignore-error/1)

#####2.5 setPathEffect(PathEffect)
	
	使用PathEffect来给图形的轮廓设置效果。对Canvas所有图形的绘制有效，也就是drawLine、drawCircle、drawPath这些方法。
	6种PathEffect，分为两类：
		单一效果类：
			CornerPathEffect      拐角变成圆角
			DiscretePathEffect    把线条进行随机的偏离，让轮廓变得乱七八糟，程度由方式和参数决定
			DashPathEffect        
			PathDashPathEffect
		组合效果类：
			SumPathEffect
			ComposePathEffect
	注意：
		PathEffect有一些不支持硬件加速
	
	
#####2.5.1.1  CornerPathEffect：
		
	new CornerPathEffect(20)
![](https://user-gold-cdn.xitu.io/2017/7/17/4fcff0e4242359e80e05eb74e099dc2a?imageView2/0/w/1280/h/960/ignore-error/1)

#####2.5.1.2  DiscretePathEffect：
	
	PathEffect pathEffect = new DiscretePathEffect(float segmentLength, float deviation);
	segmentLength:用来拼接的每个线段的长度
	deviation：偏移量
![](https://user-gold-cdn.xitu.io/2017/7/17/d2fa819b64fe14dff09f679da3672eb4?imageView2/0/w/1280/h/960/ignore-error/1)

#####2.5.1.3  DashPathEffect：

	用虚线来绘制线条
	PathEffect pathEffect = new DashPathEffect(new float[]{20, 10, 5, 10}, 0);
	float[] intervals： 虚线的格式，数组元素至少2个，按照'画线长度-空白长度'来绘制
	float phas  ： 虚线的偏移量
![](https://user-gold-cdn.xitu.io/2017/7/17/23efecea4e7391ce791882e4452d5555?imageView2/0/w/1280/h/960/ignore-error/1)


#####2.5.1.4  PathDashPathEffect  ：
	
	使用一个Path来绘制虚线
	PathDashPathEffect(Path shape, float advance, float phase, PathDashPathEffect.Style style)
	
	Path shape：     用来绘制的path
	float advance：  两个相邻的shape段之间的间隔
	float phase：    虚线的偏移量
	PathDashPathEffect.Style style  指定拐弯改变的时候shape的转换方式，有三种方式：
		TRANSLATE：位移
		ROTATE：旋转
		MORPH ：变体

![](https://user-gold-cdn.xitu.io/2017/7/17/3edcd8c5020b43350ead9123e70a78ac?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/7/17/bc7d0841e886282ed7c43622d28f6f24?imageView2/0/w/1280/h/960/ignore-error/1)

#####2.5.1.5  SumPathEffect：组合效果类的PathEffect


	很简单的行为，就是分别按照两种PathEffect分别对目标进行绘制

![](https://user-gold-cdn.xitu.io/2017/7/17/bb1fb1c35ec6a453cf486241e631ba5f?imageView2/0/w/1280/h/960/ignore-error/1)

#####2.5.1.6  ComposePathEffect 

	先对目标path使用一个PathEffect，然后再对这个改变后的Path使用另外一个PathEffect。
	PathEffect outerpe 先应用, PathEffect innerpe后应用

![](https://user-gold-cdn.xitu.io/2017/7/17/92988c0ef11a0070e457b33ad78b854a?imageView2/0/w/1280/h/960/ignore-error/1)


#####2.6 setShadowLayer(float radius,float dx,float dy,int shadowColor)  在之后绘制的内容下面加一层阴影。

	清除阴影，使用paint.clearShadowLayer();
	注意：
		在硬件加速开启的情况下，setShadowLayer只支持文字的绘制，文字外的绘制必须关闭硬件加速。
		如果shadowColor是半透明的，阴影的透明度就使用shadowColor自己的透明度，而如果shadowColor是不透明，阴影透明度就使用paint的透明度。


#####2.7 setMaskFilter(MaskFilter)  在绘制层上方的附加效果。
		
	有两种：
		BlurMaskFilter  模糊效果
		EmbossMaskFilter  
2.7.1 BlurMaskFilter 
	
	BlurMaskFilter(float radius, 
	BlurMaskFilter.Blur style）
	radius：模糊范围
	style：
		NORMAL:内外都模糊
		SOLID:内部正常绘制，外部模糊
		INNER：内部模糊，外部不绘制
		OUTER：内部不绘制，外部模糊
	
![](https://user-gold-cdn.xitu.io/2017/7/17/666f2a37a1fdfb6895608e16b72fa6ec?imageView2/0/w/1280/h/960/ignore-error/1)

2.7.2  EmbossMaskFilter 浮雕效果

	EmbossMaskFilter(float[] direction, float ambient, float specular, float blurRadius)

	direction:一个3个元素的数组，指定了光源的方向
	ambient：环境光的强度，范围0-1
	specuar：炫光的系数
	blurRadius：应用光线的范围

#####2.8 获取绘制的Path
	根据paint的设置，计算出绘制Path或文字的实际Path
2.8.1 getFillPath(Path src,Path dst)
	
	所谓实际Path：指的是drawPath的绘制内容的轮廓，要算算行线条宽度和设置的PathEffect。
	
![](https://user-gold-cdn.xitu.io/2017/7/17/e80d504826125eef2bd9976bf320e05d?imageView2/0/w/1280/h/960/ignore-error/1)

	
	通过getFillPath获取到这个实际Path，方法的参数，src是原path，dst就是实际path的保存位置。计算后，把结果保存到dst里。

2.8.2 1,getTextPath(String text,int start,int end,float x,float y,Path path)   2,getTextPath(char[] text,int index,int count,float x,float y,Path path)
![](https://user-gold-cdn.xitu.io/2017/7/17/be85350bd7bea835bb02f73e0c6fd9f6?imageView2/0/w/1280/h/960/ignore-error/1)
如下图，可以用做图形和文字的装饰效果的位置计算，比如自定义的下划线效果
![](https://user-gold-cdn.xitu.io/2017/7/17/a9a6dd9c3d140da7cbc6d7b713696830?imageView2/0/w/1280/h/960/ignore-error/1)


## 3，drawText（）相关
	
	Paint有些设置是文字绘制相关的，即和drawText()相关的

比如设置文字大小：
![](https://user-gold-cdn.xitu.io/2017/7/17/8d7981e94d43841ec6c6b97b840927c2?imageView2/0/w/1280/h/960/ignore-error/1)
比如设置文字间隔：
![](https://user-gold-cdn.xitu.io/2017/7/17/6e30f7a5328f542e7b351ac5f611393b?imageView2/0/w/1280/h/960/ignore-error/1)
比如设置各种文字效果：
![](https://user-gold-cdn.xitu.io/2017/7/17/dbdd0a0239a8251dba0910af45965a2d?imageView2/0/w/1280/h/960/ignore-error/1)

## 4，初始化类
### 4.1  reset()重置Paint的所有属性为默认值，相当于重新new一个，不过性能当然高一些。
### 4.2  set（Paint src）把src的所有属性全部赋值过来。
### 4.3  setFlags（int flags）  鼻梁设置flags，相当于依次调用它们的set方法。
	
	paint.setFlags(Paint.ANTI_ALIAS_FLAG | Paint.DITHER_FLAG);


