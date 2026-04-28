# 1. 下载
下载并解压 [Windows Terminal](https://github.com/microsoft/terminal/releases) zip到`C:\SSS\terminal\`。

# 2. 固定到磁贴

## 2.1 配置JSON

![图片](https://github.com/user-attachments/assets/fa93f2e0-e0dc-4c05-861f-7e62c5aa63df)

点击 打开JSON文件，添加：

```json
{
    "profiles": {
        "defaults": {
            "startingDirectory": null,
        }
    }
}
```

![图片](https://github.com/user-attachments/assets/3f2581aa-f557-4724-a757-4c9001b700bc)

## 2.2 添加快捷方式和磁贴

在 `shell:startup`(`C:\ProgramData\Microsoft\Windows\Start Menu\Programs`) 创建快捷方式 `C:\SSS\terminal\WindowsTerminal.exe -d "~"`。（其中 `-d "~"` 表示启动后切换到用户主目录。）

![图片](https://github.com/user-attachments/assets/b4f08151-7c83-4af8-8e04-238e66c31b5b)


# 3. 添加右键菜单项

使用[ContextMenuManager](https://github.com/BluePointLilac/ContextMenuManager)添加右键菜单 `Open Terminal here`。注意 命令参数 为空。

![图片](https://github.com/user-attachments/assets/c584328a-29ee-4c81-95e1-cad005810b7d)

![图片](https://github.com/user-attachments/assets/9aaa435e-59b2-4bcc-9f3a-b04d02e40339)

