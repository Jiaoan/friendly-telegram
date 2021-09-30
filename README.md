
# 如何對APK進行修改後，重新簽名APK
## 1. 使用 apk tool 進行反編譯:
將 apk 放到 apk tool 資料夾，為了方便作業，不用使用絕對路徑。

將 cmd 路徑切換到 apk tool，輸入以下指令:
```
apktool.bat d -o outDir source.apk
```
* outDir :表示反編譯後的資原始檔存放到哪個目錄下
* source.apk ：表示要進行反編譯的apk檔名稱

反編譯後的資源會生成在指定的資料夾內，對想要修正的檔案做修正。

若想修改版號，可以修改 apktool.yml 內的 versionCode 或者 versionName

### 補充說明: apk tool 資料夾結構示意
    apk-tool
    ├─apktool_2.6.0.jar
    ├─apktool.bat
    ├─source.apk
    ├─outDir
    │   ├─assets
    │   ├─ ...
    ─
    
***
## 2. 使用 apk tool 進行重新編譯
> 注意: cmd 路徑一樣是在 apk tool 資料夾底下。

輸入以下指令:
```
apktool.bat b -o new_no_signalign.apk outDir
```
 * new_no_singnalign.apk :新生成的apk檔案，此apk檔案沒有簽名和對齊
 * outDir ：將outDir的檔案進行編譯

編譯完成後，會得到一個沒有簽名的apk: new_no_signalign.apk

***
## 3. 重新簽名(使用apksigner，且利用 v1, v2 做簽名)
此處簡單介紹一下Android用的簽名工具，以及V1(Jar Signature) V2(Full APK Signature)兩種簽名。
1. Android 中對 APK 簽名是通過 jarsigner 或 apksigner 進行的；
2. jarsigner 是 JDK 提供的針對jar包簽名的通用工具位於: JDK/bin/jarsigner.exe；
3. apksigner 是 Google 官方提供的針對 Android apk 簽名及驗證的專用工具，位於: Android SDK/build-tools/SDK版本/apksigner.bat；

### <b>V1簽名</b>:
* 來自JDK(jarsigner), 對zip壓縮包的每個檔案進行驗證, 簽名後還能對壓縮包修改(移動/重新壓縮檔案)
* 對V1簽名的apk/jar解壓,在META-INF存放簽名檔案(MANIFEST.MF, CERT.SF, CERT.RSA),
* 其中MANIFEST.MF檔案儲存所有檔案的SHA1指紋(除了META-INF檔案), 由此可知: V1簽名是對壓縮包中單個檔案簽名驗證

### <b>V2簽名</b>:
* 來自Google(apksigner), 對zip壓縮包的整個檔案驗證, 簽名後不能修改壓縮包(包括zipalign),
* 對V2簽名的apk解壓,沒有發現簽名檔案,重新壓縮後V2簽名就失效, 由此可知: V2簽名是對整個APK簽名驗證


> 注意: apksigner工具預設同時使用V1和V2簽名,以相容Android 7.0以下版本

<br>

### <b>(補充) zipalign 記憶體對齊</b>
> 位於Android SDK/build-tools/SDK版本/zipalign.exe zipalign；是對zip包對齊的工具,使APK包內未壓縮的資料有序排列對齊,從而減少APP執行時記憶體消耗

指令如下:
```
zipalign -v 4 in.apk out.apk //4位元組對齊優化
zipalign -c -v 4 in.apk  //檢查APK是否對齊
```
### 非常重要：
>在此處介紹一下v1和v2簽名以及zipalign記憶體對齊，至於先簽名後對齊，還是先對齊後簽名與你採用的簽名方是有關係，<br>
1：zipalign可以在V1簽名後執行,但zipalign不能在V2簽名後執行,只能在V2簽名之前執行！！！<br>
2：如果不好記你就記住一個原則就是：先對齊後簽名

### <b>開始重新簽名</b>
> 注意: cmd 要先 cd 到 Android SDK/build-tools/SDK版本/目錄下。<br>
例如: <br>
C:\Users\Ian.wang\AppData\Local\Android\Sdk\build-tools\30.0.2>

輸入以下指令:

```sh
apksigner sign 
    --v1-signing-enabled true 
    --v2-signing-enabled true
    --v3-signing-enabled false 
    --v4-signing-enabled false
    --ks C:\GitFolder\jtw-jtw-unity\user.keystore
    --ks-key-alias test
    --out C:\Users\Ian.wang\Desktop\apktool\bb.apk
    C:\Users\Ian.wang\Desktop\apktool\new_no_signalign.apk
```
    apksigner sign --v1-signing-enabled true --v2-signing-enabled true --v3-signing-enabled false --v4-signing-enabled false --ks C:\GitFolder\jtw-jtw-unity\user.keystore --ks-key-alias test --out C:\Users\Ian.wang\Desktop\apktool\bb.apk C:\Users\Ian.wang\Desktop\apktool\new_no_signalign.apk

重新簽名完成之後，可以利用以下指令確認

```sh
apksigner verify -v C:\Users\Ian.wang\Desktop\apktool\bb.apk
```
結果如下:
```
Verifies
Verified using v1 scheme (JAR signing): true
Verified using v2 scheme (APK Signature Scheme v2): true
Verified using v3 scheme (APK Signature Scheme v3): false
Verified using v4 scheme (APK Signature Scheme v4): false
Verified for SourceStamp: false
Number of signers: 1
```

## 資料來源
> * https://tw511.com/a/01/15111.html#1apktooljar_17
> * https://blog.csdn.net/ZFSR05255134/article/details/111880061
> * apk tool: https://ibotpeaches.github.io/Apktool/install/