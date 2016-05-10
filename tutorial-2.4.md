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


## 5. 下载并安装Forge及Liteloader
本节将介绍Forge、Liteloader版本列表的获取，以及Forge、Liteloader的下载安装。

我们知道，Forge和Liteloader的版本在1.6（新.minecraft目录格式）之后，在versions目录中都是单独算一个版本的，比如`1.7.10-LiteLoader1.7.10`、`1.8.9-forge1.8.9-11.15.1.1757`。在jmccc也是这样，`1.7.10-LiteLoader1.7.10`、`1.8.9-forge1.8.9-11.15.1.1757`它们都是一个Minecraft版本，与`1.9`、`1.7.10`这样的版本是同等地位的，所以这些Forge、Liteloader版本可以像正常的Minecraft版本一样下载、启动。

要支持Forge和Liteloader，首先要创建一个`ForgeDownloadProvider`和`LiteloaderDownloadProvider`。它们提供了对Forge、Liteloader的解析。然后通过MinecraftDownloaderBuilder的appendProvider方法将它们添加到MinecraftDownloader的解析链中：
```java
ForgeDownloadProvider forgeProvider = new ForgeDownloadProvider();
LiteloaderDownloadProvider liteloaderProvider = new LiteloaderDownloadProvider();
MinecraftDownloader downloader = MinecraftDownloaderBuilder.create()
        .appendProvider(forgeProvider)
        .appendProvider(liteloaderProvider)
        .build();
```

注意：`appendProvider(liteloaderProvider)`需要在`appendProvider(forgeProvider)`之后调用，否则将不能下载同时Forge、Liteloader并存的版本。（否则责任链中LiteloaderDownloadProvider将无法把下载forge的任务传递到ForgeDownloadProvider）

这样，这个MinecraftDownloader就具备了下载Forge、Liteloader的能力。

要下载Forge或Liteloader，首先要获取它们的版本列表，如下：
```java
downloader.download(forgeProvider.forgeVersionList(), new CallbackAdapter<ForgeVersionList>() {...});
downloader.download(liteloaderProvider.liteloaderVersionList(), new CallbackAdapter<LiteloaderVersionList>() {...});
```

> 上面省略了Callback中的方法

对于`ForgeVersionList`，有以下方法：

|方法|意义|
|----|----|
|getVersions()|获取所有的ForgeVersion。返回一个Map，key为build number。|
|getLatests()|获取所有标记为latest的版本，即每个Minecraft版本所对应的最新的ForgeVersion。返回一个Map，key为minecraft版本，value是此minecraft版本对应的最新的ForgeVersion。|
|getRecommendeds()|获取所有标记为recommended的版本，即每个Minecraft版本所对应的推荐的ForgeVersion。返回一个Map，key为minecraft版本，value是此minecraft版本对应的推荐的ForgeVersion。|
|getLatest()|获取最最新的ForgeVersion。不考虑minecraft版本。|
|getLatest(String)|获取给定的minecraft版本最新的ForgeVersion。|
|getRecommended()|获取最新的推荐的ForgeVersion。不考虑minecraft版本。|
|getRecommended(String)|获取给定的minecraft版本推荐的ForgeVersion。|

对于`LiteloaderVersionList`，有以下方法:

|方法|意义|
|----|----|
|getLatests()|获取每个minecraft版本对应的最新的LiteloaderVersion。返回的是Map，key为minecraft版本，value为该版本对应的最新的LiteloaderVersion。|
|getLatest(String)|获取给定的minecraft版本最新的LiteloaderVersion。|

> 因为Liteloader开发者所提供的api过于变态，因此不支持snapshot版本，并且只能下载某个minecraft版本对应的最新的liteloader。
> （在2.5中会加入对liteloader snapshot的支持）

在获取到要下载的ForgeVersion或LiteloaderVersion后就可以正常下载了，比如：
```java
// 这里的forgeVersion即要下载的forge版本
downloader.downloadIncrementally(dir, forgeVersion.getVersionName(), new CallbackAdapter<Version>() {......});
```

可以看到和上文下载官方的minecraft版本并没有什么区别，只是使用了ForgeVersion的`getVersionName()`来作为要下载的minecraft的版本号罢了。要下载LiteloaderVersion如法炮制即可。


如果说要下载一个Forge和Liteloader并存的minecraft该怎么办呢？

