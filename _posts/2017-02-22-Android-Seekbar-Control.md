---
layout: post
title: Android Seekbar Control
author: magic
date:   2017-02-22
categories: Android
tags: android seekbar
permalink: /archivers/Android-Seekbar-Control
---
# Android仿酷狗音乐SeekBar——控制篇
### 需求:
1. 根据后台播放状态调整SeekBar的滑块位置;
2. 反馈用户的滑动滑块事件;

### 分析：
一般我们的视频或者音乐播放是由后台Service播放的，而SeekBar是在前台Activity或者Fragment里，所以根据播放状态我们调整SeekBar滑块可以让Service主动发送数据给前台，而反馈用户滑块事件，直接在前台获得Service实例，然后操作相关控制媒体播放的方法即可。
#### 1. 后台Service给前台发送媒体播放进度

这里我们使用Timer新建一个Timertask，间隔执行，然后在暂停或者结束后停止发送

{% highlight java %}
public void SeedPlayMsg(){
        if(timer == null) {
            Log.d(TAG, "创建timer对象");
            timer = new Timer();//timer就是开启子线程执行任务，与纯粹的子线城不同的是可以控制子线城执行的时间，
        }
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                //获取歌曲当前播放进度
                int currentPosition = player.getCurrentPosition();
                Message msg = AudioPlayActivity.handler.obtainMessage();
                Bundle bundle = new Bundle();
                msg.what = 1;
                bundle.putInt("currentPosition", currentPosition);
                bundle.putBoolean("isPlayComplete", false);
                if (player.isPlaying()) {
                    //把进度封装至消息对象中
                    msg.setData(bundle);
                    AudioPlayActivity.handler.sendMessage(msg);
                }
                else {
                    if (isPlayComplete) {
                        bundle.putBoolean("isPlayComplete", true);
                        Log.d(TAG, "发送消息给主线程,播放已结束");
                    }
                    else{
                        bundle.putBoolean("isPlayComplete", false);
                    }
                    msg.setData(bundle);
                    AudioPlayActivity.handler.sendMessage(msg);
                    Log.d(TAG, "结束TimeTask");
                    timer.cancel();
                }
            }
            //开始计时任务后的5毫秒后第一次执行run方法，以后每500毫秒执行一次
        }, 200, 500);
    }
{% endhighlight %}

*这里有个很重要的细节，就是发送播放结束的消息，一般我们认为当发送的位置等于媒体的长度时，我们就认为是结束了，可事实上不是这样的，因为MediaPlayer对象的getCurrentPosition()方法在媒体播放结束后获取到的值也与获取时常的方法getDuration()得到的值不一致，总是要小，而且还不固定。*
**所以为了精确播放是否完成，我们还是由原生接口监听去判断播放是否完成**
{% highlight java %}
        player.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            @Override
            public void onCompletion(MediaPlayer mp) {
                Log.d(TAG,"播放结束");
                isPlayComplete = true;
            }
        });
{% endhighlight %}
#### 前台Activity或者Fragment接收当前媒体播放进度信息并，调整SeekBar进度
{% highlight java %}
    static Handler handler = new Handler(){//handler是谷歌说明的定义成静态的，
        public void handleMessage(Message msg) {
            switch (msg.what) {
                default:
                case 1:
                    Bundle bundle = msg.getData();
                    boolean isPlayComplete = bundle.getBoolean("isPlayComplete");
                    int currentPosition = bundle.getInt("currentPosition");
                    //根据当前位置判断录音是否结束，结束后更新UI
                    if (isPlayComplete) {
                        seekBar.setProgress(seekBar.getMax());
                    }
                    else {
                        if (!isUserPressThumb) seekBar.setProgress(currentPosition);
                    }
                    break;
                case 2:
                    Bundle bundle2 = msg.getData();
                    int duration = bundle2.getInt("duration");
                    seekBar.setMax(duration);
                    break;
            }
        }
    };
{% endhighlight %}
*这里有个很重要的细节，和容易忽略，就是判断用户是否在拖动进度条，如果用户在拖动中，那么就执行setProgress；这个如果不做判断，在一些手机上会出现用户在媒体播放时拖动进度条还没放开，滑块跑到setProgress设置的位置上，然后马上又跑回用户拖动的位置上，很坑爹。*
**所以这里我们加一个判断if (!isUserPressThumb)**
前面我们说了getCurrentPosition()方法返回的数值（单位是毫秒）永远比getDuration()要小，而我们进度条的最大值是用getDuration()方法获取的值设置的，所以为了一致，我们要在接收到播放完成的消息后，将SeekBar调到最大值
**if (isPlayComplete) { seekBar.setProgress(seekBar.getMax()); }**
在媒体准备好后，我们要发送媒体的长度给前台，使用这个设置前台SeekBar的最大长度，当然，你也可以将SeekBar最大长度设置为某个值（默认100），然后跟进媒体播放进度进行等比换算，在这里我们不换算了，直接将媒体的长度设置为SeekBar的长度，简单易懂。
{% highlight java %}
    public void SeedPlayDuration() {
        Message msg = AudioPlayActivity.handler.obtainMessage();
        msg.what = 2;
        //把进度封装至消息对象中
        Bundle bundle = new Bundle();
        bundle.putInt("duration", getDuration());
        msg.setData(bundle);
        AudioPlayActivity.handler.sendMessage(msg);
    }
{% endhighlight %}
结合上面的前台Handler接收到消息，设置SeekBar最大长度。
*（为保证后续开始播放后发送进度调整SeekBar不会超出SeekBar原来的最大长度，所以前一篇的定时发送位置的任务，我们是延迟200ms进行的）*
#### 2. 反馈用户的滑动滑块事件
由前台监听调用后台Service方法完成

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
                if(!(seekBar.getProgress() == seekBar.getMax())) isUserPressThumb = false;
            }
        });
{% endhighlight %}

