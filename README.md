# JLicenseForJavaFileUtil说明文档

- <a href="#项目主要功能介绍">项目主要功能介绍</a>
- <a href="#关键代码介绍">关键代码介绍</a>
- <a href="#使用说明">使用说明</a>
- <a href="#反馈渠道">反馈渠道</a>

##   主要功能介绍

- 批量为某个路径下的所有java文件添加license声明
- license支持文本输入和文件输入两种方式

## 关键代码介绍

* 遍历文件和文件夹，找到需要处理的文件

```java
private static void iterativeHandleFiles(File file, String fileSuffix, FileAction... action) {

        if (file == null || !file.exists() || action == null) {
            return;
        }

        if (file.isDirectory()) {
            File[] files = file.listFiles();
            if (files == null) {
                return;
            }
            for (int i = 0; i < files.length; i++) {
                iterativeHandleFiles(files[i], fileSuffix, action);
            }
        } else {
            String name = file.getName();
            // 不是目标文件则直接返回，不处理
            if (isEmpty(name) || !name.endsWith(FILE_SUFFIX)) {
                return;
            }
            for (int i = 0; i < action.length; i++) {
                action[i].handleFile(file);
            }
        }
    }
```

* 根据license字符串添加声明

```java
 public void handleFile(File file) {

        RandomAccessFile targetRandomAccessFile = null;
        try {
            targetRandomAccessFile = new RandomAccessFile(file, "rw");

            if (targetRandomAccessFile.length() > MAX_FILE_SIZE) {
                System.out.println("file size is too long!" + file.getName());
                return;
            }

            // 读取license文本内容
            byte[] contentBytes = new byte[(int) targetRandomAccessFile.length()];
            targetRandomAccessFile.readFully(contentBytes);
            String contentStr = new String(contentBytes);

            int indexOfPackage = contentStr.indexOf("package");
            // 拼接最终的文件内容
            contentStr = mLicenseStr + "\n" + contentStr.substring(indexOfPackage);
            targetRandomAccessFile.seek(0);
            targetRandomAccessFile.setLength(contentStr.length());
            targetRandomAccessFile.write(contentStr.getBytes("UTF-8"));
        } catch (Exception e) {
            System.out.println(e.getMessage());
        } finally {
            try {
                if (targetRandomAccessFile != null) {
                    targetRandomAccessFile.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

* 根据license文件添加声明

```java
public void handleFile(File file) {

        RandomAccessFile targetRandomAccessFile = null;
        RandomAccessFile licenseRandomAccessFile = null;
        try {
            targetRandomAccessFile = new RandomAccessFile(file, "rw");
            licenseRandomAccessFile = new RandomAccessFile(mLicensePath, "rw");

            if (targetRandomAccessFile.length() > MAX_FILE_SIZE) {
                System.out.println("file size is too long!" + file.getName());
                return;
            }

            // 读取文本内容
            byte[] contentBytes = new byte[(int) targetRandomAccessFile.length()];
            targetRandomAccessFile.readFully(contentBytes);
            String contentStr = new String(contentBytes);

            // 读取license文本内容
            byte[] licenseBytes = new byte[(int) licenseRandomAccessFile.length()];
            licenseRandomAccessFile.readFully(licenseBytes);
            String licenseStr = new String(licenseBytes);

            int indexOfPackage = contentStr.indexOf("package");
            // 拼接最终的文件内容
            contentStr = licenseStr + "\n" + contentStr.substring(indexOfPackage);
            targetRandomAccessFile.seek(0);
            targetRandomAccessFile.setLength(contentStr.length());
            targetRandomAccessFile.write(contentStr.getBytes("UTF-8"));
        } catch (Exception e) {
            System.out.println(e.getMessage());
        } finally {
            try {
                if (targetRandomAccessFile != null) {
                    targetRandomAccessFile.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            if (licenseRandomAccessFile != null) {
                try {
                    licenseRandomAccessFile.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```


## 使用说明

该工程是一个android studio工程，实际上也可以将源码文件拷贝到任何一个java工程中运行。

使用步骤：

* 使用android studio导入工程
* 找到com.jwz.jlicenseforjavafileutil.file.LicenseForJavaFileUtils类
* 找到main方法
* 将main方法中的licenseStr内容替换成自己的license声明
* 将main方法中javaFilesDir路径替换成需要批量添加license的工程root路径
* 执行main方法

## 反馈渠道

- jiangwz604@163.com