假如我已经挑选好了ForgeVersion和LiteloaderVersion（forge和liteloader对应的minecraft版本要一致，不然肯定不能启动），那么只要将downloadIncrementally中的版本号替换为：
```java
liteloaderVersion.customize(forgeVersion.getVersionName()).getVersionName()
```


至此，Forge、Liteloader相关内容就讲解完了。


## 6. 获取正版玩家的皮肤
> 从本节开始，我们就将开始介绍jmccc的高级用法。一般来说，听不懂是正常的。;(手动斜眼

第3节中，我们学到了YggdrasilAuthenticator。事实上，YggdrasilAuthenticator的底层是AuthenticationService，AuthenticationService提供了一组访问Yggdrasil服务的接口。而与AuthenticationService并行的就是ProfileService，提供了和游戏角色相关的接口。

我们可以通过下面的方法来创建一个ProfileService：
```java
ProfileService profileService = YggdrasilServiceBuilder.defaultProfileService();
```

ProfileService中有三个方法，它们分别是：

|方法|意义|
|----|----|
|lookupUUIDByName(String)|查询与玩家游戏中的名称对应的UUID。|
|getGameProfile(UUID)|查询给定UUID的角色的信息。|
|getTextures(GameProfile)|从给定的角色信息中获取皮肤（等）。|

> 上面这三个方法都是要访问网络的，并且会阻塞，所以千万别脑残在UI线程之类的里面调用。

下面给出一个例子：
```java
package yushijinhun.jmccc.test;

import java.util.UUID;
import org.to2mbn.jmccc.auth.AuthenticationException;
import org.to2mbn.jmccc.auth.yggdrasil.core.GameProfile;
import org.to2mbn.jmccc.auth.yggdrasil.core.PlayerTextures;
import org.to2mbn.jmccc.auth.yggdrasil.core.ProfileService;
import org.to2mbn.jmccc.auth.yggdrasil.core.yggdrasil.YggdrasilServiceBuilder;

public class ProfileServiceTest {

        public static void main(String[] args) throws AuthenticationException {
                ProfileService profileService = YggdrasilServiceBuilder.defaultProfileService();

                // 查询ztcjohn的uuid
                UUID uuid = profileService.lookupUUIDByName("ztcjohn");
                System.out.println(uuid);

                // 根据uuid获取GameProfile
                // 注意啦！假如lookupUUIDByName没找到对应玩家就会返回null。我这里因为是演示所以偷懒不做判断，大家写的时候记得一定要判断一下
                GameProfile profile = profileService.getGameProfile(uuid);

                // 获取GameProfile的皮肤
                // 同上，假如getGameProfile没找到对应玩家就会返回null。我这里是偷懒，大家写的时候千万不要学
                PlayerTextures textures = profileService.getTextures(profile);
                System.out.println(textures);
        }
}

```

控制台输出：
```
469aa369-6304-40d4-8b99-7e63199677ac
PlayerTextures [skin=Texture [url=http://textures.minecraft.net/texture/2b28e73ff96914596963cb468da14fdb5e36217a6e327e8353efb44ec71, metadata=null], cape=Texture [url=http://textures.minecraft.net/texture/efd61c3c4ac88f1a3468fbdeef45cec89e5afb87b97a1a845bfb3c64fd0b883, metadata=null], elytra=null]
```

可以看到ztcjohn不但有一个皮肤还有一个披风。getTextures方法返回的PlayerTextures包含了皮肤、披风、elytra（1.9新加的滑翔翼）。要注意的是，并不是每个角色都有这三个东西。

（对不起咯～ztc喵）

PlayerTextures只包含皮肤的url，具体图像需要再去下载。

如果说要判断一个角色是Alex还是Steve，可以通过下面这个方法：
```java
public static boolean isAlex(PlayerTextures textures) {
        Texture skin = textures.getSkin();
        if (skin != null) {
                Map<String, String> metadata = skin.getMetadata();
                if (metadata != null) {
                        return "slim".equals(metadata.get("model"));
                }
        }
        return false;
}
```


# 7. OS X下Dock相关配置
官方启动器在OSX下时，启动Minecraft的时候会添加一些和Dock相关的参数，用来设置Minecraft在Dock中的呈现。（貌似是这样，本人没使用过OSX）

在jmccc中，您需要将`ExtraArgumentsTemplates.OSX_DOCK_NAME`和`ExtraArgumentsTemplates.OSX_DOCK_ICON(MinecraftDirectory, Version)`的返回值加进JVM参数中。在此之前，**务必**要检查当前系统是否为OSX，这两个参数只有在OSX下的JVM里才是有效的。

比如：
```java
MinecraftDirectory dir = new MinecraftDirectory(".minecraft");
LaunchOption option = new LaunchOption("1.9", new OfflineAuthenticator("test_player"), dir);

// 一定要先判断是否为OSX
if (Platform.CURRENT == Platform.OSX) {
        option.setExtraJvmArguments(Arrays.asList(
                        ExtraArgumentsTemplates.OSX_DOCK_NAME,
                        ExtraArgumentsTemplates.OSX_DOCK_ICON(dir, option.getVersion())));
}
```


# 8. 忽略Forge的数字签名
有些Forge版本要添加`-Dfml.ignoreInvalidMinecraftCertificates=true`和`-Dfml.ignorePatchDiscrepancies=true`这两项JVM参数才能正常启动。出现这种情况一般是往jar里塞了一些奇怪的东西，然后FML发现jar的数字签名损坏了。解决方法是往JVM参数列表里加上这两个参数（同前一节）。

当然，jmccc是不会让您手动打这两项参数的。这两个参数都已经在`ExtraArgumentsTemplates`里面预定义好了，只需引用即可。

```java
option.setExtraJvmArguments(Arrays.asList(
                ExtraArgumentsTemplates.FML_IGNORE_INVALID_MINECRAFT_CERTIFICATES,
                ExtraArgumentsTemplates.FML_IGNORE_PATCH_DISCREPANCISE));
```


# 9. 使用HttpAsyncClient来进行下载
下载Minecraft时一般会下载大量文件（上千个），如果逐一下载效率固然很低，所以jmccc是允许多个任务同时下载的。默认情况下，jmccc用的是jdk自带的BIO（阻塞式I/O），会给每一个链接打开一个线程。当链接数很大时，便会造成系统资源的严重浪费。解决方法是切换到NIO（非阻塞式I/O），这样仅用数个线程便可以处理上万个链接，将下载速度最大化。

如果说要在jmccc中使用NIO来下载，则需要添加Apache HttpAsyncClient这个依赖：
```
org.apache.httpcomponents:httpasyncclient:4.1.1
```

jmccc在初始化时，如果发现classpath中存在HttpAsyncClient，则会自动使用HttpAsyncClient来下载。
在将HttpAsyncClient添加到classpath中后，您便可以将最大链接数调到任意大了：
```java
MinecraftDownloader downloader = MinecraftDownloaderBuilder.create()
        .setMaxConnections(4096)
        .build();
```

比如这样将MaxConnections调到了4096，那么最多可以同时下载4096个文件。

如果说您不想使用HttpAsyncClient，您可以调用MinecraftDownloaderBuilder的`disableApacheHttpAsyncClient()`来禁用对HttpAsyncClient的支持。这样jmccc就只会使用BIO来下载文件。


# 10. 使用Ehcache缓存下载的文件
jmccc有时候会多次下载同一个文件，这样就会造成不必要的网络I/O和等待时间。所以jmccc提供了对Ehcache的支持（Ehcache是最有名的一个轻量级Java缓存框架）。要启用这个缓存功能首先要将Ehcache添加到依赖，需要注意的是jmccc使用的是**Ehcache3**，不是2。

> ehcache使用了slf4j作为logging框架，推荐您的项目中至少包含一个slf4j的实现。

将ehcache添加到classpath之后，您便可以对缓存进行配置了。

ehcache提供了三种类型的缓存：
 * 堆上缓存（heap）
   * 存储在java堆上的缓存
 * 离堆缓存（offheap）
   * 存储在本地内存中的缓存，在java堆之外，不受jvm托管
 * 磁盘缓存（disk）
   * 存储在磁盘上的缓存

jmccc默认开启32MB的堆上缓存，不开启离堆缓存和磁盘缓存，缓存的有效时间是2小时。
您可以使用下面的这些方法来设置缓存：

|方法|意义|
|----|----|
|setHeapCacheSize(long)|设置Java堆上缓存的最大大小，0则不开启，单位：MB。|
|setOffheapCacheSize(long)|设置离堆缓存（本地内存）的最大大小，0则不开启，单位：MB。|
|setDiskCacheSize(long)|设置磁盘上缓存的最大大小，0则不开启，单位：MB。开启该功能后还需调用setDiskCacheDir(File)进行设置。|
|setDiskCacheDir(File)|设置磁盘上用于存储缓存的目录。|
|setCacheLiveTime(long, TimeUnit)|配置缓存的有效时间（TTL），超过该时间的缓存将被自动清除。|

比如下面这段代码，只将最大为128M的缓存存储在磁盘上，有效时间1天：
```java
MinecraftDownloader downloader = MinecraftDownloaderBuilder.create()
        .setHeapCacheSize(0) // 关闭堆上缓存
        .setOffheapCacheSize(0) // 关闭离堆缓存
        .setDiskCacheSize(128) // 开启最大为128MB的磁盘缓存
        .setDiskCacheDir(new File("/tmp/jmccc-cache")) // 存储缓存的目录是/tmp/jmccc-cache
        .setCacheLiveTime(1, TimeUnit.DAYS)// 缓存有效时间1天
        .build();
```

如果说您不想使用缓存，那您可以调用MinecraftDownloaderBuilder的`disableEhcache()`方法来禁用对Ehcache的支持。


## 11. 修改Minecraft1.9中的version_type
大家用HMCL启动Minecraft1.9的时候可能会发现，Minecraft主界面下面有这样的东西：

![HMCL修改version_type](https://to2mbn.github.io/jmccc/images/hmcl-version-type.png)

这是怎么实现的呢？

可以打开1.9.json看看，发现其中有这样一段：
```json
"minecraftArguments": "--username ${auth_player_name} --version ${version_name} --gameDir ${game_directory} --assetsDir ${assets_root} --assetIndex ${assets_index_name} --uuid ${auth_uuid} --accessToken ${auth_access_token} --userType ${user_type} --versionType ${version_type}"
```

不知大家有没有注意到`${version_type}`这一个字符串。官方启动器在启动时会自动用Minecraft版本的type来代替这段字符串，比如`snapshot`、`release`，这样在Minecraft底部就显示为`Minecraft 1.9-pre2/snapshot`，`Minecraft 1.9/release`。但HMCL则将它替换为了`HMCL 2.4.1.41`。

jmccc的行为默认和官方启动器一样，会将version_type指定为`Version.getType()`。但jmccc也提供了一个覆写该变量的方法，它就是第2节中介绍的setCommandlineVariables。

我们可以手动指定`version_type`的值，如下：
```java
package yushijinhun.jmccc.test;

import java.util.HashMap;
import java.util.Map;
import org.to2mbn.jmccc.auth.OfflineAuthenticator;
import org.to2mbn.jmccc.launch.Launcher;
import org.to2mbn.jmccc.launch.LauncherBuilder;
import org.to2mbn.jmccc.option.LaunchOption;
import org.to2mbn.jmccc.option.MinecraftDirectory;

public class JmcccTest {

        public static void main(String[] args) throws Exception {
                Launcher launcher = LauncherBuilder.buildDefault();
                LaunchOption option = new LaunchOption("1.9", new OfflineAuthenticator("test_user"), new MinecraftDirectory("/home/yushijinhun/.minecraft"));

                // 重点：手动指定version_type
                Map<String, String> vars = new HashMap<>();
                vars.put("version_type", "JMCCC大法好！");
                option.setCommandlineVariables(vars);

                launcher.launch(option);
        }
}
```

运行效果：
![JMCCC修改version_type](https://to2mbn.github.io/jmccc/images/jmccc-version-type.png)


## 12. 使用自定义的Yggdrasil API提供商
> 如果您没有深入了解过Yggdrasil服务，那么本节对您来说可能有些困难。
> 
> wiki.vg上有一篇介绍Yggdrasil的条目（英文）：[http://wiki.vg/Authentication](http://wiki.vg/Authentication)

正版验证（即Yggdrasil）是Mojang提供的服务，但这不一定必须由Mojang提供。也就是说，您可以创建一套和Mojang并行的Yggdrasil服务，相当于Yggdrasil私服。事实上，yushijinhun（我）的authlib-agent项目就已经成功通过字节码操纵实现了对Minecraft内Yggdrasil API的重定向，并给出了一个Yggdrasil服务的开源实现。（本节的部分内容曾在第3节中出现，所以对下面的代码您可能会感到有些眼熟？）

在jmccc中，如果要手动指定Yggdrasil API提供商，就得先为YggdrasilAPIProvider编写一个子类，里面定义API的URL。

我在本地用authlib-agent架设了一个yggdrasil服务端，给YggdrasilAPIProvider编写的子类如下：
```java
package yushijinhun.jmccc.test;

import java.util.UUID;
import org.to2mbn.jmccc.auth.yggdrasil.core.yggdrasil.YggdrasilAPIProvider;
import org.to2mbn.jmccc.util.UUIDUtils;

public class yushijinhunYggdrasilAPIProvider implements YggdrasilAPIProvider {

        @Override
        public String authenticate() {
                return "http://localhost:8080/yggdrasil/authenticate";
        }

        @Override
        public String refresh() {
                return "http://localhost:8080/yggdrasil/refresh";
        }

        @Override
        public String validate() {
                return "http://localhost:8080/yggdrasil/validate";
        }

        @Override
        public String invalidate() {
                return "http://localhost:8080/yggdrasil/invalidate";
        }

        @Override
        public String signout() {
                return "http://localhost:8080/yggdrasil/signout";
        }

        @Override
        public String profile(UUID profileUUID) {
                return "http://localhost:8080/yggdrasil/profiles/minecraft/" + UUIDUtils.unsign(profileUUID);
        }

        @Override
        public String profileLookup() {
                return "http://localhost:8080/yggdrasil/profilerepo/minecraft";
        }
}
```

然后您就可以用YggdrasilServiceBuilder来配置AuthenticationService和ProfileService了：
```java
YggdrasilServiceBuilder yggdrasilBuilder = YggdrasilServiceBuilder.create()
        .setAPIProvider(new yushijinhunYggdrasilAPIProvider())
        .loadSessionPublicKey("/home/yushijinhun/yushijinhun_yggdrasil_pubkey.der"); //(1)
ProfileService profileService = yggdrasilBuilder.buildProfileService(); //(2)
AuthenticationService authenticationService = yggdrasilBuilder.buildAuthenticationService(); //(3)
```

上面代码中(1)代表从`/home/yushijinhun/yushijinhun_yggdrasil_pubkey.der`这个文件中加载Yggdrasil的数字签名公钥。该文件应是PKCS#8格式的RSA证书的SubjectPublicKeyInfo部分（与mojang authlib使用的密钥格式相同）。您也可以使用`setSessionPublicKey(PublicKey)`方法直接设置公钥。

(2)代表用上面的YggdrasilServiceBuilder创建一个ProfileService。这个ProfileService是绑定到上面指定的Yggdrasil API上的，上面第6节有介绍。

(3)代表用上面的YggdrasilServiceBuilder创建一个AuthenticationService。这个AuthenticationService是绑定到上面指定的Yggdrasil API上的。

您可以通过`new YggdrasilAuthenticator(authenticationService)`来创建一个使用指定的AuthenticationService的YggdrasilAuthenticator。

需要注意的是，如果您要序列化YggdrasilAuthenticator，请**务必**重写`createAuthenticationServiceForDeserialization()`方法。这个方法用来在反序列化过程中重新创建AuthenticationService。
```java
package yushijinhun.jmccc.test;

import java.io.IOException;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;
import org.to2mbn.jmccc.auth.yggdrasil.YggdrasilAuthenticator;
import org.to2mbn.jmccc.auth.yggdrasil.core.AuthenticationService;
import org.to2mbn.jmccc.auth.yggdrasil.core.yggdrasil.YggdrasilServiceBuilder;

public class MyYggdrasilAuthenticator extends YggdrasilAuthenticator {

        /**
         * 创建一个我自定义的AuthenticationService。
         * 
         * @return 我自定义的AuthenticationService
         */
        private static AuthenticationService createMyAuthenticationService() {
                try {
                        return YggdrasilServiceBuilder.create()
                                        .setAPIProvider(new yushijinhunYggdrasilAPIProvider())
                                        .loadSessionPublicKey("/home/yushijinhun/yushijinhun_yggdrasil_pubkey.der")
                                        .buildAuthenticationService();
                } catch (NoSuchAlgorithmException | InvalidKeySpecException | IOException e) {
                        throw new IllegalStateException("无法创建自定义的AuthenticationService！", e);
                }
        }

        public MyYggdrasilAuthenticator() {
                // 使用自定义的AuthenticationService
                super(createMyAuthenticationService());
        }

        @Override
        protected AuthenticationService createAuthenticationServiceForDeserialization() {
                // 在反序列化过程中重新创建我自定义的AuthenticationService
                return createMyAuthenticationService();
        }
}
```

题外话：由于某些原因（比如gfw），会导致国内有时候连不上Mojang的Yggdrasil服务，这时候可以用YggdrasilServiceBuilder的`setProxy(Proxy)`指定一个代理。


