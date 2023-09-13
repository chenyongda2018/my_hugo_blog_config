
---
title: "记一次在Android中调用百度的OCR接口的经历"
date: 2019-05-10T20:25:04+08:00
draft: false
toc: false
comments: false
categories:
- Android
- OCR
tags:
- Android
- OCR
---

# 前言

最近实验室开了个新项目，是一个通过扫描单词后把扫描过的单词生成游戏来让小朋友记单词的APP，扫描单词这个功能需要用到OCR.  
现在常用的OCR有  
- [Tesseract](https://github.com/tesseract-ocr/tesseract) 这个用的人比较多，而且开源，目前google正在维护，但是我尝试了一下，发现识别准确率不是特别理想。


- 微软的Azure上的[认知服务](https://www.azure.cn/zh-cn/home/features/cognitive-services/computer-vision/) 识别率很高，但是收费，现在有1元体验的套餐，而且不需要验证信用卡，感兴趣的同学可以试试。

- 百度的[文字识别](https://cloud.baidu.com/doc/OCR/OCR-API.html#.E8.AF.B7.E6.B1.82.E8.AF.B4.E6.98.8E) 之所以用这个是因为免费，不过有每天的限制次数，对于学生项目来说够用，还要什么自行车。


下面进入正文

# 如何在Android 中调用百度的OCR进行文字识别
## 1.获取百度文字识别产品服务的 Ak/Sk
1.在[百度AI开放平台](http://ai.baidu.com/tech/ocr)中进入控制台

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c3207893f26~tplv-t2oaga2asx-image.image)


2.找到文字识别 产品服务  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c33733f10e6~tplv-t2oaga2asx-image.image)  

3,创建应用  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c3205d91865~tplv-t2oaga2asx-image.image)  

4,填写信息，**注意这里的包名一定要和项目的包名一致**  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c32077c3cb0~tplv-t2oaga2asx-image.image)  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c3208e59688~tplv-t2oaga2asx-image.image)


5.获得AK/SK  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c320ae4cfd7~tplv-t2oaga2asx-image.image)  

6.下载license文件，在项目中如果直接用AK/SK明文调用百度的OCR，很不安全，可能会被别人反编译之后获得你的AK.SK  
license文件集成了AK/SK 放在项目中可以防止别人破解。
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c32bfbe0ac0~tplv-t2oaga2asx-image.image)

7.下载之后将获得的api.license文件放入main目录下的assets目录中  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c32321a5ee2~tplv-t2oaga2asx-image.image)

## 2.添加百度OCR的SDK到项目中  
1.下载 [百度OCR的android Sdk](https://ai.baidu.com/sdk#ocr)  

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c33207fef3d~tplv-t2oaga2asx-image.image)  
2.下载的SDK压缩包将其解压，并将libs下的ocr-sdk的jar包放入项目的libs目录下    
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c332114fc9f~tplv-t2oaga2asx-image.image)  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c3327d5bc7b~tplv-t2oaga2asx-image.image)

3.在main目录下新建jniLibs目录，并将libs文件夹中的其他文件放入其中  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c33330971ce~tplv-t2oaga2asx-image.image)
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c333ab33813~tplv-t2oaga2asx-image.image)  

4.在app下的build,gradle中添加  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c339602cd1c~tplv-t2oaga2asx-image.image)  
将添加在libs下的sdk JAR包编译  

5.这里下载的压缩包中包括了百度提供的相机扫描时的UI，在拍完照有裁剪框，比较方便，这里我们可以作为module引入项目中
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c33a2e8f5e2~tplv-t2oaga2asx-image.image)  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c339b46cc37~tplv-t2oaga2asx-image.image)

## 3.调用百度OCR  
做完准备工作我们就可以开始调用百度的OCR接口了。   

首先在我们需要进行识别的页面所在的文件中创建 根据License文件初始化OCR实例的函数，并在onCreate()方法中调用 


