---
title: Express文件上传-Multer

author: mmy83
date: 2025-05-14 19:22:00 +0800
categories: [框架, express]
tags: [express, 框架, multer, 上传]
math: true
mermaid: true
image:
  path: /images/2025/05/2025-05-14/Express文件上传-Multer/Express文件上传-Multer-00.jpeg
  lqip: data:image/webp;base64,UklGRkYAAABXRUJQVlA4IDoAAAAwAgCdASoIAAQAAUAmJagCdLoB+AADE3gQAADhXcvEhi/FKc5qP/ZWPqWfpWUt+Sr/5+Z3bi/nMAAA
  alt: Express文件上传-Multer
---

## 介绍

multer 是一个 Node.js 的中间件，用于处理 multipart/form-data 类型的表单数据，这在文件上传场景中非常常见。它基于 busboy 库构建，可以轻松地与 express 框架集成。

## 安装

```shell
npm install multer
```

## 使用实例

```javascript
const express = require("express");
 
const multer = require("multer");
 
const app = express();
 
// 配置 multer
const storage = multer.diskStorage({
 
  destination: function (req, file, cb) {
 
    cb(null, "uploads/");
 
  },
 
  filename: function (req, file, cb) {
 
    cb(null, file.originalname);
 
  },
 
});
 
const upload = multer({ storage: storage });
 
// 处理文件上传的路由
app.post("/upload", upload.single("file"), (req, res) => {
 
  res.send("文件上传成功");
 
});
 
const port = 3000;
 
app.listen(port, () => {
 
  console.log(`服务器运行在端口 ${port}`);
 
});
```

## 说明

### 存储（storage）

`storage` 选项用于指定文件的存储方式，`multer` 提供了两种存储引擎：

+ **diskStorag**e**：将文件存储到磁盘上。你可以通过 `destination` 和 `filename` 函数来指定文件的存储目录和文件名。

```javascript
const storage = multer.diskStorage({
 
  destination: function (req, file, cb) {
 
    cb(null, "uploads/"); // 指定存储目录
 
  },
 
  filename: function (req, file, cb) {
 
    cb(null, Date.now() + "-" + file.originalname); // 指定文件名
 
  },
 
});
```

+ **memoryStorage**：将文件存储在内存中，以 `Buffer` 对象的形式存在。适用于需要对文件进行进一步处理（如上传到云存储）而不需要将文件保存到本地磁盘的场景。

```javascript
const storage = multer.memoryStorage();
 
const upload = multer({ storage: storage });
```

### 限制（limits）

`limits` 选项用于限制上传文件的大小、字段数量等。常见的限制选项包括：

+ `fileSize`：文件的最大大小（以字节为单位）。

+ `files`：允许上传的文件数量。

+ `fields`：允许的表单字段数量。

```javascript
const upload = multer({
 
  storage: storage,
 
  limits: {
 
    fileSize: 1024 * 1024 * 5, // 限制文件大小为 5MB
 
  },
 
});
```

### 文件过滤（fileFilter）

`fileFilter` 选项用于过滤允许上传的文件类型。你可以通过回调函数来决定是否接受某个文件。

```javascript
const fileFilter = function (req, file, cb) {
 
  // 只允许上传图片文件
  if (file.mimetype.startsWith("image/")) {
 
    cb(null, true);
 
  } else {
 
    cb(new Error("只允许上传图片文件"), false);
 
  }
 
};
 
const upload = multer({
 
  storage: storage,
 
  fileFilter: fileFilter,
 
});
```

## 上传

### 单文件

使用 `upload.single(fieldname)` 处理单个文件上传，其中 `fieldname` 是表单中文件字段的名称。

```javascript
app.post("/upload", upload.single("file"), (req, res) => {
 
  // req.file 包含上传的文件信息
  console.log(req.file);
 
  res.send("文件上传成功");
 
});
```

### 多个文件上传（单字段）

使用 `upload.array(fieldname, maxCount)` 处理多个文件上传，`maxCount` 是允许上传的最大文件数量。

```javascript
app.post("/upload-multiple", upload.array("files", 3), (req, res) => {
 
  // req.files 是一个包含多个文件信息的数组
  console.log(req.files);
 
  res.send("多个文件上传成功");
 
});
```

