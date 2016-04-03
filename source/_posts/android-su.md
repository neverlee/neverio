title: android执行su命令
categories: 小笔记
date: 2016-04-03 18:59:34
tags: [android,root,su]
description: android使用su运行带root权限的命令
---

Root过后的Android使用su执行命令的java代码。

## 修改包目录权限
```java
public boolean upgradeRootPermission() {
    Process process = null;
    DataOutputStream os = null;
    String cmd="chmod 777 " + getPackageCodePath();
    try {
        process = Runtime.getRuntime().exec("su"); //切换到root帐号
        os = new DataOutputStream(process.getOutputStream());
        os.writeBytes(cmd + "\n");
        os.writeBytes("exit\n");
        os.flush();
        process.waitFor();
    } catch (Exception e) {
        return false;
    } finally {
        try {
            if (os != null) {
                os.close();
            }
            process.destroy();
        } catch (Exception e) {
        }
    }
    return true;
}

```

## 以root权限执行并获取输出
```java
public String execRootCmd(String cmd) {
    DataInputStream is = null;
    DataOutputStream os = null;
    String content = "";
    try {
        Process process = Runtime.getRuntime().exec("su");
        os = new DataOutputStream(process.getOutputStream());
        is = new DataInputStream(process.getInputStream());
        os.writeBytes(cmd + "\nexit\n");
        os.flush();
        while (true) {
            String line = is.readLine();
            if (line == null) {
                break;
            }
            content += line + "\n";
        }
        //process.waitFor();
        //return process.exitValue();
        return content;
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (is != null) {
                is.close();
            }
            if (os != null) {
                os.close();
            }
            process.destroy();
        } catch (Exception e) {
        }
    }
    return "";
}
```

> 作者原创，转载请注明出处