``` java
/**
     * 自定义license的文件路径和文件名称，以license文件方式初始化
     */
    private void initAccessTokenLicenseFile() {
        OCR.getInstance(mActivity.getApplicationContext()).initAccessToken(new OnResultListener<AccessToken>() {
            @Override
            public void onResult(AccessToken accessToken) {
                String token = accessToken.getAccessToken();
                Log.d(TAG,token);
                hasGotToken = true;
            }

            @Override
            public void onError(OCRError error) {
                error.printStackTrace();
                alertText("自定义文件路径licence方式获取token失败", error.getMessage());
            }
        }, "aip.license", mActivity.getApplicationContext());
    }
```  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c33cd0b0b3a~tplv-t2oaga2asx-image.image)    

定义我们的打开相机事件  
```java
/**
     * 打开相机，进入的相机页面是借用百度OCR 官方DEMO中的相机页面
     * 能够在相机中裁剪图片，和进入图库
     * @author cyd
     */
    private void openCameraForResult() {
        if (!checkTokenStatus()) {
            return;
        }
        Intent intent = new Intent(mActivity, CameraActivity.class);
        intent.putExtra(CameraActivity.KEY_OUTPUT_FILE_PATH,
                FileUtil.getSaveFile(getActivity()).getAbsolutePath());
        intent.putExtra(CameraActivity.KEY_CONTENT_TYPE,
                CameraActivity.CONTENT_TYPE_GENERAL);
        startActivityForResult(intent, REQUEST_CODE_GENERAL_BASIC);
    }
```

这里的CameraActivity用的是引入OCR_UI中的相机活动，自带剪裁框  

接下来需要我们新建一个可以存放OCR的识别方法的类RecognizeService  
```java
**
 * 这个类是用于将拍摄或者图库中获得的图片进行识别，返回JSON格式的字符串。
 */
public class RecognizeService {

	public interface ServiceListener {
        public void onResult(String result);
    }
    
    //高精度版
    public static void recAccurateBasic(Context ctx, String filePath, final ServiceListener listener) {
        GeneralParams param = new GeneralParams();
        param.setDetectDirection(true);
        param.setVertexesLocation(true);
        param.setLanguageType(GeneralBasicParams.ENGLISH);
        param.setRecognizeGranularity(GeneralParams.GRANULARITY_SMALL);
        param.setImageFile(new File(filePath));
        
        //这里的recognizeAccurateBasic方法为百度OCR识别的核心方法
        OCR.getInstance(ctx).recognizeAccurateBasic(param, new OnResultListener<GeneralResult>() {
            @Override
            public void onResult(GeneralResult result) {
                StringBuilder sb = new StringBuilder();
                for (WordSimple wordSimple : result.getWordList()) {
                    WordSimple word = wordSimple;
                    sb.append(word.getWords());
                    sb.append("\n");
                }
                listener.onResult(result.getJsonRes());
            }

            @Override
            public void onError(OCRError error) {
                listener.onResult(error.getMessage());
            }
        });
    }


}

```


在onActivityResult方法中，我们调用刚刚新建的类的recAccurateBasic方法，此方法接收三个参数，分别是context,拍照获取的图片路径，和在RecognizeService类中定义的监听接口，这里的获取图片路径方法，我用的是百度官方DEMO中的方法  

在onResult方法中，返回的result字符串即为识别结果的json字符串，只需要对JSON解析一下就能得到识别结果啦
```java
@Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case REQUEST_CODE_GENERAL_BASIC:
                if (resultCode == Activity.RESULT_OK) {
                    RecognizeService.recAccurateBasic(mActivity, FileUtil.getSaveFile(mActivity.getApplicationContext()).getAbsolutePath(),
                            new RecognizeService.ServiceListener() {
                                @Override
                                public void onResult(String result) {
                                    Bundle bundle = new Bundle();
                                    bundle.putString("wordResultJson",result );
                                    Intent intent = new Intent(mActivity,SelectWordsActivity.class);
                                    intent.putExtra("wordResultBundle",bundle );
                                    startActivity(intent);
                                }
                            });
                }
                break;
            default:
                Log.d(TAG, "onActivityResult: "+"run in default");
                break;
        }
    }

```  


FileUtil类  
```java
public class FileUtil {
    public static File getSaveFile(Context context) {
        File file = new File(context.getFilesDir(), "pic.jpg");
        return file;
    }
}

```




---




(完~)  
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/10/16aa0c99f8e04575~tplv-t2oaga2asx-image.image)





