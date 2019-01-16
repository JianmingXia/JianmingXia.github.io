---
title: Archiver zip打包
date: 2019/01/14 21:00:00
tags:
  - 压缩打包
categories: NodeJS
---

## 说在前面
由于现有的上传下载服务实现不合理——在内存中暂存完整内容。只要并发稍微高一点，内存占用就疯狂报警；除此之外，还伴随着段错误，服务重构迫在眉睫。<br />NodeJS 中有个zlib模块提供压缩功能，但是没有找到支持多文件压缩的文档。最后找到了archiver模块，支持zip及tar格式的压缩，使用起来也很简单。

## Archiver 介绍
### 基础事件
#### close
> listen for all archive data to be written
> 'close' event is fired only when a file descriptor is involved

<!-- more -->

监听要写入的所有存档数据，当涉及到文件描述符时事件会触发——一般来说，当可写数据完毕后，close事件会被触发

#### end
> This event is fired when the data source is drained no matter what was the data source.
> It is not part of this library but rather from the NodeJS Stream API.
> @see: https://nodejs.org/api/stream.html#stream_event_end


这个事件很奇怪，在NodeJS可写流相关文档中，没有找到end事件相关的文档；同时在end event上方的说明中，有一个[文档](https://nodejs.org/api/stream.html#stream_event_end)——在一个可读流中，如果没有更多的数据流出，end事件将会被触发。

这里突然想到，如果在压缩时，有一个转换流——当数据流完之后，end 事件会被触发<br />PS：[https://github.com/JianmingXia/StudyTest/blob/master/Archiver/endEventFired.js](https://github.com/JianmingXia/StudyTest/blob/master/Archiver/endEventFired.js)

### Archiver 事件
#### error
> [https://archiverjs.com/docs/lib_core.js.html](https://archiverjs.com/docs/lib_core.js.html)
> 在此文档搜索ArchiverError即可，可以发现有很多触发error事件的情况


```
/**
 * Error listener that re-emits error on to our internal stream.
 *
 * @private
 * @param  {Error} err
 * @return void
 */
Archiver.prototype._onModuleError = function(err) {
  /**
   * @event Archiver#error
   * @type {ErrorData}
   */
  this.emit('error', err);
};
```

比如调用finalize方法，依然调用append方法，此时error方法会被触发：

```
archive.append("", { name: "sdffsdfsd/" });

archive.finalize();

archive.append("", { name: "sdffsdfsd/" });
```

PS：[https://github.com/JianmingXia/StudyTest/blob/master/Archiver/errorEventFired.js](https://github.com/JianmingXia/StudyTest/blob/master/Archiver/errorEventFired.js)

#### warning
> [https://archiverjs.com/docs/lib_core.js.html](https://archiverjs.com/docs/lib_core.js.html)
> 在此文档搜索ArchiverError即可，可以发现有很多触发warning事件的情况


#### entry
每个正常被append的item都会触发<br />PS：[https://github.com/JianmingXia/StudyTest/blob/master/Archiver/entryAndProgressEvent.js](https://github.com/JianmingXia/StudyTest/blob/master/Archiver/entryAndProgressEvent.js)

```
    /**
     * Fires when the entry's input has been processed and appended to the archive.
     *
     * @event Archiver#entry
     * @type {EntryData}
     */
    this.emit('entry', data);
    this._entriesProcessedCount++;
    if (data.stats && data.stats.size) {
      this._fsEntriesProcessedBytes += data.stats.size;
    }
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547520254405-107f5672-e267-48b0-9405-e574ed658e54.png#align=left&display=inline&height=415&linkTarget=_blank&name=image.png&originHeight=415&originWidth=1391&size=43881&width=1391)

#### progress
每个正常被append的item都会触发<br />PS：[https://github.com/JianmingXia/StudyTest/blob/master/Archiver/entryAndProgressEvent.js](https://github.com/JianmingXia/StudyTest/blob/master/Archiver/entryAndProgressEvent.js)

```
    /**
     * @event Archiver#progress
     * @type {ProgressData}
     */
    this.emit('progress', {
      entries: {
        total: this._entriesCount,
        processed: this._entriesProcessedCount
      },
      fs: {
        totalBytes: this._fsEntriesTotalBytes,
        processedBytes: this._fsEntriesProcessedBytes
      }
    });
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547520322139-ae96efbc-a2fa-4c35-9d26-4cff891c4ab8.png#align=left&display=inline&height=511&linkTarget=_blank&name=image.png&originHeight=511&originWidth=1417&size=50451&width=1417)

### Zip 配置

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547534871016-ef79c8ce-d7a7-4fdb-b4bf-26a2c856636d.png#align=left&display=inline&height=477&linkTarget=_blank&name=image.png&originHeight=477&originWidth=1378&size=51152&width=1378)

#### zlib

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547534963703-d6761a3e-4770-4b33-9bd9-a18130548da6.png#align=left&display=inline&height=298&linkTarget=_blank&name=image.png&originHeight=298&originWidth=876&size=38297&width=876)

比如 level 可配置项：
```
zlib.constants.Z_NO_COMPRESSION
zlib.constants.Z_BEST_SPEED
zlib.constants.Z_BEST_COMPRESSION
zlib.constants.Z_DEFAULT_COMPRESSION
```

更多参数可见文档。

#### forceLocalTime

```
    const archive = archiver('zip', {
      zlib: { level: ZLIB_LEVEL.Z_BEST_COMPRESSION },
      forceLocalTime: true,
    });
```

forceLocalTime 设置为true，即可解决下面的问题：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547642132394-3e43b450-594f-4416-b7d8-0246f9a13bf2.png#align=left&display=inline&height=201&linkTarget=_blank&name=image.png&originHeight=201&originWidth=922&size=30061&width=922)

## Demo
### append
#### 追加一个文件流

```
archive.append(fs.createReadStream(__dirname + "/sourceFiles/1.txt"), { name: "file1.txt" });
```

#### 追加一个string

```
archive.append("string cheese!", { name: "file2.txt" });
```

#### 追加一个buffer

```
var buffer = Buffer.from("buff it!");
archive.append(buffer, { name: "file3.txt" });
```

#### 新增空文件夹

```
archive.append("", { name: "empty-dir/" });
```

### file
使用file追加一个文件

```
archive.file("sourceFiles/1.txt", { name: "file4.txt" });
```

### directory
以下三种方式比较类似，只是会有微小差别
#### 追加一个目录（子目录也会被增加）

```
archive.directory("sourceFiles/", "new-subdir");
```

#### 追加一个目录——类似于直接复制

```
archive.directory("sourceFiles/");
```

#### 将目标目录中的文件追加至根目录

```
archive.directory("sourceFiles/", false);
```

### glob
使用通配符匹配

```
archive.glob("*.js");
```

### 完整代码
> [https://github.com/JianmingXia/StudyTest/tree/master/Archiver](https://github.com/JianmingXia/StudyTest/tree/master/Archiver)


```
var fs = require("fs");
var archiver = require("archiver");

var output = fs.createWriteStream(__dirname + "/example.zip");
var archive = archiver("zip", {
  zlib: { level: 9 },
  comment: "testtest"
});

output.on("close", function() {
  console.log(archive.pointer() + " total bytes");
  console.log(
    "archiver has been finalized and the output file descriptor has closed."
  );
});

archive.on("warning", function(err) {
  console.log(err);
});

// good practice to catch this error explicitly
archive.on("error", function(err) {
  console.log(err);
});

archive.pipe(output);

// 追加一个文件流
archive.append(fs.createReadStream(__dirname + "/sourceFiles/1.txt"), { name: "file1.txt" });

// 追加一个string
archive.append("string cheese!", { name: "file2.txt" });

// 追加一个buffer
var buffer = Buffer.from("buff it!");
archive.append(buffer, { name: "file3.txt" });

// 追加一个现有文件
archive.file("sourceFiles/1.txt", { name: "file4.txt" });

// 追加一个目录（子目录也会被增加）
archive.directory("sourceFiles/", "new-subdir");

// 追加一个目录——类似于直接复制
archive.directory("sourceFiles/");

// 将目标目录中的文件追加至根目录
archive.directory("sourceFiles/", false);

// 新增空文件夹
archive.append('', { name: "empty-dir/" });

// 使用通配符
archive.glob("*.js");

archive.finalize();
```

#### 打包后的目录结构

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547537448379-b2d02a16-ba2e-4acf-b013-76949b6338b7.png#align=left&display=inline&height=261&linkTarget=_blank&name=image.png&originHeight=261&originWidth=1217&size=19846&width=1217)

## 资料
* [https://github.com/archiverjs/node-archiver](https://github.com/archiverjs/node-archiver)
* [https://archiverjs.com/docs/](https://archiverjs.com/docs/)
* [https://nodejs.org/api/zlib.html#zlib_class_options](https://nodejs.org/api/zlib.html#zlib_class_options)
