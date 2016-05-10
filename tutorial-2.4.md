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
|setServerInfo(ServerInfo)|设置游戏启动后要自动进入的服务器，默认为`null`。例如 `new ServerInfo("localhost", 25565)` 就描述了在localhost的25565端口上的服务器。|
|setWindowSize(WindowSize)|设置游戏窗口大小，默认为`null`（不指定）。例如 `WindowSize.fullscreen()` 方法返回一个代表全屏的WindowSize对象；`WindowSize.window(640, 480)` 返回一个代表了窗口大小是640x480的WindowSize。|
|setExtraJvmArguments(List<String>)|设置额外的JVM参数，默认为`null`。相关的用法可以跳读到7、8节。|
|setExtraMinecraftArguments(List<String>)|设置额外的Minecraft参数，默认为`null`。这些参数将被添加到默认Minecraft启动参数的末尾。|
|setCommandlineVariables(Map<String, String>)|设置额外的命令行模板参数，通过该方法指定的参数可以覆盖默认的参数。<br />`version.json`中的`minecraftArguments`是参数化的，其中`${...}`格式的字符串会被替换为对应变量的实际值。<br />例如，`minecraftArguments`出现了`${a}`这样的字符串，并且通过该方法指定了`"a" -> "233"`，那么启动时`${a}`就会被替换为`233`。再例如，`minecraftArguments`中出现了`${version_name}`，则这段字符串在启动时将自动被Minecraft的版本号代替。但如果通过该方法指定了`"version_name" -> "abc"`，则`${version_name}`会被`abc`代替，而不是Minecraft版本号，因为`"version_name" -> "abc"`覆盖了默认的参数。<br />为了帮助理解，下面给出一段Minecraft 1.8.9的`minecraftArguments`：<br />`--username ${auth_player_name} --version ${version_name} --gameDir ${game_directory} --assetsDir ${assets_root} --assetIndex ${assets_index_name} --uuid ${auth_uuid} --accessToken ${auth_access_token} --userProperties ${user_properties} --userType ${user_type}`<br />相关的使用可以跳读到第11节。|
|setRuntimeDirectory(MinecraftDirectory)|设置Minecraft运行时使用的目录，默认和`getMinecraftDirectory()`一样（同上面(3)中构造方法的第三个参数）。这里指定的runtimeDirectory包含的是存档、资源包、截图等，而上面的minecraftDirectory包含的是游戏jar（versions）、库文件（libraries）、资源文件（assets）等。所以可以用这个方法来实现各版本独立。|


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

