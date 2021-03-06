---
layout: post
title: "안드로이드 | 노트3 - 2015.11.02"
modified: 2015-11-19T23:39:55-04:00
categories: android
excerpt:
tags: [android]
image:
  feature:
date: 2016-04-04
---

<figure>
    <img src="/images/lifecycle.png" alt="image">
</figure>

#### Activity Life Cycle 다시보기
OnCreate() - 최초생성될때 호출, UI 구성 ( setContentView() )<br>
OnStart() - 거의 사용x<br>
OnReStart() - 거의 사용x<br>
OnResume() - 실행상태 전에 호출 보장 <br>
OnPause() - 종료시점의 호출 보장( 종료시 꼭 해야할 처리 )<br>
OnStop() - 거의 사용x<br>
OnDestroy() - 거의 사용x<br>

<br>
<br>
<br>
<br>

#### Fragment Life Cycle 다시보기
최초 화면이 뜰때, onCreateView() -> onViewCreated() -> onStart() -> onResume()
프레그먼트 위로 다른 액티비티가 뜰때, onStop()
위에 있던 액티비티가 사라질때, onStart() -> onResume()

<br>
<br>
<br>
<br>

#### onReceive 처리시 주의할점
브로드캐스트리시버에서 onReceive 에서 받아서 처리를 할경우 이 함수는 UI 스레드에서 처리되기 때문에, 시간이 오래 걸리는 작업을 하면 안된다. 따라서 별도의 쓰레드를 뺴서 처리를 하거나 IntentService 를 이용하여 처리해야 한다.

<br>
<br>
<br>
<br>

#### IntentService 로 구현하기
서비스는 기본적으로 UI 쓰레드에서 동작을 한다. 하지만 IntentService 는 별도의 작업쓰레드에서 처리가 되며, onHandleIntent 에서 처리가 된다. 빠른 시간에 여러번 요청이 오면 병렬처리가 되는게 아니라 스택처럼 직렬로 처리가 된다.
노티피케이션이나 알람받았을때 처리하는 동작을 하면 좋을듯 하다.

<br>
<br>
<br>
<br>

#### static 변수를 final 로 할당하는 이유
정적변수로 어떤 값을 할당해놓는 경우가 있다. 여기서 final 로 안해주고 그냥 정적변수로 해주면은 만약 프로세스가 죽으면서 메모리가 날라가게되면 이 정적변수들의 값다 다 null 로 되버릴수가 있다. 따라서 final 로 저장을 해줘야 저런경우에서도 다시 로드가 되더라도, 예외가 발생하지 않고 제대로 동작할수가 있다.

<br>
<br>
<br>
<br>

#### ScrollView 에서 제일밑에까지 스크롤이 안된다.
스크롤뷰의 하위에 Linear 또는 Relative 를 두고, 이 뷰들의 margin 을 주면 위, 왼쪽, 오른쪽은 잘 먹히는데 marginBottom 은 제대로 먹지를 않는다. 그래서 스크롤이 원하는대까지 밑으로 내려가지 않는 버그가 있다. 구글링결과 안드로이드 버그인걸로 보이고 magin 대신에 padding 을 주어야 될것 같다. 하지만 내가 만든뷰에서는 해당 자식뷰의 배경스타일이 있어 margin 을 padding 으로 바꿀수가 없다. 그래서 약간의 꼼수로 제일밑에 그냥 더미 view 로 height 만있는 투명한 뷰를 만들어 margin 의 역활을 할수 있도록 하였다.

<br>
<br>
<br>
<br>

#### Theme , 전체화면 설정하기
{% highlight java %}
<item name="android:windowFullscreen">true</item>
{% endhighlight %}
: 상태바가 사라진다 / 전체 화면

{% highlight java %}
<item name="android:windowTranslucentStatus">true</item>
{% endhighlight %}
: 상태바는 보이되, 뷰영역이 상태바 까지 모두 포함되어 겹쳐서 보이게 된다
 -> margin 을 위쪽에 두어 사용해야됨

<br>
<br>
<br>
<br>

#### SimpleDateFormat 사용시 주의사항
DateForamtting 을 위해 SimpleDateFormat 을 많이 사용한다. 하지만 이것을 쓸때 주의사항으로 쓰레드 세이프 하지 않을수가 있다.

``` java
SimpleDateFormat dateFormat_String = new SimpleDateFormat( "yyyy.MM.dd", Locale.getDefault() );
// 이 객체는 `foramt()` 함수와 `parse()` 함수를 지원한다.
```
같은 객체를 쓸경우 예상치 못한 에러가 발생할수 있어, 포메팅 스트링만 따로 빼두고 포멧팅 할때마다 새로운 객체를 생성 하는것이 안전하다.

<br>
<br>
<br>
<br>

#### Alpha 값 설정

100% — FF<br>
95% — F2<br>
90% — E6<br>
85% — D9<br>
80% — CC<br>
75% — BF<br>
70% — B3<br>
65% — A6<br>
60% — 99<br>
55% — 8C<br>
50% — 80<br>
45% — 73<br>
40% — 66<br>
35% — 59<br>
30% — 4D<br>
25% — 40<br>
20% — 33<br>
15% — 26<br>
10% — 1A<br>
5% — 0D<br>
0% — 00<br>

<br>
<br>
<br>
<br>

#### Screen 강제로 꺠우기
activity 또는 fragment 에  window 객체에 flag 를 설정해주면 된다. <br>
(하지만 htc 단말에서 2.3.5 에서 안된다고 한다.)

``` java
val window = activity.window

window.addFlags(WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED)
window.addFlags(WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD)
window.addFlags(WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON)
```

아래와 같이 하면 안좋다고 하는것 같은데 .. <br>

``` java
KeyguardManager km = (KeyguardManager) context.getSystemService(Context.KEYGUARD_SERVICE);
final KeyguardManager.KeyguardLock kl = km.newKeyguardLock("MyKeyguardLock");
kl.disableKeyguard();

PowerManager pm = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
wakeLock = pm.newWakeLock(
    PowerManager.FULL_WAKE_LOCK |
    PowerManager.ACQUIRE_CAUSES_WAKEUP |
    PowerManager.ON_AFTER_RELEASE, "MyWakeLock");
wakeLock.acquire();
```
