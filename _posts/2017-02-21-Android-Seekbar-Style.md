---
layout: post
title: Android Seekbar style
author: magic
date:   2017-02-21
categories: Android
tags: android
permalink: /archivers/Android-Seekbar-Style
---
* 目录
{:toc}
## Android仿酷狗音乐SeekBar——样式篇
需求：仿酷狗音乐SeekBar
直接上图，上代码
![1903148-676fcbf2e5048392.png](http://upload-images.jianshu.io/upload_images/1903148-0414a84bd91e4ddc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![1903148-d3e5ab81fa2acd42.png](http://upload-images.jianshu.io/upload_images/1903148-d7a9c2b37322d59d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**难点：用户点击或者移动是SeekBar滑块是改变滑块的图案**
### 1. 先画两种不同状态的滑块Thumb
平时状态：一个直径为10dp大小的白色的圆

slider_thumb_normal.xml

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"    
  android:shape="oval">    
  <size    
    android:width="10dp"    
    android:height="10dp" />
  <solid android:color="#ffffffff" />
</shape>
```

按下状态：一个直径为10dp大小的白色的圆，背景是半透明的直径为40dp的圆

slider_thumb_pressed.xml

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 白色前景 -->
    <item android:id="@android:id/secondaryProgress"
        android:gravity="center"
        android:bottom="15dp"
        android:top="15dp"
        android:right="15dp"
        android:left="15dp">
        <shape
            android:shape="oval">
            <size
                android:width="10dp"
                android:height="10dp" />
            <solid android:color="#ffffffff" />
        </shape>
    </item>
    <!-- 透明阴影 -->
    <item android:id="@android:id/background"
        android:gravity="center">
        <shape
            android:shape="oval">
            <size
                android:height="40dp"
                android:width="40dp" />
            <solid android:color="#96ffffff" />
        </shape>
    </item>
</layer-list>
```

### 2. 画进度条

(不设置高度,由SeekBar自身控制,SeekBar控件android:layout_height="wrap_content")
play_seekbar_bg.xml

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@android:id/background">
        <shape>
            <size
                android:height="2dp" />
            <corners android:radius="5dp"/>
            <solid android:color="#88ffffff" />
        </shape>
    </item>
    <item
        android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <size
                    android:height="2dp" />
                <corners android:radius="5dp"/>
                <solid android:color="#88ffffff" />
            </shape>
        </clip>
    </item>
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <size
                    android:height="2dp"/>
                <corners android:radius="5dp"/>
                <solid android:color="#ffffffff" />
            </shape>
        </clip>
    </item>
</layer-list>
```

### 3. SeekBar样式xml片段
```
<?xml version="1.0" encoding="UTF-8"?>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="8dp">
        <TextView
            android:layout_gravity="center"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:id="@+id/seekbar_slider_time"
            android:textColor="@color/color_white"
            android:textAppearance="?android:attr/textAppearanceMedium"
            android:visibility="invisible" />
    </LinearLayout>
    <LinearLayout
        android:id="@+id/seekBar_layout"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:layout_marginStart="8dp"
        android:layout_marginEnd="8dp"
        android:gravity="center_vertical"
        android:orientation="horizontal">
        <TextView
            android:id="@+id/tx_currentTime"
            android:layout_width="40dp"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textAppearance="?android:attr/textAppearanceSmall"
            android:textColor="@color/color_white"/>
        <SeekBar
            android:id="@+id/seedBar"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="-20dp"
            android:layout_marginEnd="-20dp"
            android:paddingStart="28dp"
            android:paddingEnd="28dp"
            android:background="@android:color/transparent"
            android:gravity="center"
            android:layout_weight="1"
            android:splitTrack="false"
            android:maxHeight="2dp"
            android:progressDrawable="@drawable/play_seekbar_bg"
            android:thumb="@drawable/slider_thumb_normal" />
        <TextView
            android:id="@+id/tx_maxTime"
            android:layout_width="40dp"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textAppearance="?android:attr/textAppearanceSmall"
            android:textColor="@color/color_white"/>
    </LinearLayout>
```

**SeekBar样式关键点**
- android:maxHeight="2dp"——控制进度条高度
- 设置SeekBar控件边际，以便在滑块变大是可覆盖左右两边的控件，而不会被遮住     

```
android:layout_marginStart="-20dp"
android:layout_marginEnd="-20dp"
android:paddingStart="28dp"
android:paddingEnd="28dp"
```

- android:splitTrack="false"——控制滑块覆盖在进度条的上面
- android:background="@android:color/transparent"——设置背景透明，去掉滑块变大时的周边光晕
- android:progressDrawable="@drawable/play_seekbar_bg"——默认进度条
- android:thumb="@drawable/slider_thumb_normal"——默认滑块

## 最关键的地方
使用SeekBar的setThumb方法动态设置滑块
代码
{% highlight java %}
seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if(fromUser){
                    Seekbar_slider_time.setText(updateCurrentTimeText(progress));
                }
                tx_currentTime.setText(updateCurrentTimeText(progress));
                if(progress == seekBar.getMax()){
                    pauseIcon.setLayoutParams(miss);
                    playIcon.setLayoutParams(show);
                }
            }
            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {
                Log.d(TAG,"onStartTrackingTouch");
                isUserPressThumb = true;
                Seekbar_slider_time.setVisibility(View.VISIBLE);
                //设置seekbarThumb相对位置可大于进度条15，保证thumb在变成40dp直径后可以滑动到进度条最末尾
                seekBar.setThumbOffset(15);
                seekBar.setThumb(Thumb_pressed);
            }
            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                Log.d(TAG,"onStopTrackingTouch");
                mi.seekTo(seekBar.getProgress());
                seekBar.setThumbOffset(0);
                seekBar.setThumb(Thumb_normal);
                Seekbar_slider_time.setVisibility(View.INVISIBLE);
                isUserPressThumb = false;
            }
        });
{% endhighlight %}
#### 在用户开始按下滑块时onStartTrackingTouch
//设置seekbarThumb相对位置可大于进度条15，保证thumb在变成40dp直径后可以滑动到进度条最末尾 
seekBar.setThumbOffset(15); 
//改变滑块图案
seekBar.setThumb(Thumb_pressed);

#### 在用户按下滑块结束后onStopTrackingTouch，恢复滑块及seekbar高度
seekBar.setThumbOffset(0); 
seekBar.setThumb(Thumb_normal);

## _踩坑过程_
*使用selector的xml文件设置SeekBar的android:thumb属性设置滑块*
play_seekbar_thumb.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"    
   android:constantSize="false"    
   android:variablePadding="false">    
   <item android:state_focused="true" android:state_pressed="false" android:drawable="@drawable/slider_thumb_normal" />    
   <item android:state_focused="true" android:state_pressed="true" android:drawable="@drawable/slider_thumb_pressed" />    
   <item android:state_focused="false" android:state_pressed="true" android:drawable="@drawable/slider_thumb_pressed" />    
   <item android:drawable="@drawable/slider_thumb_normal" />
</selector>
```

**坑：有些手机上按下或者移动滑块，滑块是变大了，但是由于SeekBar高度还是原来的，导致滑块被压扁成椭圆**

项目地址[https://github.com/magic5650/Recoderapp](https://github.com/magic5650/Recoderapp "magic5650 github Recoderapp")