主要是在onStopTrackingTouch中，也就是用户点击或滑动滑块完成阶段执行
mi.seekTo(seekBar.getProgress());——这里mi是获取到的Service对象，seekTo是设置当前播放位置的方法。

在onStartTrackingTouch中，也就是用户开始点击或者滑动滑块开始阶段，我们设置
isUserPressThumb = true;
以配合前面Handler中控制SeekBar位置的代码，原因我前面已经说了。
最后在onStopTrackingTouch中，设置
 if(!(seekBar.getProgress() == seekBar.getMax())) isUserPressThumb = false;(如果没有拖动到最后)

至于在onProgressChanged中，用户点击或者拖动滑块中

{% highlight java %}
                if(fromUser){
                    Seekbar_slider_time.setText(updateCurrentTimeText(progress));
                }
                tx_currentTime.setText(updateCurrentTimeText(progress));
                if(progress == seekBar.getMax()){
                    pauseIcon.setLayoutParams(miss);
                    playIcon.setLayoutParams(show);
                }
{% endhighlight %}

需要注意的是boolean fromUser这个参数，为true时，表示是用户改变的滑块位置，false时，是系统改变的，也就是我们前面Handler中进行的改变；
如果是用户拖动，我们相应的在滑块上面给用户提示当前到什么位置了
Seekbar_slider_time.setText(updateCurrentTimeText(progress));
当然，这个控件平时是隐藏的，所以点击开始时onStartTrackingTouch，我们需要显示它
Seekbar_slider_time.setVisibility(View.VISIBLE);
拖动或点击结束后onStopTrackingTouch，我们由隐藏它
Seekbar_slider_time.setVisibility(View.INVISIBLE);

在onProgressChanged我们设置显示媒体的位置，代表播放到哪里了，或者即将播放到哪里了
tx_currentTime.setText(updateCurrentTimeText(progress));

另外当滑块滑动到最后了以后（由前面Handler接收到播放完成后设置的，也可能是用户自己拖动到了最后）
我们需要变换播放或者暂停按钮
if(progress == seekBar.getMax())
{ 
pauseIcon.setLayoutParams(miss);
playIcon.setLayoutParams(show); 
}

附上毫秒装换为时间格式hh:mm:ss的方法

{% highlight java %}
public String updateCurrentTimeText(int currentPosition){
        try {
            int time= currentPosition/1000;
            if (time >= 3600) {
                int hour = time/3600;
                int minute = (time/60) % 60;
                int second = time % 60;
                String Time = String.format(Locale.getDefault(),"%02d:%02d:%02d", hour, minute, second);
                //tx_currentTime.setText(currentTime);
                return  Time;
            }
            else{
                int minute = time/60;
                int second = time % 60;
                String Time = String.format(Locale.getDefault(),"%02d:%02d", minute, second);
                //tx_currentTime.setText(currentTime);
                return  Time;
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        return null;
    };
{% endhighlight %}

项目地址[https://github.com/magic5650/Recoderapp](https://github.com/magic5650/Recoderapp "magic5650 github Recoderapp")