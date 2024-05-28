Tool to decompile and compile an apk among other stuff...
Installation:
```bash
apt-get install apktool
```
## Decompile the application
```bash
apktool d APKey.apk -o out
```
## Compile an application
Works after modifying something on the app
```bash
apktool d out -o mod.apk
```
#Note We need to [[Sign an Android apk]] in order for it to be installed.

## Standalone binary

[APKTOOL](https://ibotpeaches.github.io/Apktool/) can be installed with apt-get on linux or it can be started with java by downloading the .jar file:
```bash
java -jar apktool_2.5.0.jar b out -o mod.apk
```
Examples:
[[APKey#^0744da]]