### 统一定义Activity的切换动画

> 在应用程序中，在两个Activity之间切换可以通过调用overridePendingTransition(enterAnim, exitAnim)方法来实现当前Activity的退出动画以及下一个显示Activity的进入动画。这种方式可以实现切换，但是每一次的Activity切换都需要在每个Activity中调用这个方法，比较麻烦。可以通过设置主题，来控制一个应用中所有的Acitivty的切换动画。

###### 两个切换动画定义

	activity_enter_anim.xml Activity进入时的动画

	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android" >

    <alpha
        android:duration="500"
        android:fromAlpha="0"
        android:toAlpha="1" />

    <scale
        android:duration="500"
        android:fromXScale="0"
        android:fromYScale="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1"
        android:toYScale="1" />

	</set>


	activity_out_anim.xml	Activity退出时的动画
	
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android" >

    <alpha
        android:duration="500"
        android:fromAlpha="1"
        android:toAlpha="0" />

    <scale
        android:duration="500"
        android:fromXScale="1"
        android:fromYScale="1"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0"
        android:toYScale="0" />

	</set>

###### styles.xml文件中样式的设置

	<style name="ThemeActivity" mce_bogus="1" parent="android:Theme.Light">
        <item name="android:windowAnimationStyle">@style/AnimationActivity</item>
        <item name="android:windowNoTitle">true</item>
    </style>

    <style name="AnimationActivity" mce_bogus="1" parent="@android:style/Animation.Activity">
        <item name="android:activityOpenEnterAnimation">@anim/activity_enter_anim</item>
        <item name="android:activityOpenExitAnimation">@anim/activity_out_anim</item>
        <item name="android:activityCloseEnterAnimation">@anim/activity_enter_anim</item>
        <item name="android:activityCloseExitAnimation">@anim/activity_out_anim</item>
    </style>


##### 将定义的样式应用在清单文件的Application的theme属性上

	<application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/ThemeActivity" >


通过以上3步，就可以定义一个应用程序，所以默认的Activity切换方式。