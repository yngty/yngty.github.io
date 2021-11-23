---
title: 文件MIME类型
date: 2021-11-23 20:14:31
tags:
- Linux
- mime
categories:
- Linux
---

# 什么是 MIME 类型？

`MIME`（多用途 `Internet` 邮件扩展）的类型来识别文件格式。 `MIME` 类型构成了 `Internet` 上对文件类型进行分类的标准方法。

- `MIME Type`是用于描述文件的类型的一种表述方法，其将文件划分为多种类型，方便对其进行统一的管理。
- `MIME Type`指定了文件的类型名称、描述、图标信息，同时通过与.desktop应用程序描述文件整合，指定了文件的打开方式。

<!--more-->
`MIME` 类型名字遵循指定的格式：

类型和子类型， 在 MIME 类型中，类型和子类型不区分大小写。

```
media-type/subtype-identifier
```

目前，有十种注册类型：`application`，`audio`，`example`，`font`，`image`，`message`，`model`，`multipart`，`text`和`video`。

例如：

```
multipart/form-data
text/xml
text/csv
text/plain
application/xml
application/zip
application/pdf
```

完整MIME 类型示例：

```
application/vnd.api+json
```

`application`作为类型，`api`作为子类型，`vnd`是厂商前缀，`+json`是后缀，表示可以解析为`JSON`。

# 获取文件的 MIME 类型

## xdg-mime命令

- 显示文件的`MIME`类型：
    ```shell
    xdg-mime query filetype {file}
    ```
    例如：

    ```shell
    ➜ xdg-mime query filetype one.jpg 
    image/jpeg
    ```

- 显示`MIME` 类型的默认应用程序

    ```shell
    xdg-mime query default {mimetype}
    ```
    例如：
    ```shell
    ➜ xdg-mime query default image/jpeg
    /usr/share/applications/deepin-image-viewer.desktop

    ```
- 显示文件默认应用程序的语法

    ```shell
    xdg-mime query default "$(xdg-mime query filetype {file})"
    ```

    例如：
    ```shell
    xdg-mime query default \
        `xdg-mime query filetype "$(find ~ / -iname '*.png' -print -quit)"`
    ```

- 设置`MIME` 类型的默认打开应用程序

    ```shell
    xdg-mime default dekstop filetype
    ```

    例如：

    ```shell
    xdg-mime default dde-file-manager.desktop inode/directtory
    ```

## file 命令

- 查询文件类型：

    ```
    file --mime-type INPUT_FILE
    ```

    例如：
    ```
    ➜ file --mime-type one.jpg 
    one.jpg: inode/symlink
    ```


# 自定义的 MIME 类型

如需为系统上的所有用户添加一个自定义的 `MIME` 类型，并为该 `MIME` 类型注册一个默认的应用程序，您需要在 `/usr/share/mime/packages/` 目录下创建一个新的 `MIME` 类型说明文件，在 `/usr/share/applications/` 目录下创建一个 `.desktop` 文件。

比如我们创建一个`application/x-newtype`类型：

1. 创建 /usr/share/mime/packages/application-x-newtype.xml 文件

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
    <mime-type type="application/x-newtype">
        <comment>new mime type</comment>
        <glob pattern="*.xyz"/>
    </mime-type>
    </mime-info>
    ```
    上述 `application-x-newtype.xml` 文件定义了一种新的 `MIME` 类型`application/x-newtype`，并指定拓展名是 `.xyz` 的文件为该 `MIME` 类型。

2. 创建一个名为例如 `myapplication1.desktop` 的新的 `.desktop` 文件，并将它放置在 `/usr/share/applications/` 目录下：

    ```
    [Desktop Entry]
    Type=Application
    MimeType=application/x-newtype
    Name=My Application 1
    Exec=myapplication1
    ```

3. 请以 `root` 身份更新 `MIME` 数据库以使您的更改生效：

    ```shell
    ➜ update-mime-database /usr/share/mime
    ```

4. 请以 `root` 身份更新应用程序数据库：
    ```shell
    ➜ update-desktop-database /usr/share/applications
    ```

如需为个别用户添加自定义的 `MIME` 类型，并为该`MIME` 类型注册一个默认的应用程序，您需要在 `~/.local/share/mime/packages/` 目录下创建一个新的 `MIME` 类型说明文件，并在 `~/.local/share/applications/` 目录下创建一个 `.desktop` 文件。


# 参考资料

- [配置文件关联](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/desktop_migration_and_administration_guide/file_formats)

- [file-mime-types](https://www.baeldung.com/linux/file-mime-types)

- [mime-apps-spec](https://specifications.freedesktop.org/mime-apps-spec/mime-apps-spec-latest.html)