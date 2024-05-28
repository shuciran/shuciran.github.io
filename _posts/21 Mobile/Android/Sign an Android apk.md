We can download the [apksigner](https://apkpure.com/apk-signer/com.haibison.apksigner/download) and install it in our Android device with [ADB] 
```bash
adb.exe install -r C:\Users\exter\Downloads\apk-signer_7.0.3_Apkpure.apk
```
Upload the apk file that needs to be signed on the device:
```bash
# Pushing the mod.apk
D:\Programas\Genymotion\tools>adb.exe push C:\Users\exter\Downloads\mod.apk /storage/emulated/0/Download
```
Now that both apk files are in the device we can start apksigner inside the device to sign the modified apk. Let's choose the application and sign it:
![[Pasted image 20230205034943.png]]
If by any chance we get the following error, we can ignore it:
![[Pasted image 20230205035043.png]]
And go to the folder where the mod.signed.apk has been stored:
![[Pasted image 20230205035248.png]]
Click on it and select "install", it will be installed with no error.
Examples:
[[APKey#^d9ee93]]