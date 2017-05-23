Tomcat设置shutdown.sh

### 设置catalina.sh

1. 路径:`{tomcat}/bin/catalina.sh`
2. 找到 PRGDIR=`dirname "$PRG"`,下面添加代码

```
PRGDIR=`dirname "$PRG"`
# 下面添加
if [ -z "$CATALINA_PID" ]; then
      CATALINA_PID=$PRGDIR/CATALINA_PID
      cat $CATALINA_PID
fi
```


### 设置shutdown.sh
1. 路径:`{tomcat}/bin/设置shutdown.sh`
2. 找到 `exec "$PRGDIR"/"$EXECUTABLE" stop "$@"`(我这里是最后一行)
3. 在 stop 前加 `-force`

代码如下

```
exec "$PRGDIR"/"$EXECUTABLE" stop -force "$@"
```
