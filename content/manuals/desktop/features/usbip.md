---
title: 在 Docker Desktop 中使用 USB/IP
linkTitle: USB/IP 支持
weight: 50
description: 如何在 Docker Desktop 中使用 USB/IP
keywords: usb, usbip, docker desktop, macos, windows, linux
toc_max: 3
aliases:
- /desktop/usbip/
---

{{< summary-bar feature_name="USB/IP support" >}}

USB/IP 允许通过网络共享 USB 设备，并在 Docker 容器内进行访问。本文聚焦于共享连接在运行 Docker Desktop 的本机上的 USB 设备。你可以根据需要重复以下步骤，附加并使用更多的 USB 设备。

> [!NOTE]
>
> Docker Desktop 内置了许多常见 USB 设备的驱动，但无法保证所有 USB 设备都能在此方案下正常工作。

## 设置与使用

### 第一步：运行 USB/IP 服务器

要使用 USB/IP，需要运行一个 USB/IP 服务器。本文示例使用 [jiegec/usbip](https://github.com/jiegec/usbip) 提供的实现。

1. Clone the repository.

    ```console
    $ git clone https://github.com/jiegec/usbip
    $ cd usbip
    ```

2. 运行模拟的人机接口设备（HID）示例。

    ```console
    $ env RUST_LOG=info cargo run --example hid_keyboard
    ```

### 第二步：启动特权容器

要附加 USB 设备，请启动一个特权容器，并将 PID 命名空间设置为 `host`：

```console
$ docker run --rm -it --privileged --pid=host alpine
```

`--privileged` 赋予容器对主机的完全访问权限，`--pid=host` 允许容器共享主机的进程命名空间。

### 第三步：进入 PID 1 的挂载命名空间

进入容器后，切换到 `init` 进程的挂载命名空间，以使用预装的 USB/IP 工具：

```console
$ nsenter -t 1 -m
```

### 第四步：使用 USB/IP 工具

现在即可像在其他系统上一样使用 USB/IP 工具：

#### 列出 USB 设备

列出主机上可导出的 USB 设备：

```console
$ usbip list -r host.docker.internal
```

预期输出：

```console
Exportable USB devices
======================
 - host.docker.internal
      0-0-0: unknown vendor : unknown product (0000:0000)
           : /sys/bus/0/0/0
           : (Defined at Interface level) (00/00/00)
           :  0 - unknown class / unknown subclass / unknown protocol (03/00/00)
```

#### 附加 USB 设备

附加指定的 USB 设备（此处为模拟的键盘）：

```console
$ usbip attach -r host.docker.internal -d 0-0-0
```

#### 验证设备已附加

附加模拟键盘后，检查 `/dev/input` 目录中的设备节点：

```console
$ ls /dev/input/
```

示例输出：

```console
event0  mice
```

### 第五步：从其他容器访问该设备

在保持初始容器运行以维持 USB 设备可用的同时，你可以在另一个容器中访问该设备。例如：

1. 启动一个包含该设备的新容器。

    ```console
    $ docker run --rm -it --device "/dev/input/event0" alpine
    ```

2. 安装 `evtest` 等工具以测试模拟的键盘。

    ```console
    $ apk add evtest
    $ evtest /dev/input/event0
    ```

3. 与设备交互，并观察输出。

    示例输出：

    ```console
    Input driver version is 1.0.1
    Input device ID: bus 0x3 vendor 0x0 product 0x0 version 0x111
    ...
    Properties:
    Testing ... (interrupt to exit)
    Event: time 1717575532.881540, type 4 (EV_MSC), code 4 (MSC_SCAN), value 7001e
    Event: time 1717575532.881540, type 1 (EV_KEY), code 2 (KEY_1), value 1
    Event: time 1717575532.881540, -------------- SYN_REPORT ------------
    ...
    ```

> [!IMPORTANT]
>
> 初始容器必须保持运行，以维持与 USB 设备的连接。退出该容器会导致设备停止工作。
