# Android Builder

该仓库是使用 Github Action 自动编译 Android 项目的一种展示。具体解释可见下面的文章：

[《更新慢、弃坑了？实现 Android 应用自给自足：GitHub Actions 编译实例》](https://sspai.com/post/70427)

如果这篇文章帮到了你，不妨回来点个赞。

## 为什么选择 GitHub Actions
开箱即用的环境。从 GitHub 官方的虚拟环境仓库可以看到，Ubuntu 20.04 的环境中自带了 Java 11，Kotlin 1.6.0，Gradle 7.3，和 Android SDK 的常用版本。相比上面的传统方法，Ubuntu 这套环境其实已经解决了很多编译环境问题了，且合适大部分的 Android 项目的构建，如果遇到不满足的地方，下面也有方法告诉你怎么轻易地解决。

足够的免费配额。GitHub Actions 对于免费的账户也是有一定的限制的，具体表现在：每个仓库的构建产物限制是 500MB，每个月的运行时长是 2000 分钟。对于我们只是构建一个普通的 Android 项目安装包来说，也够用了，运行时长也是绰绰有余。

## 如何使用 GitHub Actions
下面就通过实战来看看如何方便快捷地用 GitHub Actions 来构建一个开源 Android 项目的 APK 安装包吧。我这里选择的是 FolioReader，一个 Java 编写的 ePub 阅读器，在 GitHub 上开源并获得 2k ⭐。

## 流水线

访问 .github/workflows/build.yaml

这个 Workflow 的触发条件设置为手动触发，因为还需要一些修改才能满足目标 Android 项目的构建条件，因此没有设置为常规的基于文件改动就触发。

虚拟环境这里我使用的是 ubuntu-latest，在此时就是指向 Ubuntu 20.04 这个 LTS 版本，日常开发中可能不建议使用这种不确定的版本，但在我们的场景中还是毕竟合适的，因为一个还在维护的 Android 项目一般都会适配较新的 LTS 版本的。

第一步是拉取 android_builder 的源代码，主要目的是获取 project-to-build 这份文件，里面包含了我们的目标 Android 项目的 GitHub 地址。在我们这个实战中就是 https://github.com/FolioReader/FolioReader-Android 这个地址，如需编译其它的项目，把该地址替换为其相应的 GitHub 地址即可。

第二步是设置运行环境，这里是重点。一般情况下，Android 项目中的 Java 代码语法需要一定的 Java 编译器版本，因此我这里引入了 actions/setup-java 这个 action 来快捷地设置 Java 的版本，比如这里我使用了 1.8 版本（Java 8）覆盖环境中自带的 Java 11 版本。同样地，设置 Gradle 和 Android SDK 也有快捷的 action 可以复用，分别为 gradle/gradle-build-action 和 android-actions/setup-android。GitHub 官方的 Ubuntu 20.04 的环境中自带的版本已经是比较高的版本了，一般情况下程序都有后向兼容，所以大部分的情况下你其实可以完全不用设置。这里仅是一个例子来展示如何轻松地修改版本。

第三步的目标是从 project-to-build 这份文件中读取 Android 项目的开源地址并传递给下一步进行拉取Android 项目源码。注意，目标 Android 项目要开源并且是处于公开的状态。cat project-to-build可以读取这份文件包含的地址，然后通过 GitHub Actions 中特殊的语法 ::set-output name=PROJECT::XXX设置地址为该步骤的输出。

第四步是拉取目标 Android 项目源码到虚拟环境中准备编译。首先通过 ${{ steps.get-project.outputs.PROJECT }} 获取上一步的输出地址，然后用 Git 命令克隆 Android 项目源码到虚拟环境的本地中。至此，编译前的准备工作已完成。

第五步是构建 APK 的关键步骤，这里假设目标 Android 项目是已经能够编译通过的了。gradlew 是 Gradle 包管理工具自己产生的一个 bash 脚本，用于命令行环境下的自动构建，绝大部分的开源项目已经包含了该文件，因此我加了个判断，如果不存在该文件则用 Gradle 生成出来，并赋予执行权限。得益于优秀的包管理器，Android 项目下只需要一句命令即可构建出 APK 安装包——./gradlew assembleDebug --stacktrace，该命令用于构建调试版 APK，调试版本已满足个人的使用，折腾应用签名就没有必要了。后面的 stacktrace 参数只是为了显示更多的运行信息。

运行完毕后点开 Workflow 在 Summary 的选项卡中找到 Artifacts，即可下载带 APK 的压缩包。
