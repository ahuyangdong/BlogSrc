---
title: Android实现二维码扫描功能（一）-ZXing插件接入
date: 2017-07-30 20:34:08
id: 203408
tags: [Zxing,二维码]
categories: Android
---
简介
--

关于Android扫描二维码的功能实现，网上有很多相关资料。在对比之后，选用了前辈了修改过的ZXing直接接入到项目中，特制作此demo，介绍整个过程。

效果预览
----

先上图展示效果（模拟器没有摄像头，录出来效果不好，将就看）
![这里写图片描述](https://img-blog.csdn.net/20170730200637098?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

集成步骤
----
1、拷贝本项目demo中的com.google.zxing5个包引入到自己的项目中。
![这里写图片描述](https://img-blog.csdn.net/20170730201552696?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
2、拷贝本项目demo中的布局activity_scanner.xml和toolbar_scanner.xml
![这里写图片描述](https://img-blog.csdn.net/20170730201721511?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
3、拷贝资源目录raw至本项目中，beep.ogg是扫描成功时的提示音。
![这里写图片描述](https://img-blog.csdn.net/20170730201816363?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
4、拷贝或合并文件内容attrs.xml/colors.xml/ids.xml三个文件。
![这里写图片描述](https://img-blog.csdn.net/20170730201904011?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
5、build.gradle文件中添加引用

``` java
compile 'com.google.zxing:core:3.3.0'
```
6、修改R文件引用路径
修改以下4个文件中的R文件引用地址，引用本项目的R。
``` java
com.google.zxing.activity.CaptureActivity
com.google.zxing.decoding.CaptureActivityHandler
com.google.zxing.decoding.DecodeHandler
com.google.zxing.view.ViewfinderView
```
集成部分到此结束，下面看一下如何实现功能。

权限配置
----
AndroidManifest.xml中添加权限申请代码：

``` xml
<uses-permission android:name="android.permission.INTERNET" /> <!-- 网络权限 -->
<uses-permission android:name="android.permission.VIBRATE" /> <!-- 震动权限 -->
<uses-permission android:name="android.permission.CAMERA" /> <!-- 摄像头权限 -->
<uses-feature android:name="android.hardware.camera.autofocus" /> <!-- 自动聚焦权限 -->
```
Android6.0之后Camera需要加入动态权限申请代码，在下面的实现部分会给出。

功能实现
----
完成上述集成之后，通过调用CaptureActivity就可以实现扫码功能。
MainActivity源码部分：

``` java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    Button btnQrCode; // 扫码
    TextView tvResult; // 结果

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        btnQrCode = (Button) findViewById(R.id.btn_qrcode);
        btnQrCode.setOnClickListener(this);

        tvResult = (TextView) findViewById(R.id.txt_result);
    }

    // 开始扫码
    private void startQrCode() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            // 申请权限
            ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.CAMERA}, Constant.REQ_PERM_CAMERA);
            return;
        }
        // 二维码扫码
        Intent intent = new Intent(MainActivity.this, CaptureActivity.class);
        startActivityForResult(intent, Constant.REQ_QR_CODE);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btn_qrcode:
                startQrCode();
                break;
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //扫描结果回调
        if (requestCode == Constant.REQ_QR_CODE && resultCode == RESULT_OK) {
            Bundle bundle = data.getExtras();
            String scanResult = bundle.getString(Constant.INTENT_EXTRA_KEY_QR_SCAN);
            //将扫描出的信息显示出来
            tvResult.setText(scanResult);
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case Constant.REQ_PERM_CAMERA:
                // 摄像头权限申请
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // 获得授权
                    startQrCode();
                } else {
                    // 被禁止授权
                    Toast.makeText(MainActivity.this, "请至权限中心打开本应用的相机访问权限", Toast.LENGTH_LONG).show();
                }
                break;
        }
    }


}
```

在发起CaptureActivity前，需要申请动态权限

``` java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
    // 申请权限
    ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.CAMERA}, Constant.REQ_PERM_CAMERA);
    return;
}
```
申请结果处理在onRequestPermissionsResult方法中，可参考注释。

``` java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    switch (requestCode) {
        case Constant.REQ_PERM_CAMERA:
            // 摄像头权限申请
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // 获得授权
                startQrCode();
            } else {
                // 被禁止授权
                Toast.makeText(MainActivity.this, "请至权限中心打开本应用的相机访问权限", Toast.LENGTH_LONG).show();
            }
            break;
    }
}
```
## 获得扫描结果 ##
zxing的CaptureActivity已经完美的实现了扫码和结果回传，我们只需要在调用方处理返回结果就可以了。

``` java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    //扫描结果回调
    if (requestCode == Constant.REQ_QR_CODE && resultCode == RESULT_OK) {
        Bundle bundle = data.getExtras();
        String scanResult = bundle.getString(Constant.INTENT_EXTRA_KEY_QR_SCAN);
        //将扫描出的信息显示出来
        tvResult.setText(scanResult);
    }
}
```
这里使用一个TextView呈现扫码结果。

结语
--
到这里已经完成了Android扫码功能，并可以通过调用摄像头实时扫码并处理结果，可以在项目中直接使用。感谢相关开源作者的贡献，让我们集成扫码功能如此简单。

参考资料

 - http://www.jianshu.com/p/e80a85b17920

源码下载
-----

GitHub项目地址（终版）：
https://github.com/ahuyangdong/QrCodeDemo4