# JMCCC使用教程
JMCCC是由 [@yushijinhun](https://github.com/yushijinhun) 和 [@Darkyoooooo](https://github.com/Darkyoooooo) 开发的一个Java启动器核心，支持：
 * 启动Minecraft
 * 正版验证（以及Yggdrasil的其它API）
 * 下载Minecraft
 * 下载并安装Forge/Liteloader

并且JMCCC是开源的（[MIT许可证](https://to2mbn.github.io/jmccc/LICENSE.txt)），JMCCC在GitHub上的项目：[to2mbn/JMCCC](https://github.com/to2mbn/JMCCC)。

本教程的适用人群：有一定经验的Java开发者


## 1. 环境搭建
JMCCC由三个模块组成，分别为`jmccc`、`jmccc-yggdrasil-authenticator`、`jmccc-mcdownloader`，它们分别提供了启动Minecraft、正版登录、下载Minecraft的功能。您可以按需引用这三个模块。
假如您使用Maven或者Gradle，您只需要加入如下的依赖即可（目前JMCCC最新版本为2.4）：
```
org.to2mbn:jmccc:2.4
org.to2mbn:jmccc-yggdrasil-authenticator:2.4
org.to2mbn:jmccc-mcdownloader:2.4
```

假如您不使用Maven/Gradle，您就需要手动导入jar包，上述jar可以在[Maven中心仓库](https://search.maven.org/#search|ga|1|g%3A%22org.to2mbn%22)找到，并且还需要导入[org.json](https://search.maven.org/#search|ga|1|g%3A%22org.json%22%20a%3A%22json%22)和[tukaani xz](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.tukaani%22%20a%3A%22xz%22)这两个依赖。

JMCCC要求的Java**最低版本**为Java7。


## 2. 启动游戏
首先，您需要通过`LauncherBuilder`类来创建一个`Launcher`对象：
```java
Launcher launcher = LauncherBuilder.buildDefault();
```

`buildDefault()`方法会创建一个默认配置的Launcher。当然您也可以通过方法链来自定义Launcher的配置，比如：
```java
Launcher launcher = LauncherBuilder.create()
    .setDebugPrintCommandline(true) // (1)
    .setNativeFastCheck(true) // (2)
    .build();
```

|方法|意义|
|----|----|
|setDebugPrintCommandline(boolean)|（即上面的(1)）设置是否在启动时将启动参数输出到控制台以供调试，默认为false。|
|setNativeFastCheck(boolean)|（即上面的(2)）设置是否开启对Natives文件的快速检查，默认为false。<br />假如没有开启该选项，jmccc在启动时会对Natives文件进行全文比较来判断文件是否完整，如果发现Natives文件内容不一致，则将文件替换。开启该选项后，jmccc仅会通过比较文件大小来判断文件是否完整，这样可以加快启动速度，但有可能造成某些问题。|

接着，您需要创建一个`LaunchOption`来描述启动设置，比如：
```java
LaunchOption option = new LaunchOption("1.9", new OfflineAuthenticator("test_user"), new MinecraftDirectory(".minecraft")); // (3)
```

在上面的例子中，第一个参数为为要启动的Minecraft版本，这里为`1.9`。第二个参数为验证方式，这里用的是离线验证（即所谓盗版），用户名是`test_user`。第三个参数是.minecraft目录的位置，这里使用的是当前目录下的.minecraft目录。
除此之外，您还可以对LaunchOption进行其它配置，下面列举了一些LaunchOption类的方法。

|方法|意义|
|----|----|
|setMaxMemory(int)|设置最大内存（MB），默认为1024。如果为0则不会添加`-Xmx`参数（即让JVM自己决定）。|
|setMinMemory(int)|	设置最小内存（MB），默认为0。如果为0则不会添加`-Xms`参数（即让JVM自己决定）。|
|setServerInfo(ServerInfo)|设置游戏启动后要自动进入的服务器，默认为`null`。<br />例如 `new ServerInfo("localhost", 25565)` 就描述了在localhost的25565端口上的服务器。|
|setWindowSize(WindowSize)|设置游戏窗口大小，默认为`null`（不指定）。<br />例如 `WindowSize.fullscreen()` 方法返回一个代表全屏的WindowSize对象；`WindowSize.window(640, 480)` 返回一个代表了窗口大小是640x480的WindowSize。|
|setExtraJvmArguments(List<String>)|设置额外的JVM参数，默认为`null`。相关的用法可以跳读到7、8节。|
|setExtraMinecraftArguments(List<String>)|设置额外的Minecraft参数，默认为`null`。这些参数将被添加到默认Minecraft启动参数的末尾。|
|setCommandlineVariables(Map<String, String>)|设置额外的命令行模板参数，通过该方法指定的参数可以覆盖默认的参数。<br />`version.json`中的`minecraftArguments`是参数化的，其中`${...}`格式的字符串会被替换为对应变量的实际值。<br />例如，`minecraftArguments`出现了`${a}`这样的字符串，并且通过该方法指定了`"a" -> "233"`，那么启动时`${a}`就会被替换为`233`。再例如，`minecraftArguments`中出现了`${version_name}`，则这段字符串在启动时将自动被Minecraft的版本号代替。但如果通过该方法指定了`"version_name" -> "abc"`，则`${version_name}`会被`abc`代替，而不是Minecraft版本号，因为`"version_name" -> "abc"`覆盖了默认的参数。<br />为了帮助理解，下面给出一段Minecraft 1.8.9的`minecraftArguments`：<br />`--username ${auth_player_name} --version ${version_name} --gameDir ${game_directory} --assetsDir ${assets_root} --assetIndex ${assets_index_name} --uuid ${auth_uuid} --accessToken ${auth_access_token} --userProperties ${user_properties} --userType ${user_type}`<br />相关的使用可以跳读到第11节。|
|setRuntimeDirectory(MinecraftDirectory)|设置Minecraft运行时使用的目录，默认和`getMinecraftDirectory()`一样（同上面(3)中构造方法的第三个参数）。<br />这里指定的runtimeDirectory包含的是存档、资源包、截图等，而上面的minecraftDirectory包含的是游戏jar（versions）、库文件（libraries）、资源文件（assets）等。所以可以用这个方法来实现各版本独立。|


然后您就可以通过调用`launch(LaunchOption)`方法启动游戏了：
```java
launcher.launch(option);
```

假如启动失败则会抛出一个`LaunchException`，如果该`LaunchException`是一个`MissingDependenciesException`（`MissingDependenciesException`是`LaunchException`的子类），则代表有libraries缺失。

如果您想获取Minecraft进程的控制台日志，则可以使用`launch(LaunchOption, GameProcessListener)`：
```java
launcher.launch(option, new GameProcessListener() {

    @Override
    public void onLog(String log) {
        System.out.println(log); // (4)
    }

    @Override
    public void onErrorLog(String log) {
        System.err.println(log); // (5)
    }

    @Override
    public void onExit(int code) {
        System.err.println("游戏进程退出，状态码：" + code); // (6)
    }
});
```

上面代码中(4)处的`onLog(String)`方法会在游戏进程的**标准输出**输出日志时调用，(5)处的`onErrorLog(String)`方法会在游戏进程的**标准错误**输出日志时调用，而(6)处的onExit(int)会在游戏进程结束时调用。上面这段代码把游戏进程的日志都输出到了自己的控制台，并且在游戏结束时还会输出 "游戏进程退出，状态码：xxx" 这样的字符串。

下面给出一段演示代码：
```java
package yushijinhun.jmccc.test;

import org.to2mbn.jmccc.auth.OfflineAuthenticator;
import org.to2mbn.jmccc.exec.GameProcessListener;
import org.to2mbn.jmccc.launch.Launcher;
import org.to2mbn.jmccc.launch.LauncherBuilder;
import org.to2mbn.jmccc.option.LaunchOption;
import org.to2mbn.jmccc.option.MinecraftDirectory;

public class JmcccTest {

    public static void main(String[] args) throws Exception {
        // 创建一个Launcher对象
        Launcher launcher = LauncherBuilder.create()
                .setDebugPrintCommandline(true) // 将启动命令打印到控制台以便调试
                .build();

        // 启动配置
        LaunchOption option = new LaunchOption(
                "1.9", // 游戏版本
                new OfflineAuthenticator("test_user"), // 使用离线验证，用户名test_user
                new MinecraftDirectory("/home/yushijinhun/.minecraft")); // .minecraft目录

        // 最大内存2048M
        option.setMaxMemory(2048);

        // 启动游戏
        launcher.launch(option, new GameProcessListener() {

            @Override
            public void onLog(String log) {
                System.out.println(log); // 输出日志到控制台
            }

            @Override
            public void onErrorLog(String log) {
                System.err.println(log); // 输出日志到控制台（同上）
            }

            @Override
            public void onExit(int code) {
                System.err.println("游戏进程退出，状态码：" + code); // 游戏结束时输出状态码
            }
        });
    }

}
```

控制台输出：
```
jmccc:
/usr/lib/jvm/jdk1.8.0_66/jre/bin/java
-Xmx2048M
-Djava.library.path=/home/yushijinhun/.minecraft/versions/1.9/1.9-natives
-cp
/home/yushijinhun/.minecraft/libraries/com/paulscode/soundsystem/20120107/soundsystem-20120107.jar:/home/yushijinhun/.minecraft/libraries/net/java/dev/jna/platform/3.4.0/platform-3.4.0.jar:/home/yushijinhun/.minecraft/libraries/com/google/guava/guava/17.0/guava-17.0.jar:/home/yushijinhun/.minecraft/libraries/commons-io/commons-io/2.4/commons-io-2.4.jar:/home/yushijinhun/.minecraft/libraries/org/apache/httpcomponents/httpcore/4.3.2/httpcore-4.3.2.jar:/home/yushijinhun/.minecraft/libraries/com/ibm/icu/icu4j-core-mojang/51.2/icu4j-core-mojang-51.2.jar:/home/yushijinhun/.minecraft/libraries/net/sf/jopt-simple/jopt-simple/4.6/jopt-simple-4.6.jar:/home/yushijinhun/.minecraft/libraries/commons-codec/commons-codec/1.9/commons-codec-1.9.jar:/home/yushijinhun/.minecraft/libraries/com/paulscode/codecjorbis/20101023/codecjorbis-20101023.jar:/home/yushijinhun/.minecraft/libraries/commons-logging/commons-logging/1.1.3/commons-logging-1.1.3.jar:/home/yushijinhun/.minecraft/libraries/com/google/code/gson/gson/2.2.4/gson-2.2.4.jar:/home/yushijinhun/.minecraft/libraries/org/apache/commons/commons-lang3/3.3.2/commons-lang3-3.3.2.jar:/home/yushijinhun/.minecraft/libraries/com/paulscode/codecwav/20101023/codecwav-20101023.jar:/home/yushijinhun/.minecraft/libraries/com/paulscode/librarylwjglopenal/20100824/librarylwjglopenal-20100824.jar:/home/yushijinhun/.minecraft/libraries/org/apache/logging/log4j/log4j-api/2.0-beta9/log4j-api-2.0-beta9.jar:/home/yushijinhun/.minecraft/libraries/org/apache/logging/log4j/log4j-core/2.0-beta9/log4j-core-2.0-beta9.jar:/home/yushijinhun/.minecraft/libraries/io/netty/netty-all/4.0.23.Final/netty-all-4.0.23.Final.jar:/home/yushijinhun/.minecraft/libraries/org/lwjgl/lwjgl/lwjgl/2.9.4-nightly-20150209/lwjgl-2.9.4-nightly-20150209.jar:/home/yushijinhun/.minecraft/libraries/net/java/jutils/jutils/1.0.0/jutils-1.0.0.jar:/home/yushijinhun/.minecraft/libraries/com/mojang/realms/1.8.7/realms-1.8.7.jar:/home/yushijinhun/.minecraft/libraries/org/apache/commons/commons-compress/1.8.1/commons-compress-1.8.1.jar:/home/yushijinhun/.minecraft/libraries/oshi-project/oshi-core/1.1/oshi-core-1.1.jar:/home/yushijinhun/.minecraft/libraries/net/java/dev/jna/jna/3.4.0/jna-3.4.0.jar:/home/yushijinhun/.minecraft/libraries/org/lwjgl/lwjgl/lwjgl_util/2.9.4-nightly-20150209/lwjgl_util-2.9.4-nightly-20150209.jar:/home/yushijinhun/.minecraft/libraries/org/apache/httpcomponents/httpclient/4.3.3/httpclient-4.3.3.jar:/home/yushijinhun/.minecraft/libraries/net/java/jinput/jinput/2.0.5/jinput-2.0.5.jar:/home/yushijinhun/.minecraft/libraries/com/mojang/authlib/1.5.22/authlib-1.5.22.jar:/home/yushijinhun/.minecraft/libraries/com/paulscode/libraryjavasound/20101123/libraryjavasound-20101123.jar:/home/yushijinhun/.minecraft/versions/1.9/1.9.jar:
net.minecraft.client.main.Main
--username
test_user
--version
1.9
--gameDir
/home/yushijinhun/.minecraft
--assetsDir
/home/yushijinhun/.minecraft/assets
--assetIndex
1.9
--uuid
109fb39758f23a9a92c15a21ac5113c2
--accessToken
5b54b31a68574518a71fdd7e9846f87d
--userType
mojang
--versionType
release

[18:01:52] [Client thread/INFO]: Setting user: test_user
[18:01:52] [Client thread/INFO]: (Session ID is token:5b54b31a68574518a71fdd7e9846f87d:109fb39758f23a9a92c15a21ac5113c2)
[18:01:54] [Client thread/INFO]: LWJGL Version: 2.9.4
[18:01:54] [Client thread/INFO]: Reloading ResourceManager: Default
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:block.note.pling
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.bat.loop
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.cat.hiss
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.ghast.scream
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.player.breath
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.small_slime.jump
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.snowman.ambient
[18:01:54] [Client thread/WARN]: Missing sound for event: minecraft:entity.wolf.howl
[18:01:54] [Sound Library Loader/INFO]: Starting up SoundSystem...
[18:01:55] [Thread-5/INFO]: Initializing LWJGL OpenAL
[18:01:55] [Thread-5/INFO]: (The LWJGL binding of OpenAL.  For more information, see http://www.lwjgl.org)
[18:01:55] [Thread-5/INFO]: OpenAL initialized.
[18:01:55] [Sound Library Loader/INFO]: Sound engine started
[18:01:56] [Client thread/INFO]: Created: 1024x512 textures-atlas
[18:02:02] [Realms Notification Availability checker #1/INFO]: Could not authorize you against Realms server: Invalid session id
[18:02:03] [Client thread/INFO]: Stopping!
[18:02:03] [Client thread/INFO]: SoundSystem shutting down...
[18:02:03] [Client thread/WARN]: Author: Paul Lamb, www.paulscode.com
游戏进程退出，状态码：0
```


## 3. 正版登录
正版登录是jmccc的一个**可选**功能，您必须确保您已经导入了`jmccc-yggdrasil-authenticator`这个依赖。

（有必要解释一下yggdrasil的意思，yggdrasil就是mojang正版验证服务的代号）

正版登录的功能由`YggdrasilAuthenticator`类提供。

YggdrasilAuthenticator类存储了一个正版验证的session。当每次向正版验证服务器刷新session时，YggdrasilAuthenticator都会将新的session存储起来，以备下次使用。假如说因为某种原因session失效了（如长时间不使用），YggdrasilAuthenticator则会要求提供密码来重新登录。


![YggdrasilAuthenticator流程图](https://to2mbn.github.io/jmccc/images/YggdrasilAuthenticator.png)

注：上面的逻辑就是auth()方法中的逻辑。

### 3.1. 如何创建一个YggdrasilAuthenticator？
YggdrasilAuthenticator有两个工厂方法，分别是`YggdrasilAuthenticator.password(String, String)`和`YggdrasilAuthenticator.token(String, String)`。（这两个方法还有若干重载）

|方法|意义|
|----|----|
|password(String, String)|创建一个YggdrasilAuthenticator，并用所给的密码初始化。|
|token(String, String)|创建一个YggdrasilAuthenticator，并用所给的token初始化。|

使用上面这两个方法创建的YggdrasilAuthenticator都已经存储着了一个有效的session，因此您可以直接将它们拿来使用：（举第2节中的例子）
```java
LaunchOption option = new LaunchOption("1.9", YggdrasilAuthenticator.password("email@xxx.com", "password"), new MinecraftDirectory(".minecraft"));
```

需要注意的是YggdrasilAuthenticator有一个无参的构造方法。不同于上面的两个工厂方法，用这个构造方法创建出来的YggdrasilAuthenticator是不带有有效的session的。也就是说，`new YggdrasilAuthenticator()`创建出来的YggdrasilAuthenticator要刷新一次之后才能使用。

### 3.2. 如何刷新YggdrasilAuthenticator中的session？
YggdrasilAuthenticator中session的刷新分为**被动刷新**和**主动刷新**。

被动刷新就像本节一开始所说的，当要使用YggdrasilAuthenticator进行验证，但当前的session又无效时，就需要向用户询问密码来重新登录，用户此时是被动的。

YggdrasilAuthenticator默认情况下是不允许被动刷新的（因为YggdrasilAuthenticator不知道如何与用户交互），此时您需要为YggdrasilAuthenticator编写子类来实现被动刷新，例如：
```java
package yushijinhun.jmccc.test;

import java.util.Scanner;
import org.to2mbn.jmccc.auth.AuthenticationException;
import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;

public class MyYggdrasilAuthenticator extends YggdrasilAuthenticator {

        // 下面两个构造方法并没有什么好看的
        // 从超类生成过来的罢了
        public MyYggdrasilAuthenticator() {
                super();
        }

        public MyYggdrasilAuthenticator(AuthenticationService sessionService) {
                super(sessionService);
        }

        @Override
        protected PasswordProvider tryPasswordLogin() throws AuthenticationException {
                // 这个方法会在进行被动刷新时调用

                // 向用户询问邮箱与密码
                Scanner scanner = new Scanner(System.in);
                System.out.print("邮箱：");
                String email = scanner.nextLine();
                System.out.print("密码：");
                String password = scanner.nextLine();

                return YggdrasilAuthenticator.createPasswordProvider(email, password, null);
        }

}
```

我们可以来测试一下：
```java
package yushijinhun.jmccc.test;

import org.to2mbn.jmccc.auth.AuthenticationException;
import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;

public class AuthTest {

        public static void main(String[] args) throws AuthenticationException {
                // 用无参构造函数创建的YggdrasilAuthenticator是不带有session的。
                // 所以在第一次使用YggdrasilAuthenticator时会触发一次被动刷新，
                // 要求用户输入邮箱和密码。
                YggdrasilAuthenticator authenticator = new MyYggdrasilAuthenticator();

                // 循环十次要求提供登录信息
                for (int i = 0; i < 10; i++)
                        System.out.println(authenticator.auth());
        }
}
```

控制台输出：
```
邮箱：********@qq.com
密码：********
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
```

> 因为是个人敏感信息，所以我就无耻的打码了

可以看到虽然调用了使用了十次YggdrasilAuthenticator（`调用auth()`），但只向用户询问了一次密码。这说明YggdrasilAuthenticator记住了之前的登录状态。

主动刷新是指，程序主动要求YggdrasilAuthenticator刷新session。可以通过调用下面两个方法实现：

|方法|意义|
|----|----|
|refreshWithPassword(String, String)|用邮箱和密码来刷新当前session|
|refreshWithToken(String, String)|用token来刷新当前session|

我们来测试一下：
```java
package yushijinhun.jmccc.test;

import org.to2mbn.jmccc.auth.AuthenticationException;
import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;

public class AuthTest {

        public static void main(String[] args) {
                // 这样创建的YggdrasilAuthenticator不带有有效session
                // 并且我们也没有实现被动刷新
                YggdrasilAuthenticator authenticator = new YggdrasilAuthenticator();

                // 第一次使用YggdrasilAuthenticator
                // 由于没有有效session，并且也无法进行被动刷新
                // 所以会出错
                try {
                        System.out.println(authenticator.auth());
                } catch (AuthenticationException e) {
                        e.printStackTrace();
                }

                // 此时我们主动去刷新它
                try {
                        authenticator.refreshWithPassword("email@xxx.com", "password");
                } catch (AuthenticationException e) {
                        e.printStackTrace();
                }

                // 第二次使用YggdrasilAuthenticator
                // 由于经过主动刷新，已经有有效的session了
                // 所以成功执行
                try {
                        System.out.println(authenticator.auth());
                } catch (AuthenticationException e) {
                        e.printStackTrace();
                }
        }
}
```

控制台输出：
```
org.to2mbn.jmccc.auth.AuthenticationException: no more authentication methods to try
        at org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator.refresh(YggdrasilAuthenticator.java:293)
        at org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator.session(YggdrasilAuthenticator.java:265)
        at org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator.auth(YggdrasilAuthenticator.java:236)
        at yushijinhun.jmccc.test.AuthTest.main(AuthTest.java:17)
AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
```

有一点要注意：在编写和用户交互的启动器时，**使用被动刷新**。尽量不用YggdrasilAuthenticator.password()和token()这些工厂方法，也尽量避免主动刷新。因为出现被动刷新只有在**真的有必要**要用密码登录时才会发生。

### 3.3. 如何保存登录信息？
一般情况下我们的启动器都会有个“记住密码”的功能。但难道启动器真的保存了密码吗？这显然是不安全的。事实上启动器保存的是上次的session，到下一次再打开启动器时，便会加载上次的session。

那么如何在jmccc中实现这个功能呢？答案有两个。

第一种办法是直接序列化YggdrasilAuthenticator。此时YggdrasilAuthenticator中所含的session将一同被序列化。示例如下：
```java
package yushijinhun.jmccc.test;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import org.to2mbn.jmccc.auth.AuthenticationException;
import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;

public class AuthTest {

        public static void main(String[] args) throws Exception {
                // 找一个临时文件
                File file = File.createTempFile("jmccc-test", ".dat");

                saveAuth(file);
                loadAuth(file);
        }

        /**
         * 创建一个YggdrasilAuthenticator并把它序列化到文件里。
         *
         * @param file 要保存到的文件
         * @throws AuthenticationException 假如出现验证错误
         * @throws IOException 假如出现I/O异常
         */
        static void saveAuth(File file) throws AuthenticationException, IOException {
                // 用密码创建一个包含有效session的YggdrasilAuthenticator
                YggdrasilAuthenticator authenticator = YggdrasilAuthenticator.password("email@xxx.com", "password");

                try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file))) {
                        // 序列化YggdrasilAuthenticator
                        out.writeObject(authenticator);
                }

                System.out.printf("YggdrasilAuthenticator已保存到%s%n", file);
        }

        /**
         * 从文件里加载YggdrasilAuthenticator，并把它的session输出出来。
         *
         * @param file 要加载YggdrasilAuthenticator的文件
         * @throws AuthenticationException 假如出现验证错误
         * @throws IOException 假如出现I/O异常
         * @throws ClassNotFoundException 假如类未找到（反序列化错误）
         */
        static void loadAuth(File file) throws AuthenticationException, IOException, ClassNotFoundException {
                YggdrasilAuthenticator authenticator;
                try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(file))) {
                        // 反序列化YggdrasilAuthenticator
                        authenticator = (YggdrasilAuthenticator) in.readObject();
                }

                System.out.printf("从%s中加载了一个YggdrasilAuthenticator%n", file);
                System.out.printf("调用YggdrasilAuthenticator的auth()：%s%n", authenticator.auth());
        }

}
```

控制台输出：
```
YggdrasilAuthenticator已保存到/tmp/jmccc-test3142550737446373738.dat
从/tmp/jmccc-test3142550737446373738.dat中加载了一个YggdrasilAuthenticator
调用YggdrasilAuthenticator的auth()：AuthInfo [username=********, token=********, uuid=********, properties={}, userType=mojang]
```

另一种方式是，调用`getCurrentSession()`返回当前的session（可能为`null`），然后将这个对象序列化。下一次时使用无参的构造方法创建YggdrasilAuthenticator，然后调用`setCurrentSession(Session)`将session设置回去。在此便不多叙述。

### 3.4. 如何实现角色的选择？
对于这一段的小标题读者可能不大理解，什么叫角色选择呢？其实Yggdrasil允许一个账户拥有**多个**角色，这里的角色就可以相当于minecraft中的玩家。当然，mojang目前似乎还没开放这个功能。但可以通过使用第三方的Yggdrasil服务提供商体验一下（比如通过[authlib-agent](https://github.com/to2mbn/authlib-agent)）。（好像有点扯远了）

虽然mojang没有实现这个功能，但我们总得防范与未然吧，而且正版启动器以及HMCL等启动器都有实现这个功能。那么在jmccc中应该如何做到呢？

首先要为`CharacterSelector`接口编写一个实现类。这个接口中定义了一个`select(GameProfile[])`方法，即从所给的角色中选择一个。（GameProfile即角色）

下面给出一个例子：
```java
package yushijinhun.jmccc.test;

import java.util.Scanner;
import org.to2mbn.jmccc.auth.yggdrasil.CharacterSelector;
import org.to2mbn.jmccc.auth.yggdrasil.core.GameProfile;

public class MyCharacterSelector implements CharacterSelector {

        @Override
        public GameProfile select(GameProfile[] availableProfiles) {
                // 与用户交互
                for (int i = 0; i < availableProfiles.length; i++) {
                        System.out.printf("[%d] %s%n", i, availableProfiles[i].getName());
                }
                System.out.printf("请从上面的角色中选择一个（输入序号）：");
                Scanner scanner = new Scanner(System.in);
                int index = scanner.nextInt();

                // 返回要使用的角色
                return availableProfiles[index];
        }
}
```

上面我们知道了可以使用`YggdrasilAuthenticator.password(String, String)`这个工厂方法创建一个YggdrasilAuthenticator，也知道了可以用`YggdrasilAuthenticator.refreshWithPassword(String, String)`刷新当前session。
如果说我们要进行角色选择，应该用什么方法来传递给YggdrasilAuthenticator一个CharacterSelector，让它知道在选择角色时通知我们呢？
这就要使用`YggdrasilAuthenticator.password(String, String, CharacterSelector)`和`YggdrasilAuthenticator.refreshWithPassword(String, String, CharacterSelector)`。（两个重载方法）
在进行登录时，假如需要对角色进行选择，则YggdrasilAuthenticator会调用CharacterSelector的`select(GameProfile[])`。
（注：只有当可以进行角色选择时才会调用`select(GameProfile[])`）

示例如下：
（注：因为mojang不能有多个角色，所以我使用了authlib-agent自己建了个yggdrasil服务端，下面的代码和上面的会略有出入。关于jmccc自定义yggdrasil服务提供商，请见第12节）
```java
package yushijinhun.jmccc.test;

import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;
import org.to2mbn.jmccc.auth.yggdrasil.core.AuthenticationService;
import org.to2mbn.jmccc.auth.yggdrasil.core.yggdrasil.YggdrasilServiceBuilder;

public class AuthTest {

        public static void main(String[] args) throws Exception {
                AuthenticationService authenticationService = YggdrasilServiceBuilder.create()
                                .setAPIProvider(new yushijinhunYggdrasilAPIProvider())
                                .loadSessionPublicKey("/home/yushijinhun/yushijinhun_yggdrasil_pubkey.der")
                                .buildAuthenticationService();

                // 用特定的AuthenticationService创建一个AuthenticationService
                // 这样这个YggdrasilAuthenticator就会向我的Yggdrasil服务请求，而不是Mojang的
                YggdrasilAuthenticator authenticator = new YggdrasilAuthenticator(authenticationService);

                // 然后用密码登录
                authenticator.refreshWithPassword("yushijinhun@gmail.com", "123456", new MyCharacterSelector());

                // 输出登录信息
                System.out.println(authenticator.auth());
        }
}
```

控制台输出：
```
[0] test_player
[1] yushijinhun
请从上面的角色中选择一个（输入序号）：1
AuthInfo [username=yushijinhun, token=9395d1cdc7cf40a9a9fcf728aabdd7e7, uuid=8faf6e06d9e147fa8f63b9a6c19c5d5b, properties={}, userType=mojang]
```

> 因为这个yggdrasil服务端是我测试用的，在mojang服务器上并没有这个账号，所以不打码也无妨。

除了在主动刷新时进行角色选择，在被动刷新时也可以进行角色选择。只需对上面的MyYggdrasilAuthenticator做一下修改。
将tryPasswordLogin()中的return改为如下：
```java
return YggdrasilAuthenticator.createPasswordProvider(email, password, new MyCharacterSelector());
```

可以看到第三个参数发生了变化。原来是`null`，代表使用默认的角色选择器。而现在是使用我们所指定的角色选择器。

再编写代码测试一下：
```java
package yushijinhun.jmccc.test;

import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;
import org.to2mbn.jmccc.auth.yggdrasil.core.AuthenticationService;
import org.to2mbn.jmccc.auth.yggdrasil.core.yggdrasil.YggdrasilServiceBuilder;

public class AuthTest {

        public static void main(String[] args) throws Exception {
                AuthenticationService authenticationService = YggdrasilServiceBuilder.create()
                                .setAPIProvider(new yushijinhunYggdrasilAPIProvider())
                                .loadSessionPublicKey("/home/yushijinhun/yushijinhun_yggdrasil_pubkey.der")
                                .buildAuthenticationService();

                // 用特定的AuthenticationService创建一个AuthenticationService
                // 这样这个YggdrasilAuthenticator就会向我的Yggdrasil服务请求，而不是Mojang的
                YggdrasilAuthenticator authenticator = new MyYggdrasilAuthenticator(authenticationService);

                // 输出登录信息
                System.out.println(authenticator.auth());
        }
}
```

可以看到只是将上面例子里的refreshWithPassword删掉了，并且换成了MyYggdrasilAuthenticator（因为我们要处理被动刷新）。变主动刷新为被动刷新，不指定密码，让YggdrasilAuthenticator来询问我们密码。

控制台输出：
```
邮箱：yushijinhun@gmail.com
密码：123456
[0] test_player
[1] yushijinhun
请从上面的角色中选择一个（输入序号）：1
AuthInfo [username=yushijinhun, token=889af3a4d0f04277aaf3431f0a48a38e, uuid=8faf6e06d9e147fa8f63b9a6c19c5d5b, properties={}, userType=mojang]
```


第3节到此便结束了，内容可能不太好理解，但只要各位多写代码运行运行就没问题。


## 4. 下载游戏
游戏下载是jmccc的一个可选功能，您必须确保您已经导入了`jmccc-mcdownloader`这个依赖。

先介绍一下jmccc-mcdownloader中几个比较重要的类：
 * MinecraftDownloader
   * jmccc-mcdownloader中最为重要的类，提供了下载的接口方法。所有下载任务都是提供这个类提交上去的。
 * MinecraftDownloaderBuilder
   * 用来配置及创建MinecraftDownloader的类。下载时的代理、超时时间、下载源，如何检查缺失文件，缓存策略等都是通过这个类配置的。
 * Callback
   * 异步处理的回调接口。包含了`done()`，`failed()`，`cancelled()`三个回调方法，分别在任务成功完成、任务失败、任务被取消时调用。
 * DownloadTask
   * 代表了一个下载任务。每一个下载任务都有一个明确的URL表明数据来源。
 * DownloadCallback
   * 下载的异步处理的回调接口，继承自Callback。还包含了`updateProgress()`和`retry()`用来汇报下载时的进度和重试情况。
 * CombinedDownloadTask
   * 由多个下载任务组合而成的组合任务。每一个CombinedDownloadTask都可以派生出若干个DownloadTask。
 * CombinedDownloadCallback
   * 组合任务的异步处理的回调接口。还包含了`taskStart()`，在该CombinedDownloadTask派生出一个DownloadTask时会调用该方法。

首先要搞清楚DownloadTask与CombinedDownloadTask的关系。DownloadTask是下载一个文件的任务，CombinedDownloadTask是由多个DownloadTask组合而成的任务。如下图：

![DownloadTask与CombinedDownloadTask的关系](https://to2mbn.github.io/jmccc/images/CombinedDownloadTask-and-DownloadTask.png)


要下载Minecraft，首先要创建一个MinecraftDownloader对象，可以通过以下方式：
```java
MinecraftDownloader downloader = MinecraftDownloaderBuilder.buildDefault();
```

然后便可以调用它的`downloadIncrementally(MinecraftDirectory, String, CombinedDownloadCallback<Version>)`方法来下载Minecraft了。

其中第个一参数是.minecraft目录位置；第二个参数是版本名称，如`1.9`；第三个是回调接口，返回的是一个Version，代表实际下载到的Minecraft版本。

downloadIncrementally方法会自动检查缺失的文件（如libraries、assets），并下载。

MinecraftDownloader中所有的downloadXXX/fetchXXX方法都是异步的（当然也包括上面的downloadIncrementally）。调用之后会立即返回。任务完成后会通知回调。

如下面的这段代码下载了Minecraft 1.9：
（注：可以使用CallbackAdapter来避免接口内写重复的空方法，类似于swing的Adapter）
```java
package yushijinhun.jmccc.test;

import org.to2mbn.jmccc.mcdownloader.MinecraftDownloader;
import org.to2mbn.jmccc.mcdownloader.MinecraftDownloaderBuilder;
import org.to2mbn.jmccc.mcdownloader.download.DownloadCallback;
import org.to2mbn.jmccc.mcdownloader.download.DownloadTask;
import org.to2mbn.jmccc.mcdownloader.download.concurrent.CallbackAdapter;
import org.to2mbn.jmccc.option.MinecraftDirectory;
import org.to2mbn.jmccc.version.Version;

public class DownloadTest {

        public static void main(String[] args) {
                // 下载位置（要下载到的.minecraft目录）
                MinecraftDirectory dir = new MinecraftDirectory("/home/yushijinhun/.minecraft");

                // 创建MinecraftDownloader
                MinecraftDownloader downloader = MinecraftDownloaderBuilder.create().build();

                // 下载Minecraft1.9
                downloader.downloadIncrementally(dir, "1.9", new CallbackAdapter<Version>() {

                        @Override
                        public void done(Version result) {
                                // 当完成时调用
                                // 参数代表实际下载到的Minecraft版本
                                System.out.printf("下载完成，下载到的Minecraft版本：%s%n", result);
                        }

                        @Override
                        public void failed(Throwable e) {
                                // 当失败时调用
                                // 参数代表是由于哪个异常而失败的
                                System.out.printf("下载失败%n");
                                e.printStackTrace();
                        }

                        @Override
                        public void cancelled() {
                                // 当被取消时调用
                                System.out.printf("下载取消%n");
                        }

                        @Override
                        public <R> DownloadCallback<R> taskStart(DownloadTask<R> task) {
                                // 当有一个下载任务被派生出来时调用
                                // 在这里返回一个DownloadCallback就可以监听该下载任务的状态
                                System.out.printf("开始下载：%s%n", task.getURI());
                                return new CallbackAdapter<R>() {

                                        @Override
                                        public void done(R result) {
                                                // 当这个DownloadTask完成时调用
                                                System.out.printf("子任务完成：%s%n", task.getURI());
                                        }

                                        @Override
                                        public void failed(Throwable e) {
                                                // 当这个DownloadTask失败时调用
                                                System.out.printf("子任务失败：%s。原因：%s%n", task.getURI(), e);
                                        }

                                        @Override
                                        public void cancelled() {
                                                // 当这个DownloadTask被取消时调用
                                                System.out.printf("子任务取消：%s%n", task.getURI());
                                        }

                                        @Override
                                        public void retry(Throwable e, int current, int max) {
                                                // 当这个DownloadTask因出错而重试时调用
                                                // 重试不代表着失败
                                                // 也就是说，一个DownloadTask可以重试若干次，
                                                // 每次决定要进行一次重试时就会调用这个方法
                                                // 当最后一次重试失败，这个任务也将失败了，failed()才会被调用
                                                // 所以调用顺序就是这样：
                                                // retry()->retry()->...->failed()
                                                System.out.printf("子任务重试[%d/%d]：%s。原因：%s%n", current, max, task.getURI(), e);
                                        }
                                };
                        }
                });
        }
}
```

当下载成功时控制台输出：
```
开始下载：https://launchermeta.mojang.com/mc/game/version_manifest.json
子任务完成：https://launchermeta.mojang.com/mc/game/version_manifest.json
开始下载：https://launchermeta.mojang.com/mc/game/6768033e216468247bd031a0a2d9876d79818f8f/1.9.json
子任务完成：https://launchermeta.mojang.com/mc/game/6768033e216468247bd031a0a2d9876d79818f8f/1.9.json
开始下载：https://launchermeta.mojang.com/mc-staging/assets/1.9/092c59b361816c7fa7f000587caa977c515b179c/1.9.json
开始下载：https://launcher.mojang.com/mc/game/1.9/client/2f67dfe8953299440d1902f9124f0f2c3a2c940f/client.jar
开始下载：https://libraries.minecraft.net/com/mojang/authlib/1.5.22/authlib-1.5.22.jar
子任务完成：https://launchermeta.mojang.com/mc-staging/assets/1.9/092c59b361816c7fa7f000587caa977c515b179c/1.9.json
开始下载：http://resources.download.minecraft.net/4b/4b90ff3a9b1486642bc0f15da0045d83a91df82e
子任务完成：http://resources.download.minecraft.net/4b/4b90ff3a9b1486642bc0f15da0045d83a91df82e
子任务完成：https://libraries.minecraft.net/com/mojang/authlib/1.5.22/authlib-1.5.22.jar
下载完成，下载到的Minecraft版本：1.9
子任务完成：https://launcher.mojang.com/mc/game/1.9/client/2f67dfe8953299440d1902f9124f0f2c3a2c940f/client.jar
```

当下载失败时（通过拔网线实现）控制台输出：
```
开始下载：https://launchermeta.mojang.com/mc/game/version_manifest.json
开始下载：https://launchermeta.mojang.com/mc/game/6768033e216468247bd031a0a2d9876d79818f8f/1.9.json
子任务完成：https://launchermeta.mojang.com/mc/game/version_manifest.json
子任务完成：https://launchermeta.mojang.com/mc/game/6768033e216468247bd031a0a2d9876d79818f8f/1.9.json
开始下载：https://launchermeta.mojang.com/mc-staging/assets/1.9/092c59b361816c7fa7f000587caa977c515b179c/1.9.json
开始下载：https://launcher.mojang.com/mc/game/1.9/client/2f67dfe8953299440d1902f9124f0f2c3a2c940f/client.jar
开始下载：https://libraries.minecraft.net/com/mojang/authlib/1.5.22/authlib-1.5.22.jar
子任务重试[1/3]：https://launcher.mojang.com/mc/game/1.9/client/2f67dfe8953299440d1902f9124f0f2c3a2c940f/client.jar。原因：java.net.UnknownHostException: launcher.mojang.com: unknown error
子任务重试[2/3]：https://launcher.mojang.com/mc/game/1.9/client/2f67dfe8953299440d1902f9124f0f2c3a2c940f/client.jar。原因：java.net.UnknownHostException: launcher.mojang.com
子任务失败：https://launcher.mojang.com/mc/game/1.9/client/2f67dfe8953299440d1902f9124f0f2c3a2c940f/client.jar。原因：java.net.UnknownHostException: launcher.mojang.com
子任务重试[1/3]：https://libraries.minecraft.net/com/mojang/authlib/1.5.22/authlib-1.5.22.jar。原因：java.net.UnknownHostException: libraries.minecraft.net: unknown error
子任务重试[2/3]：https://libraries.minecraft.net/com/mojang/authlib/1.5.22/authlib-1.5.22.jar。原因：java.net.UnknownHostException: libraries.minecraft.net
子任务失败：https://libraries.minecraft.net/com/mojang/authlib/1.5.22/authlib-1.5.22.jar。原因：java.net.UnknownHostException: libraries.minecraft.net
子任务取消：https://launchermeta.mojang.com/mc-staging/assets/1.9/092c59b361816c7fa7f000587caa977c515b179c/1.9.json
下载失败
java.net.UnknownHostException: launcher.mojang.com
        at java.net.InetAddress.getAllByName0(InetAddress.java:1280)
......// 此处省略
```

如果说要下载Minecraft版本列表，则需要调用`fetchRemoteVersionList(CombinedDownloadCallback<RemoteVersionList>)`方法。如下：
```java
    downloader.fetchRemoteVersionList(new CallbackAdapter<RemoteVersionList>() {

            @Override
            public void done(RemoteVersionList result) {
                    System.out.printf("版本列表下载完成：%s%n", result);
            }
            
            // ............省略其它方法
    });
```

控制台输出如下：
```
版本列表下载完成：[latestSnapshot=1.RV-Pre1, latestRelease=1.9.2, versions={16w05b=RemoteVersion [version=16w05b, ............
```

注：如果说要在下载完后启动Minecraft的话，可以直接将Version对象传进LaunchOption的构造函数中。另外实际下载到的Minecraft的version id可能会与downloadIncrementally指定的不同，一般出现在下载forge时（下一节会有介绍）。

当使用MinecraftDownloaderBuilder创建MinecraftDownloader时，可以通过方法链来自定义配置。下面是MinecraftDownloaderBuilder里的一些方法：

|方法|意义|
|----|----|
|setMaxConnections(int)|设置下载时的最大链接数|
|setMaxConnectionsPerRouter(int)|设置NIO下每个I/O Dispatcher线程的最大链接数|
|setConnectTimeout(int)|设置连接超时的毫秒数|
|setSoTimeout(int)|设置Socket超时的毫秒数|
|setBaseProvider(MinecraftDownloadProvider)|设置下载源|
|appendProvider(MinecraftDownloadProvider)|将一个拓展下载源添加到解析链中|
|setPoolMaxThreads(int)|设置线程池的最大线程数|
|setPoolThreadLivingTime(long)|设置线程池里线程在不使用后最大的存活时间（毫秒）|
|setDefaultTries(int)|设置下载失败后最大的尝试次数（默认为3，不宜过大）|
|setUseVersionDownloadInfo(boolean)|设置是否从json中指定的url下载（即1.9的新json格式，默认true）|
|setCheckAssetsHash(boolean)|设置是否通过计算assets的hash来判断完整性（默认true）|
|setCheckLibrariesHash(boolean)|设置是否通过计算libraries的hash来判断完整性（默认false，文件hash会与1.9新json中指定的hash值比较）|

关于如何使用自定义的下载源，下面以BMCL API为例：
```java
package org.to2mbn.jmccc.mcdownloader.wiki.provider;

import org.to2mbn.jmccc.mcdownloader.provider.DefaultLayoutProvider;

public class BmclApiProvider extends DefaultLayoutProvider {

        @Override
        protected String getLibraryBaseURL() {
                return "http://bmclapi2.bangbang93.com/libraries/";
        }

        @Override
        protected String getVersionBaseURL() {
                return "http://bmclapi2.bangbang93.com/versions/";
        }

        @Override
        protected String getAssetIndexBaseURL() {
                return "http://bmclapi2.bangbang93.com/indexes/";
        }

        @Override
        protected String getVersionListURL() {
                return "http://bmclapi2.bangbang93.com/mc/game/version_manifest.json";
        }

        @Override
        protected String getAssetBaseURL() {
                return "http://bmclapi2.bangbang93.com/assets/";
        }

}
```

然后只要在创建MinecraftDownloader时，调用setBaseProvider即可：
```java
MinecraftDownloader downloader = MinecraftDownloaderBuilder.create()
        .setBaseProvider(new BmclApiProvider())
        .build();
```


最后，您必须**手动关闭**MinecraftDownloader，否则MinecraftDownloader占用的资源将不会释放（如缓存、线程、链接等）：
```java
downloader.shutdown();
```

要注意的是，MinecraftDownloader是一个重量级对象，创建和销毁都会消耗大量的系统资源。所以在一般情况下推荐启动器全局**共用**一个MinecraftDownloader对象。