### 多个文件上传（多字段）

使用 `upload.fields(fields)` 处理包含多个文件字段的表单，`fields` 是一个包含每个字段名称和最大文件数量的数组。

```javascript
app.post(

  "/upload-mixed",
 
  upload.fields([
 
    { name: "avatar", maxCount: 1 },
 
    { name: "photos", maxCount: 3 },
 
  ]),
 
  (req, res) => {
 
    // req.files 是一个对象，包含每个字段的文件信息
    console.log(req.files);
 
    res.send("混合文件上传成功");
 
  }

);

```

## 上传文件信息

当文件上传成功后，`multer` 会将文件的相关信息添加到 `req.file`（单个文件上传）或 `req.files`（多个文件上传）中。常见的文件信息包括：

+ `fieldname`：表单中文件字段的名称。

+ `originalname`：文件的原始名称。

+ `encoding`：文件的编码类型。

+ `mimetype`：文件的 MIME 类型。

+ `size`：文件的大小（以字节为单位）。

+ `destination`：文件的存储目录（使用 `diskStorage` 时）。

+ `filename`：文件在存储目录中的名称（使用 `diskStorage` 时）。

+ `path`：文件的完整路径（使用 `diskStorage` 时）。

+ `buffer`：文件的二进制数据（使用 `memoryStorage` 时）。

## 处理错误

### 直接判断

在上传文件的过程中，可能会发生错误，如文件大小超过限制、文件类型不被允许等。你可以通过监听 error 事件来处理这些错误。如果 req.file 不存在，但存在 req.err，则表示上传过程中出现了错误。

```javascript
app.post('/upload', upload.single('file'), (req, res) => {
  if (req.file) {
    // 文件上传成功
    res.send(`File uploaded successfully: ${req.file.filename}`);
  } else if (req.err) {
    // 处理错误
    res.status(500).send('File upload failed');
  }
});
```

### 错误处理中间件

可以通过编写错误处理中间件来捕获multer处理文件上传过程中发生的错误。这可以通过在Express应用中使用app.use方法来注册一个中间件函数，并将其放在multer中间件之后

```javascript
app.use(function(err, req, res, next) {
  if (err instanceof multer.MulterError) {
    // Multer发生错误，可以根据err.code进行不同的处理
    if (err.code === 'LIMIT_FILE_SIZE') {
      // 处理文件大小超出限制的情况
      return res.status(400).send('文件大小超出限制');
    }
    // 其他Multer错误的处理
    // ...
  } else if (err) {
    // 其他类型的错误处理
    console.error(err);
    return res.status(500).send('发生了一个错误');
  }
  next();
});
```

### 路由处理函数中手动捕获错误

可以在Express应用的路由处理函数中手动捕获multer处理文件上传过程中发生的错误，并进行相应的处理。例如：

```javascript
app.post('/upload', function(req, res, next) {
  upload.single('file')(req, res, function(err) {
    if (err instanceof multer.MulterError) {
      // Multer发生错误，可以根据err.code进行不同的处理
      if (err.code === 'LIMIT_FILE_SIZE') {
        // 处理文件大小超出限制的情况
        return res.status(400).send('文件大小超出限制');
      }
      // 其他Multer错误的处理
      // ...
    } else if (err) {
      // 其他类型的错误处理
      console.error(err);
      return res.status(500).send('发生了一个错误');
    }
    // 文件上传成功，继续处理其他逻辑
    // ...
  });
});
```

## 参考资料

1. [multer 依赖详解](https://blog.csdn.net/weixin_64684095/article/details/145939592)

2. [express里面的文件上传功能之multer使用总结](https://blog.csdn.net/qq_27702739/article/details/137561655)

3. [在multer中处理错误？](https://cloud.tencent.com/developer/information/%E5%9C%A8multer%E4%B8%AD%E5%A4%84%E7%90%86%E9%94%99%E8%AF%AF%EF%BC%9F)

4. [multer](https://blog.csdn.net/weixin_46901555/article/details/122372169)

5. [《Node.js 文件上传神器：Multer ✨》](https://juejin.cn/post/7435137830257737739)
