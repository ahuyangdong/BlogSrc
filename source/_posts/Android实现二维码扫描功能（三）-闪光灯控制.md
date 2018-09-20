---
title: Android实现二维码扫描功能（三）-闪光灯控制
date: 2017-08-06 20:17:54
id: 201754
tags: [Zxing,二维码,闪光灯]
categories: Android
---
简介
--

上一篇[Android实现二维码扫描功能（二）-ZXing个性化与近距离识别优化](http://www.ahuyangdong.top/2017/07/30/314141 "Android实现二维码扫描功能（二）-ZXing个性化与近距离识别优化")介绍了ZXing框架个性化定制和识别优化方法。

本篇我们对光线暗淡情况下闪光灯的使用做出介绍。

## 效果 ##
晚上测试时：

 - 开灯后：
![开灯效果图](https://img-blog.csdn.net/20170806195023964?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 - 未开灯：
![未开灯效果图](https://img-blog.csdn.net/20170806195042186?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## 实现步骤 ##
1、在activity_scanner.xml界面上加上闪光灯开关按钮。可以是Button、Checkbox等控件。

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    ...>

    ...

    <FrameLayout
        ...>

        <SurfaceView
            .../>

        <com.google.zxing.view.ViewfinderView
            ... />

        <ImageButton
            android:id="@+id/btn_flash"
            android:layout_width="40dip"
            android:layout_height="40dip"
            android:padding="6dip"
            android:layout_gravity="bottom|center_horizontal"
            android:layout_marginBottom="30dip"
            android:background="?attr/selectableItemBackground"
            android:scaleType="centerInside"
            android:src="@drawable/flash_off" />
    </FrameLayout>

</LinearLayout>
```
编辑区域预览
![编辑区域预览](https://img-blog.csdn.net/20170806200825590?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、修改com.google.zxing.camera.CameraManager类，添加setFlashLight方法。

``` java
/**
     * 打开或关闭闪光灯
     * @param isOpen 是否开启闪光灯
     * @return boolean 操作成功/失败。
     */
public boolean setFlashLight(boolean isOpen) {
    if (camera == null || !previewing) {
        return false;
    }
    Camera.Parameters parameters = camera.getParameters();
    if (parameters == null) {
        return false;
    }
    List<String> flashModes = parameters.getSupportedFlashModes();
    // 检查手机是否有闪光灯
    if (null == flashModes || 0 == flashModes.size()) {
        // 没有闪光灯则返回
        return false;
    }
    String flashMode = parameters.getFlashMode();
    if (isOpen) {
        if (Camera.Parameters.FLASH_MODE_TORCH.equals(flashMode)) {
            return true;
        }
        // 开启
        if (flashModes.contains(Camera.Parameters.FLASH_MODE_TORCH)) {
            parameters.setFlashMode(Camera.Parameters.FLASH_MODE_TORCH);
            camera.setParameters(parameters);
            return true;
        } else {
            return false;
        }
    } else {
        if (Camera.Parameters.FLASH_MODE_OFF.equals(flashMode)) {
            return true;
        }
        // 关闭
        if (flashModes.contains(Camera.Parameters.FLASH_MODE_OFF)) {
            parameters.setFlashMode(Camera.Parameters.FLASH_MODE_OFF);
            camera.setParameters(parameters);
            return true;
        } else {
            return false;
        }
    }
}
```


3、在com.google.zxing.activity.CaptureActivity类中添加闪光灯开关相关代码。

``` java
btnFlash = (ImageButton) findViewById(R.id.btn_flash);
btnFlash.setOnClickListener(flashListener);

/**
 *  闪光灯开关按钮
 */
private View.OnClickListener flashListener = new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        try {
            boolean isSuccess = CameraManager.get().setFlashLight(!isFlashOn);
            if(!isSuccess){
                Toast.makeText(CaptureActivity.this, "暂时无法开启闪光灯", Toast.LENGTH_SHORT).show();
                return;
            }
            if (isFlashOn) {
                // 关闭闪光灯
                btnFlash.setImageResource(R.drawable.flash_off);
                isFlashOn = false;
            } else {
                // 开启闪光灯
                btnFlash.setImageResource(R.drawable.flash_on);
                isFlashOn = true;
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
};
```
运行即可查看效果。

参考
---

http://iluhcm.com/2016/01/08/scan-qr-code-and-recognize-it-from-picture-fastly-using-zxing/

## 源码下载 ##
GitHub项目地址（终版）：
https://github.com/ahuyangdong/QrCodeDemo4