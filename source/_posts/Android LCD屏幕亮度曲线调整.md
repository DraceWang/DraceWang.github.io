---
title: Android LCD屏幕亮度曲线调整   
categories: Android  
tags: 
	- LCD 
	- BSP 
date: 2019-02-13
---

![post-cover](https://s2.loli.net/2023/09/21/UswxOvhH9yug4LT.jpg)


由于公司项目比较急，我也是刚开始接触源码上修改代码，但是正因如此这次的需求来了之后对Android屏幕亮度曲线调节有了一个新的认知，记录下来也算时一种经验分享。我会从自动和手动两个方面来说明，最后在补充一些使用的测试仪器时候的一些心得吧。

---

## 1.对Android源码背光亮度方面的研究

公欲善其事，必先利其器。所以首先我们先来看下源码。同样我们也时分为两个部分，手动和自动。至于为啥要分开说，其实研究过源码后，其实就知道了，源码就是通过自动背光的开关，直接把两个部分分开的。因此首先要纠正一个错误的观点，那就是：“手动背光曲线是什么样的，开了自动背光开关后，环境光一定的情况下，滑动亮度调节进度条，这时候的曲线和手动背光时候的曲线是一样的。”这个想法时完完全全错了，手动和自动背光就是分开的，两种情况下的背光曲线没有任何关联，当然定制化的不算，我们仅说源码角度(基于Android7.1),说起来也奇怪，自动背光这部分的代码在8.0之后又调整了算法，我们暂时不管它，下面讲到的时候再具体说明。
### 1.1 基础知识普及

 1. pwm 脉冲宽度调制（占空比），基本原理：控制方式就是对逆变电路开关器件的通断进行控制，使输出端得到一系列幅值相等的脉冲，用这些脉冲来代替正弦波或所需要的波形。也就是在输出波形的半个周期中产生多个脉冲，使各脉冲的等值电压为正弦波形，所获得的输出平滑且低次谐波少。按一定的规则对各脉冲的宽度进行调制，即可改变逆变电路输出电压的大小，也可改变输出频率。

![pwm](https://i.loli.net/2020/10/27/Qjw3lGXpCn4Iadq.png)

 2. gamma（屏幕灰度），灰度使用黑色调表示物体。 每个灰度对象都具有从 0%(白色)到100%(黑色)的亮度值。 使用黑白或灰度扫描仪生成的图像通常以灰度显示。

![gamma](https://i.loli.net/2020/10/27/UYRAVqr2axsEw6g.png)

![人眼感知与gamma值的变化](https://i.loli.net/2020/10/27/sdK96MOrVJcLybF.png)

 3. 屏幕亮度设定（0-255)，这个设定值相当于是与屏幕亮度之间的一个关系，一般当项目流程交给软件工程师来调整曲线时，这个对应关系就是确定的了。当然不同的供应商会对应有不同的对应关系，这个关系也不一定是纯线性的，一般来说会呈现如下图这样的趋势。

 4. Android背光调整架构
下图是MTK平台的背光调整架构图从下至上对应硬件到软件层。

![Android MTK平台背光调整架构](https://i.loli.net/2020/10/27/5xFEd8mrOYjvtkb.png)
### 1.2 手动背光

此功能在`settings--->display--->brightness`下面，可知有自动调节和手动调节背光亮度的功能，其中手动是通过进度条(slider)来调节的，此应用对应的布局文件为`\packages\apps\Settings\res\layout\preference_dialog_brightness.xml `如果项目有overlay请自行找下对应复写布局

在`\frameworks\base\core\res\res\values\config.xml`下定义了手动背光亮度的最小值
```
    <!-- Minimumscreen brightness allowed by the power manager. -->  
    <integernameintegername="config_screenBrightnessDim">20</integer>
```
在`\frameworks\base\core\java\android\os\Power.java`中定义：

```
    /** 
     * Brightness value for fully off 
     */  
    public static final int BRIGHTNESS_OFF = 0;  

    /** 
     * Brightness value for dim backlight 
     */  
    public static final int BRIGHTNESS_DIM =20;  

    /** 
     * Brightness value for fully on 
     */  
    public static final int BRIGHTNESS_ON =255;  

    /** 
     * Brightness value to use when battery islow 
     */  
    public staticfinal int BRIGHTNESS_LOW_BATTERY = 10;  
```
在`\frameworks\base\packages\SettingsProvider\res\values\defaults.xml`中定义了默认值
```
    <!-- Default screen brightness, from 0 to 255.  102 is 40%. -->
    <integernameintegername="def_screen_brightness">102</integer>  
    <boolnameboolname="def_screen_brightness_automatic_mode">false</bool>  
```
由这里的代码我们可以看到亮度调节范围是从20-255.这里我们需要注意的时屏幕的最小亮度不能设置为0，0代表的含义是息屏，也就是屏幕完全不亮，允许用户调节的时候一旦设置成了0，就会导致屏幕直接黑了且无法再触控调整。

刚才我们找到了布局，所以我们看下`/frameworks/base/packages/SystemUI/src/com/android/systemui/settings/BrightnessController.java`中覆写的进度条onchanged方法：
```
    @Override
    public void onChanged(ToggleSlider toggleSlider, boolean tracking, boolean automatic, int value, boolean stopTracking) {
        updateIcon(mAutomatic);
        if (mExternalChange) return;

        if (mSliderAnimator != null) {
            mSliderAnimator.cancel();
        }

        final int min;
        final int max;
        final int metric;
        final String setting;

        if (mIsVrModeEnabled) {
            metric = MetricsEvent.ACTION_BRIGHTNESS_FOR_VR;
            min = mMinimumBacklightForVr;
            max = mMaximumBacklightForVr;
            setting = Settings.System.SCREEN_BRIGHTNESS_FOR_VR;
        } else {
            metric = mAutomatic
                    ? MetricsEvent.ACTION_BRIGHTNESS_AUTO
                    : MetricsEvent.ACTION_BRIGHTNESS;
            min = mMinimumBacklight;
            max = mMaximumBacklight;
            setting = Settings.System.SCREEN_BRIGHTNESS;
        }

        final int val = convertGammaToLinear(value, min, max);

        if (stopTracking) {
            MetricsLogger.action(mContext, metric, val);
        }

        setBrightness(val);
        if (!tracking) {
            AsyncTask.execute(new Runnable() {
                    public void run() {
                        Settings.System.putIntForUser(mContext.getContentResolver(),
                                setting, val, UserHandle.USER_CURRENT);
                    }
                });
        }

        for (BrightnessStateChangeCallback cb : mChangeCallbacks) {
            cb.onBrightnessLevelChanged();
        }
    }
```
主要的就是这个setBrightness(val);的方法，它最后是在PowerManager中去设置了settings.db数据库中的值，原理和`Settings.System.putIntForUser(mContext.getContentResolver(),setting, val, UserHandle.USER_CURRENT);`这个代码基本时一致，当然这个代码生效需要系统签名。
这里给出一个与可以adb shell下修改这个背光亮度值的一个方法：
获取当前背光亮度值（0-255）`cat sys/class/leds/lcd-backlight/brightness`
设置背光亮度（0-255）`echo 102 > sys/class/leds/lcd-backlight/brightness`
当然这里的方法可以看到在BrightnessController.java类中定义了一个内部类BrightnessObserver，这个类的功能时时监听settings.db数据库中背光亮度的变化，发生变化后会再通过hanlder去更新进度条，将自定义的ToggleSlider设置进度条的总长和当前进度位置。因此基于这个机制我们可以认为源码的进度条调节机制是线性曲线。
### 1.3 自动背光亮度

这里有一篇博客[Android7.1 亮度自动调节][1]写的不错的，可以直接看下，我下就拿其中重点的提下就行了。
[1]: https://blog.csdn.net/kitty_landon/article/details/64128140
```
    private void updateAutoBrightness(boolean sendUpdate) {
        if (!mAmbientLuxValid) {
            return;
        }

        float value = mScreenAutoBrightnessSpline.interpolate(mAmbientLux);//从mScreenAutoBrightnessSpline中获取到当前环境光照mAmbientLux对应的屏幕亮度值与255的比值value。
        float gamma = 1.0f;

        if (USE_SCREEN_AUTO_BRIGHTNESS_ADJUSTMENT
                && mScreenAutoBrightnessAdjustment != 0.0f) {
            final float adjGamma = MathUtils.pow(mScreenAutoBrightnessAdjustmentMaxGamma,
                    Math.min(1.0f, Math.max(-1.0f, -mScreenAutoBrightnessAdjustment)));//pow(x,y)求x的y次方。
            gamma *= adjGamma;//计算gamma值，
            if (DEBUG) {
                Slog.d(TAG, "updateAutoBrightness: adjGamma=" + adjGamma);
            }
        }

        if (mUseTwilight) {
            TwilightState state = mTwilight.getLastTwilightState();
            if (state != null && state.isNight()) {
                final long duration = state.sunriseTimeMillis() - state.sunsetTimeMillis();
                final long progress = System.currentTimeMillis() - state.sunsetTimeMillis();
                final float amount = (float) Math.pow(2.0 * progress / duration - 1.0, 2.0);
                gamma *= 1 + amount * TWILIGHT_ADJUSTMENT_MAX_GAMMA;
                if (DEBUG) {
                    Slog.d(TAG, "updateAutoBrightness: twilight amount=" + amount);
                }
            }
        }

        if (gamma != 1.0f) {
            final float in = value;
            value = MathUtils.pow(value, gamma);
            if (DEBUG) {
                Slog.d(TAG, "updateAutoBrightness: gamma=" + gamma
                        + ", in=" + in + ", out=" + value);
            }
        }

        int newScreenAutoBrightness =
                clampScreenBrightness(Math.round(value * PowerManager.BRIGHTNESS_ON));//round()四舍五入，计算当前环境光照实际亮度值
        if (mScreenAutoBrightness != newScreenAutoBrightness) {
            if (DEBUG) {
                Slog.d(TAG, "updateAutoBrightness: mScreenAutoBrightness="
                        + mScreenAutoBrightness + ", newScreenAutoBrightness="
                        + newScreenAutoBrightness);
            }

            mScreenAutoBrightness = newScreenAutoBrightness;
            mLastScreenAutoBrightnessGamma = gamma;
            if (sendUpdate) {
                mCallbacks.updateBrightness();//回调到DisplayPowerController中更新亮度。
            }
        }
    }
```
这是的自动背光的主要算法，这个时候可能对`mScreenAutoBrightnessSpline.interpolate(mAmbientLux)`这个方法觉得很没有头脑，我这里就不po源码了，太多了，还是一个算法，感兴趣朋友可以去看下，我本人不喜欢算法就不多研究了，大致就是从两个端点上取出一个直线来获取对应输入端的输出。我们现不做展开，后面再说这个。

通过上面代码的可以发现，与最终背光亮度有关的其实有两个变量，一个是从spline中算出的值，一个是gamma值。那我们来剖析下。
#### 1.3.1从spline中算出的值

`mScreenAutoBrightnessSpline.interpolate(mAmbientLux)`这个方法是干什么呢？首先它的入参是环境光变量，是直接从环境光传感器中读出来的数据，单位时lux或者nit。值的范围一般大约是0-5000。一般我们正常的工作写字等这种书写照明环境一般在350左右。那么这个spline是啥？spline相当于是个数组，创建的时候就用配置写好的曲线值来匹配。看下它的创建方法
```
    private static Spline createAutoBrightnessSpline(int[] lux, int[] brightness) {
        if (lux == null || lux.length == 0 || brightness == null || brightness.length == 0) {
            Slog.e(TAG, "Could not create auto-brightness spline.");
            return null;
        }
        try {
            final int n = brightness.length;
            float[] x = new float[n];
            float[] y = new float[n];
            y[0] = normalizeAbsoluteBrightness(brightness[0]);
            for (int i = 1; i < n; i++) {
                x[i] = lux[i - 1];
                y[i] = normalizeAbsoluteBrightness(brightness[i]);
            }

            Spline spline = Spline.createSpline(x, y);
            if (DEBUG) {
                Slog.d(TAG, "Auto-brightness spline: " + spline);
                for (float v = 1f; v < lux[lux.length - 1] * 1.25f; v *= 1.25f) {
                    Slog.d(TAG, String.format("  %7.1f: %7.1f", v, spline.interpolate(v)));
                }
            }
            return spline;
        } catch (IllegalArgumentException ex) {
            Slog.e(TAG, "Could not create auto-brightness spline.", ex);
            return null;
        }
    }
```
可以看到这个就是这个spline的创建过程，但是我们更关心两个入参，入参决定了这个环境光0-5000和背光亮度0-255之间的对应关系，这个方法是被以下方法所调用的：
```
    private void createAutoBrightnessSpline() {
        screen_auto_brightness_adj = Settings.System.getFloat(mContext.getContentResolver(), Settings.System.SCREEN_AUTO_BRIGHTNESS_ADJ, -1);
        float temp = Math.round(screen_auto_brightness_adj);
        int[] lux;
        int[] screenBrightness;
        lux = resources.getIntArray(
                com.android.internal.R.array.config_autoBrightnessLevels);
        screenBrightness = resources.getIntArray(
                com.android.internal.R.array.config_autoBrightnessLcdBacklightValues);
        mScreenAutoBrightnessSpline = createAutoBrightnessSpline(lux, screenBrightness);
    }
```
通过上面的方法其实就可以指导，是从xml配置文件中拿到这个对应关系的。那我们看下这个xml：
```xml
    <integer-array name="config_autoBrightnessLevels">
        <item>50</item>
        <item>300</item>
        <item>400</item>
        <item>600</item>
        <item>800</item>
        <item>1000</item>
        <item>1300</item>
        <item>1600</item>
        <item>2000</item>
        <item>3000</item>
        <item>4000</item>
    </integer-array>
    <integer-array name="config_autoBrightnessLcdBacklightValues">
        <item>20</item>   <!-- 0-50 -->
        <item>380</item>  <!-- 50-300 -->
        <item>400</item>  <!-- 300-400 -->
        <item>475</item>  <!-- 400-600 -->
        <item>580</item>  <!-- 600-800 -->
        <item>650</item>  <!-- 800-1000 -->
        <item>750</item>  <!-- 1000-1300 -->
        <item>820</item>  <!-- 1300-1600 -->
        <item>1100</item> <!-- 1600-2000 -->
        <item>1450</item> <!-- 2000-3000 -->
        <item>1700</item> <!-- 3000-4000 -->
        <item>2047</item> <!-- 4000+ -->
    </integer-array>
```
通过这一步步的代码追踪，现在就可以明确的知道mScreenAutoBrightnessSpline.interpolate(mAmbientLux)是从这个xml中获取了对应关系，然后再通过生成算法，拼接除了一个曲线，然后在通过算法，获取出的一个特定的值。
#### 1.3.2.gamma

这个gamma的调整算法在Android 8.0后去掉了
这个gamma值我们看到是通过`MathUtils.pow(mScreenAutoBrightnessAdjustmentMaxGamma,Math.min(1.0f, Math.max(-1.0f,-mScreenAutoBrightnessAdjustment)))`这种指数运算得来的，而自动亮度开关打开时，进度条的调整值时-1到1，进度条的一半50%的位置mScreenAutoBrightnessAdjustment这个值是0.而最终的gamma值是受到这个adjGamma的影响gamma *= adjGamma;//计算gamma值而adjGamma又是通过这个mScreenAutoBrightnessAdjustmentMaxGamma的指数运算得来的，mScreenAutoBrightnessAdjustmentMaxGamma的值是在`frameworks/base/core/res/res/values/config.xml`中定义：
```
<!-- The maximum range of gamma adjustment possible using the screen
         auto-brightness adjustment setting. -->
    <fraction name="config_autoBrightnessAdjustmentMaxGamma">300%</fraction>
```
我们看到原生是3意味着变化是在1/3次方到3次方之间，这个趋势是符合人眼在低光线亮度时对光线变化相对比较敏感的处理。
这里gamma值的调整变化时完全根据进度度条来的，同时通过指数运算放大，然后最后作用于亮度上。
## 2如何调整背光曲线

这里同样我们也是按照手动和自动两个部分来说。

![dev.png](https://i.loli.net/2020/10/27/DTgSO5mzafjdGpF.png)
### 2.1手动背光调节思路

上面我们调查源码这部分的时候看到，源码是有个回调机制，因此导致了进度条调节的背光曲线为线性变化，那我们现在需要让这个背光曲线变成指数型曲线或者其他任何自定义的曲线怎么处理呢？
### 2.2算法拟合曲线

其实很简单，就是把最终设置到settings.db数据库中的值给定死，那么屏幕亮度就会按照我们的想法变化。有了这个目标我们就可以找到设置亮度的地方前增加一个算法就可以了，用这个算法来调整原先线性的曲线变为我们想要的曲线。这个算法的公式可以通过客户给出的具体要求来做曲线拟合。

至于如何才能拟合曲线，这里给个方法，当然数学好的同学肯定自有高招，我这里只是给个还算比较方便的方法，就是用excel，先在表格中用给定的值记录进去做出表格，然后先做出目标曲线（这里要注意的是，图标做的时候用的不是折线图，而是应该选带平滑线的散点图），然后在图表中添加趋势线，在编辑中把几种类型都选下看看那种更接近目标。
### 2.3修复进度条问题

但是算法有了就万事大吉了吗？并不是！源码的时候发现改写后的背光亮度设置进去后，会出发回调重新设置进度条的最大值和当前进度，这个事情就比较麻烦了。首先我们会发现我们增加的算法会影响进度条的调整，在我尝试改动的时候就发现以下的问题:

 >   1.在手动设置了亮度后发现进度标识自动跳走的
 >   2.进度条百分比显示不再线性
 >   3.无法自由滑动进度条
 >   4.进度条只能在特定点停下，设置到别的点会自动跳到附近的点
 >   5.进度条百分比不再完整

这些问题的出现弄得我一度崩溃，但是还是要慢慢来，首先我们知道了时因为回调机制的原因导致进度条怎么都不能符合我们的要求，那么我们先把它去掉，不再回调。这个时候发现,诶？进度条似乎不再能到255了，只有0-100了，而之前的百分比似乎也不能调到96%以后，就是因为回调设置进去的进度条的值是配置文件中亮度最大值和亮度最小值的差值，并且显示的百分比又是在这个范围内计算的比例。知道了这个我们只要预先直接给进度条设置好总长值，然后回调的时候让其根据我们增加的曲线算法反向计算一次得出目前的屏幕亮度对应0-255的值时多少就好了。这里可能有点绕，我强调下，**我们不仅需要能通过进度条得到对应设置0-255的亮度值，还要能根据这个亮度值反向计算得到对应于进度条上0-255的比例值，然后设置到进度条上，这样才能正确显示进度条。**回调会从settings数据库中得到当前设置的背光亮度值，然后反算完后就得到对应目前进度条上的亮度，再回设置就
这样处理后时的确可以解决进度条显示的问题，但是百分比的显示是在settings里面的，并且不止一个地方有监听调用，这样的话真的要把这个东西改成线性就太麻烦了，观察市面上的大部分手机后，**建议直接去除亮度百分比的显示**，其实只要目标曲线不是线性的，那么我们的百分比也就不会是线性的。

![原手动背光调节.png](https://i.loli.net/2020/10/27/Uu1iZRAKCg5xmJL.png)
### 2.4其他问题

除了刚才说的进度条问题我在调试的过程中还发现了几个问题。

 >   1.推出设置中的显示界面，进度条位置就不记录了。
 >>   这个主要是回调那部分没有改好。

 >   2.恢复出厂后手动进度的位置不在原来的50%的位置。
 >>   这个问题主要是原先时读取的默认102在<integernameintegername="def_screen_brightness">102</integer>但是这个值对应的现在曲线的百分比就不是在50%而是应该按照新算法计算，所以需要改成自己的值。

### 3.自动背光调节思路

源码提供了frameworks/base/core/res/res/values/config.xml这个配置中直接更改节点值的方式来修改自动背光曲线，这种方式我们这里不做过多的讨论了。下面来说说自定义的背光曲线怎么弄。

![原自动背光调节1.png](https://i.loli.net/2020/10/27/f3qMLmUe54u72pA.png)
### 3.1 测定0-255亮度值对应屏幕nit亮度

这个如标题，至于为啥要这么做，主要是在屏幕供应商确定，驱动确定的基础上，这个0-255的亮度值与屏幕nit亮度之间的关系就基本时确定的了。测量出了这个关系，对后面应用亮度时可以提高不少效率。
### 3.2 通过算法来匹配自动背光曲线

我们在匹配的时候可能会遇到需要适配各种不同的情况，比如限定了进度条各位置的背光曲线，这样的话，等于我们有两个输入变量，第一个时环境光，第二个时进度条的位置带来的亮度调整。
首先，我们通过算法满足环境光变化时，背光亮度的的曲线。这里要重点说的是，如果定制曲线不仅仅只规定了一条，而是连调节变化范围的区间也规定了的话，应该考虑直接替换原码的gamma的计算，毕竟源码是通过定了一条背光曲线后，通过255阶的gamma来调整基于设定配置好的背光曲线的进度条最小或最高的调整。大约是从1/3到3次方的一个变化，因此如果需要定制则建议直接替换自己的定制算法，并且在Android8.0以后已经去掉gamma值的仅算调整这个算法，这里就提一句，主要是替换算法的时候可不要有什么心里负担。
### 3.3 自动背光曲线调整进度条不平滑

在这里要重点讲下这个问题，由于我们考虑的时客户给定了变化范围的定制曲线，我们利用算法框住这个范围的时候会发现，原生算法中进度条的变化范围时-1到1的。我们自己处理这个曲线的时候是很难用出面积来表示的，当然数学好的有办法的除外。我这里就说下，把-1到1这个范围取整后利用曲线取出的数值后怎么办？
当我们完成曲线后，会发现由于取整，比如我们有三条曲线，我们会在进度条左右1/3的点时发生亮度突变（当然一边左1/3点亮度相对暗，突变看起来特别明显），这个突变时由于我们的计算结果因取整从相邻的曲线跃迁到目标曲线的一个结果。我们可以通过在增加一个调整算法来将类似的变化调整为线性的。
增加的算法直接划出的环境光区间内的计算最低值和最高值的线性条件（这个是斜线）也可以计算区间内环境光最低和最好的两条线性，再以线性条件的平均值来作为区间变化的线性条件，并在总环境光区间内添加多条这样的线性约束后，整体的变化就会类似与面积处理的结果，肉眼就难以看出了。同时我们指导环境光的变化一般时固定的，也不会就算有变化，也没有拖拉进度条这么快，因此每次经过这两次的计算，得到的屏幕亮度基本就可以符合定制要求了。
## 4.总结

通过上面的描述和算法对屏幕亮度的调整等措施，我们确实可以在系统的设置中做到对背光曲线的调整，但从最终角度讲不应该是这么来处理的。应该是由部件和厂家来测量出需要的值，让软件端将调整好的曲线写入配置文件中才是一个比较妥善的方式。
