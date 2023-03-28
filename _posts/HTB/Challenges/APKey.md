# Content

- APK Installation with ADB
- JADX decompilation from .dex to .java
- Source Code Review (Smali decompilation with APKTool) 
- APKTool binary for decompilation/modification/compilation
- APKSigner, signing a compiled binary
- Authentication Bypass

The file is an .apk file so let's start by installing the apk into our genymmotion virtual device:
```bash
adb.exe install -r C:\Users\shuciran\Downloads\APKey.apk
```
The application has only a Login panel, and there is are no default credentials at first glance:
![[Pasted image 20230205031517.png]]
Let's decompile the .apk with JADX-GUI installed in our kali: ^4c9fa3
```bash
jadx-gui
```
We go directly to the com folder and into the MainActivity Class:
![[Pasted image 20230205004833.png]]
Inside this Class we can identify that there is a comparison at the "user" and "password" field form:
![[Pasted image 20230205032006.png]]

JADX only converts the .dex file to .java code, but in order to bypass the authentication mechanism on the device we need to decompile the application first; this can be done by using [[APKTool]]: ^0744da
```bash
apktool d -s APKey.apk -o out
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.6.1-dirty on APKey.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /root/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Copying raw classes.dex file...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```
A good idea is to try and modify the smali code first because smali is an assembly language and its instructions are easier to modify than the java code.
The MainActivity is to be found at $DIRPATH/smali/com/example/apkey/MainActivity\$a.smali and there we can see a very interesting line where all the authentication is relying on:
```bash
:goto_1
    const-string v1, "a2a3d412e92d896134d9c9126d756f"

    .line 2
    invoke-virtual {p1, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result p1

    if-eqz p1, :cond_1

    iget-object p1, p0, Lcom/example/apkey/MainActivity$a;->b:Lcom/example/apkey/MainActivity;

    invoke-virtual {p1}, Landroid/app/Activity;->getApplicationContext()Landroid/content/Context;

```
As we can see, this condition "if-eqz" means that the login is only possible if the comparison between p1 variable which is the hash and our typed password are the same. All we need to do is convert it to a negative conditional in such way that it accepts any password as correct:
```bash
if-nez p1, :cond_1
```
This [smali instructions](http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html) also known as Dalvik Opcodes can be instructed on our smali code in such a way to bypass the login, now that we modify the code, we need to compile our application again, since our kali was having troubles to recompile it:
```bash
apktool b out -o mod.apk   
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.6.1-dirty
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/commons/text/StringEscapeUtils
```
We downloaded the Standalone .jar APKTool binary, this can be executed from bot Linux and Windows:
```cmd
C:\Users\shuciran\Downloads>java -jar apktool_2.5.0.jar b out -o mod.apk
I: Using Apktool 2.5.0
I: Checking whether sources has changed...
I: Checking whether resources has changed...
I: Building apk file...
I: Copying unknown files/dir...
I: Built apk...
```
Once that our apk is compiled, and since a modified application cannot  be installed as usual due to the integrity verification between its certificate and the application per se, we need to sign it, for this we can use the [apksigner.apk](https://apkpure.com/apk-signer/com.haibison.apksigner/download), install it and push (upload) the modified apk: ^d9ee93
```bash
# Installing the apksigner.apk (all in one line)
D:\Programas\Genymotion\tools>adb.exe install -r C:\Users\exter\Downloads\apk-signer_7.0.3_Apkpure.apk
# Pushing the mod.apk
D:\Programas\Genymotion\tools>adb.exe push C:\Users\exter\Downloads\mod.apk /storage/emulated/0/Download
```
#Note Don't forget to uninstall the original application to avoid discrepances between both apps.
Now that both apk files are in the device we can start apksigner inside the device to sign the modified apk. Let's choose the application and sign it:
![[Pasted image 20230205034943.png]]
If by any chance we get the following error, we can ignore it:
![[Pasted image 20230205035043.png]]
And go to the folder where the mod.signed.apk has been stored:
![[Pasted image 20230205035248.png]]
Click on it and select "install", it will be installed with no error:
![[Pasted image 20230205035325.png]]
Finally, if we enter any credential we get the flag:
![[Pasted image 20230205035504.png]]