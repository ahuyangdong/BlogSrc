---
title: Android实现二维码扫描功能（四）-ZXing识别图片二维码，相册选图
date: 2017-08-22 20:53:49
id: 205349
tags: [Zxing,二维码,相册二维码]
categories: Android
---
## 简介 ##
上一篇[ Android实现二维码扫描功能（三）-闪光灯控制](http://www.ahuyangdong.top/2017/08/06/201754)介绍了光线较弱情况下开启闪光灯来辅助二维码识别的方法。

本篇我们介绍如何识别相册中的图片（含二维码）

## 效果 ##
因为模拟器文件路径有问题（也可能是我没琢磨对），就没有录制gif了，这里放几张过程图。
![主界面](https://img-blog.csdn.net/20170822202347413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![进入扫码页](https://img-blog.csdn.net/20170822202416505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![选择相册中的图片](https://img-blog.csdn.net/20170822202525773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![识别图片为二维码结果](https://img-blog.csdn.net/20170822202544719?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 实现步骤 ##
1、com.google.zxing.activity.CaptureActivity中实现点击“相册”功能。

``` java
private View.OnClickListener albumOnClick = new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        //打开手机中的相册
        Intent innerIntent = new Intent(Intent.ACTION_GET_CONTENT);
        innerIntent.setType("image/*");
        startActivityForResult(innerIntent, REQUEST_CODE_SCAN_GALLERY);
    }
};
```
2、覆写com.google.zxing.activity.CaptureActivity#onActivityResult方法，处理返回图数据，并识别二维码结果。

``` java
@Override
protected void onActivityResult(final int requestCode, int resultCode, Intent data) {
    if (resultCode==RESULT_OK) {
        switch (requestCode) {
            case REQUEST_CODE_SCAN_GALLERY:
                handleAlbumPic(data);
                break;
        }
    }
    super.onActivityResult(requestCode, resultCode, data);
}

/**
 * 处理选择的图片
 * @param data
 */
private void handleAlbumPic(Intent data) {
    //获取选中图片的路径
    photo_path = UriUtil.getRealPathFromUri(CaptureActivity.this, data.getData());

    mProgress = new ProgressDialog(CaptureActivity.this);
    mProgress.setMessage("正在扫描...");
    mProgress.setCancelable(false);
    mProgress.show();

    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            mProgress.dismiss();
            Result result = scanningImage(photo_path);
            if (result != null) {
                Intent resultIntent = new Intent();
                Bundle bundle = new Bundle();
                bundle.putString(Constant.INTENT_EXTRA_KEY_QR_SCAN ,result.getText());

                resultIntent.putExtras(bundle);
                CaptureActivity.this.setResult(RESULT_OK, resultIntent);
                finish();
            } else {
                Toast.makeText(CaptureActivity.this, "识别失败", Toast.LENGTH_SHORT).show();
            }
        }
    });
}
```
其中，scanningImage方法实现读取path对应的图片，并交由QRCodeReader类来解析图片内容，中间涉及到图片的压缩处理，见：

``` java
/**
 * 扫描二维码图片的方法
 * @param path
 * @return
 */
public Result scanningImage(String path) {
    if(TextUtils.isEmpty(path)){
        return null;
    }
    Hashtable<DecodeHintType, String> hints = new Hashtable<>();
    hints.put(DecodeHintType.CHARACTER_SET, "UTF8"); //设置二维码内容的编码

    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true; // 先获取原大小
    scanBitmap = BitmapFactory.decodeFile(path, options);
    options.inJustDecodeBounds = false; // 获取新的大小
    int sampleSize = (int) (options.outHeight / (float) 200);
    if (sampleSize <= 0)
        sampleSize = 1;
    options.inSampleSize = sampleSize;
    scanBitmap = BitmapFactory.decodeFile(path, options);
    RGBLuminanceSource source = new RGBLuminanceSource(scanBitmap);
    BinaryBitmap bitmap1 = new BinaryBitmap(new HybridBinarizer(source));
    QRCodeReader reader = new QRCodeReader();
    try {
        return reader.decode(bitmap1, hints);
    } catch (NotFoundException e) {
        e.printStackTrace();
    } catch (ChecksumException e) {
        e.printStackTrace();
    } catch (FormatException e) {
        e.printStackTrace();
    }
    return null;
}
```
3、CaptureActivity成功关闭之后，在MainActivity中捕获Result。即覆写com.dommy.qrcode.MainActivity#onActivityResult方法。

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
到这里，相册图片二维码数据的识别就结束了。由于ZXing提供了相应的识别接口，因此我们在做的时候就比较简单。概括为：

 1. 实现相册选择功能。
 2. 实现相册文件路径的转换（Uri转String path）
 3. 实现图片的缩放处理（也许是防止识别时候溢出吧，具体没去分析了）
 4. 识别结果，并返回给相应的界面。
 

工具类
---
在上节代码中说到了Uri转String path的方法，使用到的UriUtil.getRealPathFromUri是踩了很多坑之后找到的代码，因为android系统版本不同，Uri的表示方法也不一样，这样的话就需要这样的工具类了。

``` java
package com.dommy.qrcode.util;

import android.annotation.TargetApi;
import android.content.Context;
import android.content.CursorLoader;
import android.database.Cursor;
import android.net.Uri;
import android.os.Build;
import android.provider.DocumentsContract;
import android.provider.MediaStore;

/**
 * Uri路径工具
 */

public class UriUtil {
    /**
     * 根据图片的Uri获取图片的绝对路径(适配多种API)
     *
     * @return 如果Uri对应的图片存在, 那么返回该图片的绝对路径, 否则返回null
     */
    public static String getRealPathFromUri(Context context, Uri uri) {
        int sdkVersion = Build.VERSION.SDK_INT;
        if (sdkVersion < 11) return getRealPathFromUri_BelowApi11(context, uri);
        if (sdkVersion < 19) return getRealPathFromUri_Api11To18(context, uri);
        else return getRealPathFromUri_AboveApi19(context, uri);
    }

    /**
     * 适配api19以上,根据uri获取图片的绝对路径
     */
    @TargetApi(Build.VERSION_CODES.KITKAT)
    private static String getRealPathFromUri_AboveApi19(Context context, Uri uri) {
        String filePath = null;
        String wholeID = DocumentsContract.getDocumentId(uri);

        // 使用':'分割
        String[] ids = wholeID.split(":");
        String id = null;
        if (ids == null) {
            return null;
        }
        if (ids.length > 1) {
            id = ids[1];
        } else {
            id = ids[0];
        }

        String[] projection = {MediaStore.Images.Media.DATA};
        String selection = MediaStore.Images.Media._ID + "=?";
        String[] selectionArgs = {id};

        Cursor cursor = context.getContentResolver().query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,//
                projection, selection, selectionArgs, null);
        int columnIndex = cursor.getColumnIndex(projection[0]);
        if (cursor.moveToFirst()) filePath = cursor.getString(columnIndex);
        cursor.close();
        return filePath;
    }

    /**
     * 适配api11-api18,根据uri获取图片的绝对路径
     */
    private static String getRealPathFromUri_Api11To18(Context context, Uri uri) {
        String filePath = null;
        String[] projection = {MediaStore.Images.Media.DATA};
        CursorLoader loader = new CursorLoader(context, uri, projection, null, null, null);
        Cursor cursor = loader.loadInBackground();

        if (cursor != null) {
            cursor.moveToFirst();
            filePath = cursor.getString(cursor.getColumnIndex(projection[0]));
            cursor.close();
        }
        return filePath;
    }

    /**
     * 适配api11以下(不包括api11),根据uri获取图片的绝对路径
     */
    private static String getRealPathFromUri_BelowApi11(Context context, Uri uri) {
        String filePath = null;
        String[] projection = {MediaStore.Images.Media.DATA};
        Cursor cursor = context.getContentResolver().query(uri, projection, null, null, null);
        if (cursor != null) {
            cursor.moveToFirst();
            filePath = cursor.getString(cursor.getColumnIndex(projection[0]));
            cursor.close();
        }
        return filePath;
    }
}
```
这个主要是参照这个博客来的：
http://www.cnblogs.com/baiqiantao/p/6795684.html

结语
--

到这里已经完成了图片二维码的识别，同时还发现了Uri表示方法的坑，留下了脚印。

结合前面3篇文章，Android二维码扫描功能已基本完成与完善，告一段落。
 

源码下载
----
GitHub项目地址（终版）：
https://github.com/ahuyangdong/QrCodeDemo4