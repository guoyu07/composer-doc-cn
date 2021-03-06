<!--
    tagline: Expose command-line scripts from packages
-->

# 二进制供应库和 `vendor/bin` 目录

## 什么是二进制供应库？

Any command line script that a Composer package would like to pass along
to a user who installs the package should be listed as a vendor binary.

一个 Composer 资源包，想要传递给安装它的用户的任何命令行脚本，
都应该被列为 `二进制供应库`。

If a package contains other scripts that are not needed by the package
users (like build or compile scripts) that code should not be listed
as a vendor binary.

若一个资源包包含有不被用户所需要的其他脚本（比如 build 或 编译脚本）
那么这些代码不应该被列为二进制供应库。


## 如何定义？

It is defined by adding the `bin` key to a project's `composer.json`.
It is specified as an array of files so multiple binaries can be added
for any given project.

它是通过在项目的 `composer.json` 里添加一个 `bin` 键定义的。
它是以一种文件的数组的形式定义的，这样任意给定项目
都可以添加多个二进制文件。

```json
{
    "bin": ["bin/my-script", "bin/my-other-script"]
}
```

## 在 composer.json 里定义二进制供应库的作用是？

It instructs Composer to install the package's binaries to `vendor/bin`
for any project that **depends** on that project.

他指导 Composer 如何安装资源包里的二进制文件到 `vendor/bin`，
以方便所有**依赖于**该项目的项目引用。

This is a convenient way to expose useful scripts that would
otherwise be hidden deep in the `vendor/` directory.

这是一个不错的，只暴露有用的脚本，同时把其他用不上的脚本深深地隐藏在 `vendor/` 目录里的
方法。

## 当 Composer 运行于定义了二进制供应库的 composer.json 时发生了什么？

For the binaries that a package defines directly, nothing happens.

对于被某个包直接定义的二进制库，什么也不会发生。


## 当 Composer 运行于标明依赖于某二进制供应库的 composer.json 时发生了什么？

Composer looks for the binaries defined in all of the dependencies. A
symlink is created from each dependency's binaries to `vendor/bin`.

Composer会检查所有依赖库里定义的二进制文件。
并为每一个依赖的二进制库设立一个指向 `vendor/bin` 的软连接。

Say package `my-vendor/project-a` has binaries setup like this:

比如 `my-vendor/project-a` 资源包的二进制库就是这样安装的：

```json
{
    "name": "my-vendor/project-a",
    "bin": ["bin/project-a-bin"]
}
```

Running `composer install` for this `composer.json` will not do
anything with `bin/project-a-bin`.

在该 `composer.json` 上执行 `composer install` 命令，
不会对 `bin/project-a-bin` 造成任何影响。

Say project `my-vendor/project-b` has requirements setup like this:

但是如果 `my-vendor/project-b` 项目定义有这样的需求：


```json
{
    "name": "my-vendor/project-b",
    "require": {
        "my-vendor/project-a": "*"
    }
}
```

Running `composer install` for this `composer.json` will look at
all of project-b's dependencies and install them to `vendor/bin`.

在该 `composer.json` 上执行 `composer install` 命令时，
会检查 project-b 的所有依赖，并把它们中的二进制库安装到 `vendor/bin`。

In this case, Composer will make `vendor/my-vendor/project-a/bin/project-a-bin`
available as `vendor/bin/project-a-bin`. On a Unix-like platform
this is accomplished by creating a symlink.

这种情况下，Composer 会允许以 `vendor/bin/project-a-bin` 格式访问
`vendor/my-vendor/project-a/bin/project-a-bin`。在类-Unix 的平台上，
这是通过创建 symlink 软连接实现的。


## 对于 Windows 环境和 .bat 文件呢？

Packages managed entirely by Composer do not *need* to contain any
`.bat` files for Windows compatibility. Composer handles installation
of binaries in a special way when run in a Windows environment:

完全由 Composer 管理的包并不*需要*包含任何用以兼容 Windows 的
`.bat`文件。在 Windows 环境下运行时，Composer 会用一种特殊的方式处理
二进制文件的安装：

 * A `.bat` file is generated automatically to reference the binary
 * A Unix-style proxy file with the same name as the binary is generated
   automatically (useful for Cygwin or Git Bash)

 * 一个用以引用此二进制文件的 `.bat` 文件会自动生成
 * 一个与该二进制文件同名的 Unix-风格的代理文件也会自动生成
 （方便 Cygwin 或 Git Bash使用）

Packages that need to support workflows that may not include Composer
are welcome to maintain custom `.bat` files. In this case, the package
should **not** list the `.bat` file as a binary as it is not needed.

若某包存在不涉及 Composer 的工作流程，
那么它也可以维护一些定制的 `.bat` 文件。这种情况下，该包
**不**应该把它们作为二进制文件罗列，因为 Composer 不需要知道它们。


## 二进制供应库可以安装在不是 vendor/bin 的地方么？

当然，这里有两种指定二进制供应库的其他可选位置的方法：

 1. 在 `composer.json` 文件中的 `bin-dir` 配置属性处设置
 1. 设置环境变量 `COMPOSER_BIN_DIR`

前者的案例如下：

```json
{
    "config": {
        "bin-dir": "scripts"
    }
}
```

Running `composer install` for this `composer.json` will result in
all of the vendor binaries being installed in `scripts/` instead of
`vendor/bin/`.

在这个 `composer.json` 上运行 `composer install` 会把所有的二进制供应库
都安装在 `scripts/` 目录，而不是缺省的 `vendor/bin/` 目录。
