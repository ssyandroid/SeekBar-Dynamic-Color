# SeekBar-Dynamic-Color

# Android 动态改变SeekBar进度条颜色与滑块颜色
遇到个动态改变SeekBar进度条颜色与滑块颜色的需求，如图：


有的是根据不同进度改变成不同颜色。

对于这个怎么做呢？大家都知道设置下progressDrawable与thumb即可，但是这样设置好就是确定的了，要动态更改需要在代码里实现。

>- 用shape进度条与滑块

>- SeekBar设置

>- 代码里动态设置setProgressDrawable与setThumb

画图形，大家都比较熟悉，background是背景图，secondaryProgress第二进度条，progress进度条：

		<layer-list xmlns:android="http://schemas.android.com/apk/res/android"> 
		    <item android:id="@android:id/background">
		        <shape>
		            <corners android:radius="5dip" />
		            <gradient
		                android:startColor="@color/gray_cc"
		                android:centerColor="@color/gray_cc"
		                android:centerY="0.75"
		                android:endColor="@color/gray_cc"
		                android:angle="270"/>
		        </shape>
		    </item>
			< !-- 我的没有第二背景，故第二背景图没有画 -->
		    <item android:id="@android:id/secondaryProgress">
		        <clip>
		            <shape>
		                <corners android:radius="5dip" />
		                <gradient
		                    android:startColor="#80ffd300"
		                    android:centerColor="#80ffb600"
		                    android:centerY="0.75"
		                    android:endColor="#a0ffcb00"
		                    android:angle="270"/>
		            </shape>
		        </clip>
		    </item>
		    <item android:id="@android:id/progress">
		        <clip>
		            <shape>
		                <corners android:radius="5dip" />
		                <gradient
		                    android:startColor="@color/gray_cc"
		                    android:centerColor="@color/gray_cc"
		                    android:centerY="0.75"
		                    android:endColor="@color/gray_cc"
		                    android:angle="270"/>
		            </shape>
		        </clip>
		    </item>
		</layer-list>

然后画滑块：

		<selector xmlns:android="http://schemas.android.com/apk/res/android">
		    <item android:drawable="@drawable/seekbar_thumb_gray" android:state_focused="true" android:state_pressed="true" />
		    <item android:drawable="@drawable/seekbar_thumb_gray" android:state_focused="false" android:state_pressed="false" />
		    <item android:drawable="@drawable/seekbar_thumb_gray" android:state_focused="true" android:state_pressed="false" />
		    <item android:drawable="@drawable/seekbar_thumb_gray" android:state_focused="true" />
		</selector>

最后xml里设置：

		android:progressDrawable="@drawable/seekbar_light"
        android:thumb="@drawable/seekbar_thumb"

这里提醒下滑块滑到0或者最大可能展示不完，滑块只展示一半；还有有的背景太高；所以这些需要设置一下即可：

		android:maxHeight="5dp"
        android:minHeight="5dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"

上面maxHeight与minHeight可以让背景不那么高，更改值可以控制高度。
paddingLeft与paddingRight最好是滑块的宽度一半，这样即可展示完全。

下面进入正题就是代码里动态设置颜色。


1. 我们在代码里可以使用getProgressDrawable得到进度背景，所以可以以此来更改。
由于得到的是LayerDrawable，包含多个层次，当然上面我们只画了三个层次背景；不确定，可以循环进行得到ID进行判断：

		//获取seerbar层次drawable对象
        LayerDrawable layerDrawable = (LayerDrawable) sb.getProgressDrawable();
        // 有多少个层次（最多三个）
        int layers = layerDrawable.getNumberOfLayers();
        Drawable[] drawables = new Drawable[layers];
        for (int i = 0; i < layers; i++) {
            switch (layerDrawable.getId(i)) {
                // 如果是seekbar背景
                case android.R.id.background:
                    drawables[i] = layerDrawable.getDrawable(0);
                    break;
                // 如果是拖动条第一进度
                case android.R.id.progress:
					//这里为动态的颜色值
                    drawables[i] = new PaintDrawable(progressColor);
                    drawables[i].setBounds(layerDrawable.getDrawable(0).getBounds());
                    break;
				...
            }
        }
        sb.setProgressDrawable(new LayerDrawable(drawables));
		sb.setThumb(thumb);
 		sb.invalidate();

上面可以得到不同的背景，然后动态进行更改设置即可，上面代码可以完成最上面的图，因为背景不需要变化颜色，所以背景图不做变化，然后拿到进度条背景后我们采用的是PaintDrawable画颜色，当然你可以采用其他的构造一个背景Drawable，然后设置你需求的样式。滑块也可以是背景图片，或者给进度背景差一样可以使用getThumb到到背景重新设置即可。

2. 我上篇文章提到过Android更改纯色背景图片颜色，不清楚的可以点我进去查看[Android更改纯色背景图片颜色](http://blog.csdn.net/susanyuanaijia/article/details/53183403)。
因为我们这里也是纯颜色，这就更好办，更简单：
	
		//获取seerbar层次drawable对象
        LayerDrawable layerDrawable = (LayerDrawable) sb.getProgressDrawable();
		//因为画背景图时候第二进度背景图没有画,所以直接为1
		Drawable drawable = layerDrawable.getDrawable(1);
        drawable.setColorFilter(progressColor, PorterDuff.Mode.SRC);
		//获取滑块背景
        Drawable thumb = sb.getThumb();
        thumb.setColorFilter(thumbColor, PorterDuff.Mode.SRC);
        sb.invalidate();
由上我们可以利用更改纯色方法改得到的背景颜色，当然前提是你们背景图也是纯色。
这样又可以愉快玩耍了。
### 注意 ###
1. 第一种方案可以画多种样式，主要看你的对Drawable的使用了，但是如果对背景图，如果进度左右两端有圆弧，动态画图时候需要格外设置圆弧，估计实现也不简单吧，不然背景图是没有圆弧的。
2. 第二种方案对纯色的可以使用，对复杂多样式很难行得通。
