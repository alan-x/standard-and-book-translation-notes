### File API

### 概述
这个一个提供了一个表示网页应用文件对象的 API，也以编程方式选择他们并访问他们的数据。这包括：
- 一个 FileList 接口，它表示一个从底层系统选中的独立文件数组。选择的用户接口可以通过 <input type="file"> 调用，比如，当 input 元素在文件上传状态。
- 一个 Blob 接口，它表示不可变的原始二进制数据，并且允许将 Blob 中区间的数据作为分离的 Blob 对象访问。
- 一个 File 接口，包含关于一个文件只读的信息属性，比如它的名字和文件最后的修改日期（在磁盘上）。
- 一个 FileReader 接口，提供方法去读取一个 File 或者一个 Blob，并且一个事件模型获取了这些读取的结果。

- 一个使用二进制数据，比如文件的 URL 方案，这样他们可以在网页应用中引用他们。

此外，这个规格定义了用于线程内网页应用同步读取文件的对象。

章节 10 需求和用例覆盖这个规格背后的动机。

这个 API 设计用于结合其他网页平台的 API 和元素，尤其：
XMLHttpRequest(比如，超载的 send() 方法使用 File 或者 Blob 参数)，postMessage()，DataTransfer（定义在 HTML 中的部分拖拽 API）和 Web Workers。此外，它应该能够以编程方式的从 <input> 元素获取一个列表的文件，当它在文件上传状态的时候。这些类型的行为定义在适当的隶属的规格。

### 这个文档的状态

### 1. 介绍
这个章节是提供信息的。

网页应用应该有尽可能多个能力能力去操作用户输入，包括用户可能希望上传到远程服务的文件或者操作内部的富网页应用。这个规格定义了文件的基本表示，文件列表，访问文件时的错误出现，和编程方式读取文件。此外，这个规格也定义了一个接口，它比哦啊是“原始数据”，可以在遵从用户代理的主线程被异步处理。定义在这个规格的接口和 API 可以i 和其他暴露在网页平台的的接口和 API一起使用。

FIle 接口表示文件数据，通常从底层文件系统获取来的， Blob 接口（“Binary Large Object” - 一个最初在 Google Gears 引入的名字）表示不可变数据。File 或者 Blob 读取应该在主线程异步发生，还有一个可选的同步 API 在线程网页应用中使用。一个读取文件异步的 API 阻止用户代理的主线程堵塞和 UI “冻结”。这个规格定义了一个异步的 API，基于事件模型去访问 File 和 Blob 数据。一个 FileReader 对象提供异步读取方法去访问这些文件的数据，通过事件处理器内容属性和事件触发。事件和事件处理器的时候允许分离代码块，有能力去监控读取的进度（这对于远程驱动或者挂载的驱动很有用，因为文件访问表现可能和本地驱动有差异）和读取一个文件的过程中可能出现的错误条件。一个栗子将会描绘。

### 2. 术语和算法
当这个规格说终止一个算法，用户代理必须在完成当前步骤之后终止算法。这个规格定义的异步读取方法可能因为有问题而终止之前返回，可以通过调用 abort 终止。

这个规格中的算法和步骤使用下面的数学操作：
- max(a, b) 返回 a 和 b 的最大值，并且总是在整形上使用，就像他们定义在 Web IDL；例如 max(6, 4) 返回 6。这个操作也定义在 ECMAScript。
- min(a, b) 返回 a 和 b 的最小值，并且总是在整形上执行，就像他们定义在 WebIDL；比如 min(6, 4) 返回 4。这个操作也定义在 ECMAScript。
- 数学比较，比如 <（小于），<=（小于等于），和 >（大于）和 ECMAScript 中一样。

术语 Unix Epoch 在这个章节中用来引用 1970 年 1 月 1 日 00:00:00 UTC（或者 1970-01-01T00:00:00Z ISO 8601）；这和 ECMA Script 中概念为“0”的时间一样。

```
function startRead(){
    var file = document.getElementById('file').files[0];
    if(file){
        getAsText(file);
    }
}

function getAsText(readFile){
    var reader = new FileReader();

    // 用 UTF-16 编码将数据读取进入内存
    reader.readAsText(readFile, "UTF-16");

    // 处理进度，成功，和错误
    reader.onprogress = updateProgress;
    reader.onload = loaded;
    reader.onerror = errorHandler;
}

function updateProgress(evt){
    if(evt.lengthComputable){
        // evt.loaded 和 evt.total 是 ProgressEvent 的属性
        var loaded = (evt.loaded / evt.total);
        if(loaded < 1){
            // 增加进度条的长度
            // style.width = (loaded * 200) + "px";
        }
    }
}

function loaded(evt) {
    // 获取读取的文件数据
    var fileString = evt.target.result;
    // 处理 UTF-16 文件转存
    if(utils.regxp.isChinest(fileString)){
        // 中文字符 + 名字校验
    }
    else {
        // 执行其他字符集测试
    }
    // xhr.send(fileString)
}

function errorHandler(evt){
    if(evt.target.error.name == "NotReadableError"){
        // 文件无法读取
    }
}

```

### 3. Blob 接口和二进制数据
一个 Blob 对象引用一个字节序列，有一个 size 属性，他是字节序列的总字节数，type 属性，是一个 ASCII 小写编码字符串，表示字节序列的媒体类型。

每一个 Blob 必须有一个内部的快照状态，它必须吃书画设置为底层存储的状态，如果存在底层存储。更多关于快照规范定义可以在 Fiel 找到。
```
[Exposed=(Window,Worker), Serializable]
interface Blob {
    constructor(optional sequence<BlobPart> blobParts, 
                optional BlobPropertyBag options = {})

    readonly attribute unsigned long long size;
    readonly attribute DOMString type;

    //切割 Blib 到字节范围串
    Blob slice(optional [Clamp] long long start,
                optional [Clamp] long long end,
                optional DOMString contentType)

    // 从 Blob 读取数据
    [NewObject] ReadableStream steam()
    [NewObject] Promise<USVString> text()
    [NewObject] Promise<ArrayBuffer> arrayBuffer()
};

enum EndingType { "transparent", "native" };

dictionary BlobPropertyBag {
    DOMString type = "";
    EndingType endings = "transparent";
}

typedef (BufferSource or Blob or USVString) BlobPart;
```
Blob 是可序列对象。他们的序列化步骤，给定 value 和 serialized，是：
1. 设置 serialized.[[SnapshotState]] 到 value 的 snapshot state。
2. 设置 serialize.[[ByteSequence]] 到 value 的底层字节序列。

他们的反序列化步骤，给定 serialized 和 value，是：
1. 设置 value 的 snapshop 到 serialized.[[SnapshotState]]
2. 设置 value 的底层字节序列到 serialized.[[ByteSequence]]

一个 Blob blob 有一个关联的获取流算法，执行下面步骤：
1. 让 stream 为构造一个 ReadableStream 对象的结果。
2. 同步执行瞎买呢步骤：
    1. 当 blob 的数据北美被读取完全：
        1. 让 bytes 为从 blob 中读取一个串的结果
        2. 如果读取 bytes 发生文件读取错误，错误的 stream 携带一个失败原因，并终止这些步骤
        3. 入队一个 Uint8Array 包裹一个 ArrayBuffer 对象包含 bytes 到 stream。如果抛出一个异常，错误的 stream 和一个异常，终止这些步骤
        > 问题1 我们需要更具体的指定实际从 Blob 读取神恶魔，什么错误可能发生，可能一些关于串大小，等。
3. 返回 stream

### 3.1 构造器
Blob() 构造器可以使用 0 个或者多个参数调用。当 Blob() 构造器被调用，用户代理必须执行下面的步骤：
1. 如果使用 0 个参数调用，返回一个新的 Blob 对象，他由 0 个字节组成，size 是0，type 是空字符串。
2. 让 bytes 为给定 blob blobParts 和 ptions 处理 blob parts 的结果。
3. 如果 options 参数成员 type 不是空字符串，执行下面的子步骤：
    1. 让 t 为 type 字典成员。如果 t 包含任何 U+0020 打破 U+007E 范围之外的字节，则设置 t 为空字符串并从这些子步骤返回。
    2. 将 t 中的任何字节转化为 ASCII 小写。
4. 返回一个 Blob 对象，它引用 bytes 作为他关联的字节序列，设置 size 为 bytes 的长度，type 设置为从上面子步骤获取到的 t 的值。

### 3.1.1 构造器参数
Blob() 构造器可以使用下面的参数调用

- 一个 BlobParts 序列：可以是任何数量的下面类型的元素，任何顺序：
    - BufferSource 元素
    - Blob 元素
    - USVString 元素
- 一个可选的 BlobPropertyBag：有两个可选的成员
    - type：小写的 ASCII 编码字符串，表示 Blob 的媒体类型。这个成员规范的条件提供在 3.1 构造器
    - endings：一个 enum，有值“transparent”或者“native”。默认值是“transparent”。如果设置为“native”，blobParts 中的任何 USVString 元素的行结束将会被转化为本地。

处理 blob parts，给定一个序列的 BlobParts parts 和 BlobPropertyBag options，执行下面的步骤：
1. 让 bytes 为空的字节序列
2. 使用 element 遍历 parts：
    1. 如果 element 是 USVString，执行下面的子步骤
        1. 让 s 为 element
        2. 如果 options 的成员 endings 是 “native”，设置 s 为 elemen t转化行结束到原生的结果
        3. 拼接 s UTF-8 编码的结果到 bytes
    2. 如果 element 是一个 BufferSource，获取 bytes 的复制，并持有 bffer source。并拼接这些字节到 bytes
    3. 如果 element 是 Blob，拼接它表示的字节到 bytes。
    > Blob 数组元素的 type 被忽略，并且不会影响返回 Blob 对象的 type。
3. 返回 bytes。


将字符串 s 的行结束为 native，执行下面的步骤：
1. 让 native line ending 为码点 U+000A LF 
2. 如果底层平台的转化将新行表示为回车和换行序列，设置 native line ending 为码点 U+000D CR，后面跟着码点 U+000A LF。
3. 设置 result 为空字符串
4. 让 position 为 s 的位置变量，初始化为 s 的开始。
5. 让 token 为从 s 给定 position 收集码点不等于 U+000A LF 或者 U+000D CR 的序列的结果。
6. 拼接 token 到 result
7. 当 position 不超过 s 的结束
    1. 如果 s 在 position 的码点不等于 U+000D CR：
        1. 拼接 native line ending 到result
        2. position 增加 1
        3. 如果 position 不超过 s 的结束，并且 s 在 position 的码点不等于 U+000A LF，position 增加 1
    2. 否则，如果 s 在 position 处的码点不等于 U+000A LF，position 增加 1，并拼接 native line ending 到 result。
    3. 让 token 为从 s 给定 position收集不等于 U+000A LF 或者 U+000D CR 的码点序列的结果。
    4. 拼接 token 到 result
```
// 创建一个新的 Blob 对象
var a = new Blob()

// 创建一个 1024 字节 ArrayBuffer
// buffer 也可以从文件中读取来
var buffer = new ArrayBuffer(1024)

// 基于 buffer 创建 ArrayBufferView 对象
var shorts = new Uint16Array(buffer, 512, 128)
var bytes = new Uint*Array(buffer, shorts.byteOffset + shorts.byteLength)

var b = new Blob(["foobarbazetcetc" + "birdiebirdieboo"], {type: "text/plain;charset=utf-8"})

var c = new Blob([b, shorts])

var a = new Blob([b, c, bytes])

var d = new Blob([buffer, b, c, bytes])
```

### 3.2 属性
- size，类型是 unsigned long long，只读：返回字节序列的字节数量。获取的时候，确保用户代理必须返回可以被 FileReader 或者 FileReaderSync 对象读取的总字节数量，如果 Blob 没有字节可以读取，则返回 0。 
- type，类型是 DOMString，只读：ASCII彪马的小写字符串，表示 Blob 的媒体类型。获取的时候，用户代理必须返回 Blob 的类型作为 ASCII 小写编码字符串，当它转化为字节序列，它是一个可转化的媒体类型，或者空字符串 - 0 字节 - 如果类型不能被决定

type 属性可以被网页应用设置，通过构造器调用和 slice() 调用；在这些栗子中，更多敢于这个属性的额规范条件各自在 3.1 构造器，4.1构造器，和 3.3.1 slice() 方法。用户代理也可以决定 Blob 的类型，特别是如果字节序列是从磁盘文件来的；这种场景下，更多的规范条件在文件类型指南。
> 注意：Blob 的类型 t 被认为是可转化的 MIME 类型，如果在从表示 Blob 对象的 ASCII 编码字符串的字节序列上执行转化 MIME 类型算法没有返回失败。
> 注意：使用 type 属性同志包数据的算法，在获取 blob URL 的时候决定 Content-Type 头部


### 3.3 方法和参数

### 3.3.1 slice() 方法
slice() 方法返回新的 Blob 对象，包含从可选的 start 参数到不包含 end 参数的字节区间，type 属性是可选的 contentType 参数的值，他必须如下表现：
1. 让 O 为 Blob slice() 方法调用的上下文对象
2. 可选的 start 参数是 slice() 调用的开始点的值，必须像字节序位置一样对待，第0位表示第一个字节。用户代理必须使用  start 处理 slice() ，如下规范化：
    a. 如果可选的 start 参数这次调用的时候不作为参数使用，让 relativeStart 为 0
    b. 如果 start 是负，让 relativeStart 为 max((size + start), 0)
    c. 否则，让 relativeStart 为 min(start, size)。
3. 可选的 end 参数是 slice() 调用的结束点的值。用户代理必须使用 end 处理 slice() 通过下面：
    a. 如果可选的 end 参数这次调用的时候不作为参数使用，让 relativeEnd 为 size
    b. 如果 end 是负数，让 relativeEnd 为 max((size + end), 0)。
    c. 否则，让 relativeEnd 为 min(endm size)
4. 如果可选的 contentType 参数用来设置 ASCII 小写编码字符串表示 Blob 的媒体类型。用户代理必须规范的根据下面使用 contentType 执行 slice()：
    a. 如果 contentType 没有提供，让 relativeContentType 设置为空字符串
    b. 否则让 relativeContentType 设置为 contentType，并执行下面子步骤：
        1. 如果 relativeContentType 包含任何 U+0020 到 U+007E 区间外的任何字符，则设置 relativeContentType 为空字符串，并从子步骤返回。
        2. 将 relativeContentType 的任何字符转化为 ASCII 小写。

5. 让 span 为 max((relativeEnd - relativeStart), 0)
6. 返回一个新的 Blob 对象和下面的特征：
    a. S 引用 soan 从 O 的连续字节，从字节序位置 relativeStart 开始的字节
    b. S.size = span
    c. S.type = relativeContentType
```
// 通过 DOM 获取 input 元素
var file = document.getElementById('file;).files[0]
if(file){

    // 创建一个文件的复制标识
    // 下面的两种调用是相等的
    var fileClone = file.slice()
    var fileClone2 = file.slice(0, file.size)

    // 从文件的中间开始切割文件到 1/2 串
    // 注意使用负数
    var fileChunkFromEnd = file,slice(-(Mathi.roud(file.size/2)))

    // 从文件的开始将文件切割称 1/2 的串
    var fileCHunkFromStart = file.slice(0, Math.round(file.size/2))

    // 从文件开始到结束前150 字节切割文件
    var fileNoMetadata = file.slice(0, -150, "application/experimental")
}
```
### 3.3.2 stream() 方法
stream() 方法调用的时候，必须返回在上下文对象调用获取流的结果。

### 3.3.3 text() 方法
text() 方法，调用的时候，必须执行这些步骤：
1. 让 stream 为在上下文对象调用获取流的结果
2. 让 reader 为从 stream 中获取 reader 的结果。如果抛出一个异常，返回一个新的 promise，并使用异常 reject。
3. 让 promise 为使用 reader 从 stream 中读取所哟字节的结果
4. 返回通过返回在它的第一个参数执行 UTF-8 解码的 fulfillment 处理器转化 promise 的结果。

> 注意：这和 readAsText() 不同，它和 Fetch 的 text() 的行为更一致。特别是这个方法将总是使用 UTF-8 作为编码，FileReader 可以使用不同编码取决于 blob 的类型和传入的编码名字。

### 3.3.4 arrayBuffer() 方法

arrayBuffer 方法调用的时候，必须执行下面的步骤：
1. 让 stream 为在上下文对象调用获取 stream 的结果
2. 让 reder 为从 stream 获取 reader 的结果。如果抛出一个异常，返回一个新的 promise，并使用这个异常 rejected。
3. 让 promise 为使用 reader 从 stream 读取所有字节的结果
4. 返回通过一个返回一个新的包含它的第一个参数的 ArrayBuffer 的 fulfillment 处理器转化 promsie 的结果。


### 4. File 接口

一个 File 对象是一个 Blob 对象，有一个 name 属性，它是一个字符串；它可以通过构造器在网页应用汇中创建，或者从一个底层文件系统引用的文件字节序列。

如果一个 File 对象引用一个来自磁盘的文件引用，那么它的 snapshop state 应该设置为 File 对象被创建时候文件在磁盘的状态，

> 注意：这是一个用户代理非正式的需求，虽然不应该，但是应该。用户代理应该有一个 File 对象的 snapshot state 被设置为磁盘底层存储的状态，在引用发生的时候。如果文件在被引用的时候被修改，File 的 snapshot state 将会和底层存储的状态不同。用户代理可能使用修改时间戳和其他机制来维护 snapshot state，但是这是遗留的实现细节。

文件类型指导如下：
- 用户代理必须返回 ASCIII 编码的小写字符串 type，当它转化为相符的字节序列，它是一个可转化的 MIME 类型，或者空字符串 —— 0 字节 —— 如果类型无法决定。
- 如果文件的类型是 text/plain 用户代理必须不拼接一个 charset 参数到媒体类型的 dictionary of parameters 部分。
- 用户代理必须不尝试启发式的决定编码。包括统计方法。
```
[Exposed=(Window,Worker), Serializable]
interface File : Blob {
    constructor(sequence<BlobPart> fileBits,
                USVString fileName,
                optional FilePropertyBag options = {});
    readonly attribute DOMString name;
    readonly attribute long long lastModified;
};

dictionaru FilePropertyBag : BlobPropertyBag {
    long long lastModified;
};
```

File 对象是可序列化的对象。他们的序列化步骤，给定 value 和 serialized，如下：
1. 设置 serialized.[[SnapshotState]] 到 value 的 snapshot state
2. 设置 serialized.[[ByteSequence]] 到 value 的底层字节序列
3. 设置 serialized.[[Name]] 到 value 的 name 属性的值
4. 设置 serialized.[[LastModified]] 到 value 的 lastModified 的值

他们反序列化步骤，给定 value 和 serialized，如下：
1. 设置 value 的 snapshot state 到 serialized.[[SnapshotState]]
2. 设置 value 的底层字节序列到 serialized.[[ByteSequence]]
3. 初始化 value 的 name 属性的值到 serialized.[[Name]]
4. 初始化 value 的 lastModified 属性到 serialized.[[LastModified]]

 


### 4.1 构造器

File 构造器被二到三个参数调用，依赖于可选的字典参数被使用。当 File() 构造器被调用，用户代理必须执行下面的步骤：
1. 让 bytes 为使用 fileBits 和 options 处理 blob parts 的结果
2. 让 n 为新的字符串，他的大小和构造器 fileName 参数一致。复制 fileName 的每一个字符到 fileName 到 n。使用“:”（U+003A COLON）替换任何“/”字符（U+002F SOLIDUS）
> 注意：底层 OS 系统为文件系统使用不同的转化；构造的文件，转化文件名为字节序列的使用 UTF-16 可以较少歧义。
3. 执行下面子步骤处理 FilePropertyBag 字典参数：
    1. 如果提供的 type 成员不是空字符串，让 t 设置为 type 字典成员。如果 t 包含 U+0020 到 U+007E 区间之外的任何字符，则设置 t 为空字符串从这些子步骤返回。
    2. 转化 t 内的任何字符到 ASCII 小写
    3. 如果提供了 lastModified 成员，让 d 设置为 lastModified 字典成员。如果没有提供，设置 d 为当前的日期和时间为从 Unix Epoch 到现在的秒数（和 Date.now()相等）。
4. 如下返回一个新的 File 对象 F ：
    2. F 引用 bytes 的字节序列
    3. F.size 设置为 bytes 的全部字节数
    4. F.name 设置为 n
    5. F.type 设置为 t
    6. F.lastModified 设置为 d

### 4.1.1 构造器参数
File() 构造器可以使用下面的参数调用:

- fileBits 序列：以任何顺序接收下列任何数量的元素：
    - BufferSource 元素
    - Blob元素，包含 File 元素
    - USVString 元素
- fileName 参数：一个 USVString 参数表示文件的名字；这个构造器参数规范化条件可以在 4.1  构造器找到
- 可选的 FilePropertyBag 字典：额外的 BlobPropertyBag 的成员接收一个成员：
    - 一个可选的 lastModified 成员。必须返回一个 long long；这个成员规范化的条件提供在 4.1 构造器


### 4.2 属性

- name，DOMString 类型，只读：文件的名字，获取的时候，这个必须返回文件的名字作为字符串。不同的底层 OS 文件系统使用多种类型的文件名约定和变体；这指示文件的名字，没有路径信息。在获取的时候，如果用户代理不能让这个信息可以获取，必须返回空字符串。如果 FIle 对象使用构造器被创建，这个属性的更多规范条件可以在 4.1 构造器 找到。
- lastModified，long long 类型，只读：文件最后的修改时间。获取的时候，如果用户代理可以获取到这个信息。必须返回一个 long long 类型的数据，他设置为文件以后修改时间从 Unix Epoch 开始到现在的秒数。如果最后修改时期和时间不知道，属性必须返回当前的日期和时间，表示为从 Unix Epoch 开始的毫秒时间；这和 Date.now()。如果 File 对象使用构造器创建，这个属性更多规范的条件可以从 4.1 构造器 找到。

File  接口可以从 FileList 接口暴露的属性获取；这些对象定义在 HTML。File 对象，继承自 Blob，是不可变的，表示读取操作初始化的时候可以读取到内存的数据。用户代理必须处理文件读取时文件不存在的错误，作为一个错误。抛出一个 NotFoundError 异常，如果使用一个 FileReaderSync 在 WebWorker 或者使用 error 属性触发一个 error 事件，返回一个 NotFoundError。

```
var file = document.getElementById("filePicker").files[0];
var date = new Date(file.lastModified);
println("You selected the file " + file.name + " which was modified on " + date.toDateString() + ".");

...

// Generate a file with a specific last modified date

var d = new Date(2013, 12, 5, 16, 23, 45, 600);
var generatedFile = new File(["Rough Draft ...."], "Draft1.txt", {type: "text/plain", lastModified: d})

...
```
### 5. FileList 接口
FileList 接口是“有风险的”，因为通常的趋势是网页平台打算使用 ECMAScript 的 Array 平台对象替换这类对象。特别是，这意味着 filelist.item(0) 是有风险的；FileList 其他编程使用不太可能受升级到 Array 类型的影响。

这个接口是一个列表的 File 对象。
```
[Exposed=(Window,Worker), Serializable]
interface FileList {
    getter File? item(unsigned long index);
    readonly attribute unsigned long length;
}
```
FileList 对象是可序列化对象。他们的序列化步骤，给定 value 和 serialized，是：
1. 设置 serialized.[[Files]] 为空列表。
2. 使用 file 遍历 value，拼接 file 的子序列化到 serialized.[[Files]]
他们的反序列化步骤，给定 serialized 和 value，是：
1. 使用 file 遍历 serialized.[[Files]]，添加 file 的子反序列化到 value。
```
// uploadData 是一个 form元素
// fileChooser 是一个‘file’类型 input
var file = document.forms['uploadData']['fileChooser'].files[0]

// 可替换的语法
// var file = document.forms['uploadData']['fileChooser'].files.item(0)
if(file){
    // 执行文件操作
}
```

### 5.1 属性
lengith，unsigned long 类型，只读：必须返回 FileList 对象中文件的数量，如果没有文件，这个属性必须返回0。


### 5.2 方法和参数

- item(index)：必须返回 FileList 中第 index 个文件。如果在 FileList 中没有第 index 个文件对象，必须返回 null。index 必须被用户代理对待为 File 对象在 FileList 中的位置，0 表示第一个文件。Supported property indices are the numbers in the range zero to one less than the number of File objects represented by the FileList object. If there are no such File objects, then there are no supported property indices.（没看懂）

> 注意：HTMLInputElement 接口有一个只读属性，类型是 FileList，就是上面栗子中访问的。其他的接口有一个类型是 FileList 的只读属性，包括 DataTransfer。



### 6. 读取数据

### 6.1 文件读取任务源
这个规格定义了一个新的通用的任务源，叫做文件读取任务源，用来这个规格读取字节序关联 Blob 和 File 对象入队。他用于触发异步读取二进制数据响应功能。

### 6.2 FileReader API
```
[Exposed=(Window,Worker)]
interface FileReader: EventTarget {
    constructor();
    // 异步读取方法
    void readAsArrayBuffer(Blob blob);
    void readAsBinaryString(Blob blob);
    void readAsText(Blob blob, optional DOMString encoding);
    void readAsDataURL(Blob blob);

    void abort():

    // 状态
    const unsigned short EMPTY = 0;
    const unsigned short LOADING = 0;
    const unsigned short NONE = 0;

    readonly attribute unsigned short readState;

    readonly attribute(DOMString or ArrayBuffer)? result;
    
    readonly attribute DOMException? error;

    // 事件处理器内容属性
    attribute EventHandler onloadstart;
    attribute EventHandler onprogress;
    attribute EventHandler onload;
    attribute EventHandler onabort;
    attribute EventHandler onerror;
    attribute EventHandler onloadend;
}
```

一个 FileReader 有一个关联的 state，他的值是“empty”，“loading”，或者“done”，初始值是“empty”

一个 FileReader 有一个关联的 result（null，一个 DOMString 或者一个 ArrayBuffer），它的初始值是 null。

一个 FileReader 有一个关联的 error（null 或者一个 DOMException）。它的初始值是 null。

FileReader() 构造器调用的时候，必须返回一个新的 FileReader 对象。

readyState 属性获取器，调用的时候，切换上下文对象的状态，并执行关联步骤：
- “empty”：返回 EMPTY
- “loading”：返回 LOADING
- “done”：返回 DONE

result 属性获取器调用的时候，必须返回上下文对象的 result

error 属性获取器调用的时候，必须返回上下文对象的 error

一个 FileReader fr 有一个关联的读取操作算饭，给定 blob，type 和可选的 encodingName，执行下面的步骤：
1. 如果 fr 的 state 是“loading”，抛出一个 InvalidStateError DOMException
2. 设置 fr 的 state 为 “loading”
3. 设置 fr 的 result 为 null
4. 设置 fr 的 error 为 null
5. 让 stream 为在 blob 上调用获取流的结果
6. 让 reader 为 从 stream 获取 reader 的结果
7. 让 byte 为一个空的字节序列
8. 让 chunkPromise 为使用 reader 从 stream 读取一个串的结果
9. 让 isFirstChunk 为 true
10. 同步，直到为真：
    1. 等待 chunkPromise 为 fullfilled 或者 rejected
    2. 如果 chunkPromise 是 fullfilled，并且 isFirstChunk 为 true，入队一个任务在 fr 上去触发进度事件，叫做 loadstart
    3. 设置 isFirstChunk 为 false
    4. 如果 chunkPromise 是 fullfilled，对象的 done 属性是 false，value 属性是一个 Uint8Array 对象，执行下面的步骤：
        1. 让 bs 为 Uint8Array 表示的字节序列
        2. 拼接 bs 到 bytes
        3. 如果最后一次调用这些步骤大约过了 50 ms，入队一个任务在 fr 上触发进度事件，叫做 progress
        4. 设置 chunkPromise 为使用 reader 从 strem 读取一个串的结果
    5. 否则，如果 chunkPromise 是 fullfilled，它的对象的 done 属性是 true，入队一个任务去执行下面的步骤，并停止当前算法：
        1. 设置 fr 的 state 为 ”done“
        2. 让 result 为使用 bytes，type，blob 的 type，和 encodingName 的 package data
        3. 如果 package data 抛出一个异常：
            1. 设置 fr 的 error 为 error
            2. 在 fr 上触发名为 error 的进度事件
        4. 否则：
            1. 设置 fr 的 result 为 result
            2. 在 fr 上触发名为 load 的进度事件
        5. 如果 fr 的 state 不是“loading”，在 fr 上触发名为 loadend 的进度事件
    6. 否则，如果 chunkPromise 被一个 error rejected，入队一个任务去执行下面的步骤并终终止这个算法：
        1. 设置 fr 的 state 为“done”
        2. 设置 fr 的 error 为 error
        3. 在 fr 上触发一个名为 error 的进度事件
        4. 如果 fr 的 state 不是“loading”，在 fr 上触发一个名为 loadend 的进度事件
    
为这些任务使用使用文件读取任务源

### 6.2.1 事件处理器内容属性

### 6.2.2 FileReader 状态

### 6.2.3  读取文件或者 Blob



### 6.3 打包数据
一个 Blob 有一个关联的打包数据算法，给定 bytes，一个 type，一个可选的 mimeType，和一个可选的 encodingName，根据 type 执行关联的步骤：
- DataURL: 遵循下面事项，返回 bytes 的 DataURL 表示
    - 使用 mimeType 作为 DataURL 的一部分，如果他在 Data URL 规格中可用。
    - 如果 mimeType 不可用，返回没有 mime-type 的 Data URL
- Text：
    1. 让 encoding 为 failure
    2. 如果 encodingName 有值，设置 encoding 为 从 encodingName 中获取一个编码的结果
    3. 如果 encoding 是 failure，并且 mimeType 是失败的：
        1. 让 type 为从 mimeType 转化 MIME type 的结果
        2. 如果 type 没有失败，设置 encoding 为 从 type 的参数 ["charset"] 获取一个编码的结果
    4. 如果 encondig 是百事的，则设置 encoding 为 UTF-8
    5. 使用回滚的编码 encondig 解码 bytes，返回结果。

- ArrayBuffer：返回一个新的 ArrayBuffer，它的内容是 bytes
- BinaryString：返回 bytes 作为二进制字符串，每一个字节表示为对应的[0..255]的码点

### 6.4 事件
FileReader 对象必须是这个规格中所有事件的目标

当这个规格说去触发一个进度事件叫做 e（给定 FileReader reader 对于一些 ProgressEvent e 是上下文对象），下面是规范的：
- 进度事件 e 不冒泡，e.bubbles 必须是 false
- 进度事件 e 不可取消，e.cancelable 必须是 false

### 6.4.1 事件描述

下面的事件在 FileReader 对象上被触发

| 事件名 | 接口 | 触发时机 |
| - | - | - |
| loadstart | ProgressEvebt | 当读取开始的时候 |
| progress | ProgressEvebt | 读取（和解码）blob |
| abort | ProgressEvebt | 当读取被放弃的时候，比如，调用 abort() 方法 |
| error | ProgressEvebt | 当读取失败（查看文件读取失败） |
| load | ProgressEvebt | 当读取成功完成 |
| loadend | ProgressEvebt | 当请求完成（成功或者失败） |

### 6.4.2 事件不变的描述
这个章节提供信息

下面是这个规格中异步读取方法触发事件接收的不变性。
1. 一旦 loadstart 被触发，一个对应的 loadend 在读取完成的时候被触发，除非下面的一个条件是 true：
- 读取方法通过调用 abort() 取消，并且一个新的读取方法被调用
- load 事件处理器函数初始化一个新的读取
- error 事件处理器初始化一个新的读取
> 注意：事件 loadstart 和 loadend  不是一对一的
```
reader.readAsText(file);
reader.onload = function(){reader.readAsText(alternateFile)}

reader.readAsText(file)
reader.abort()
reader.onabort = function(){reader.readAsText(updateFile)}
```
2. 一个 progress 事件将会 blob 被全部读取进内存的时候触发
3. 没有 progress 进度事件在 loadstart 之前触发
4. 没有进度事件在 abort，load，和 error 中任何一个被触发之后被触发。对于一次读取，最多一个 abort，load，和 error 可以被触发
5. 没有 abort，load，或者 error 在 loadend 之后被触发

### 6.5 在线程中读取数据

Web Worker 允许使用异步的 File 或者 Blob 读取 API，因为在这些线程上读取不会堵塞主线程。这个章节定义了一个同步的 API，可以在 Workers内使用[[Web Worker]]。Workers 可以使用异步 API（FileReader 对象） 和 同步 API（FileReaderSync 对象）

### 6.5.1 FileReaderSync API
这个接口提供了方法去同步读取 File 和 Blob 对象到内存
```
[Exposed=(DedicateWorker,SharedWorker)]
interface FileReaderSync {
    constructor();

    ArrayBuffer readAsArrayBuffer(Blob blob);
    DOMString readAsBinaryString(Blob blob);
    DOMString readAsText(Blob blobm optional DOMString encoding);
    DOMString readAsDataURL(Blob blob);
}
```

### 6.5.1.1 构造器
当 FileReaderSync() 调用的时候，用户代理必须返回一个新的 FileReaderSync 对象

### 6.5.1.2 readAsText() 方法
readAsText(blob, encoding) 方法调用的时候，必须执行这些步骤：
1. 让 stream 为在 blob 调用获取流的结果
2. 让 reader 为从 stream 获取一个 reader 的结果
3. 让 promise 为使用 reader 从 stream 中读取所有字节的结果
4. 等待 promise 变成 fullfilled 或者 rejected
5. 如果 promise 被字节序列 bytes fulfilled
    1. 返回 bytes，Text，blob 的类型，和 encoding 打包数据的结果
6. 抛出 promise 的 rejection 原因

### 6.5.1.3 readAsDataURL() 方法
当 readAsDataURL(blob) 方法调用的时候，必须执行这些步骤：
1. 让 stream 为在 blob 上调用获取 stream 的结果
2. 让 reader 为从 stream 获取 reader 的结果
3. 让 promise 为使用 reader 从 stream 读取所有字节的结果
4. 等待 promise 被 fulfilled 或者 rejected。
5. 如果 promise 被字节序列 bytes fulfilled
    1. 返回 bytes，Text，blob 的类型，和 encoding 打包数据的结果
6. 抛出 promise 的 rejection 原因

### 6.5.1.4 readAsArrayBuffer() 方法
当 readAsArrayBuffer(blob) 方法调用的时候，必须执行这些步骤：
1. 让 stream 为在 blob 上调用获取 stream 的结果
2. 让 reader 为从 stream 获取 reader 的结果
3. 让 promise 为使用 reader 从 stream 读取所有字节的结果
4. 等待 promise 被 fulfilled 或者 rejected。
5. 如果 promise 被字节序列 bytes fulfilled
    1. 返回 bytes，Text，blob 的类型，和 encoding 打包数据的结果
6. 抛出 promise 的 rejection 原因

### 6.5.1.5 readAsBinaryString() 方法
当 readAsArrayBuffer(blob) 方法调用的时候，必须执行这些步骤：
1. 让 stream 为在 blob 上调用获取 stream 的结果
2. 让 reader 为从 stream 获取 reader 的结果
3. 让 promise 为使用 reader 从 stream 读取所有字节的结果
4. 等待 promise 被 fulfilled 或者 rejected。
5. 如果 promise 被字节序列 bytes fulfilled
    1. 返回 bytes，Text，blob 的类型，和 encoding 打包数据的结果
6. 抛出 promise 的 rejection 原因
> 注意：readAsArrayBuffer() 比 readAsBinaryString() 更好，提供它是为了向后兼容性

### 7. 错误和异常
文件读取错误会在底层文件系统读取文件的时候出现。下面列出的潜在的错误条件是提供信息的
- 被访问的 File 或者 Blob 可能在同步读取方法或者异步读取方法调用的时候不存在。这可能因为它被获取饮用后移除或者删除。（比如，另一个应用同步修改）查看 NotFoundError）
- 一个 File 或者 Blob 可能无法读取。这可能因为一个 File 或者 Blob 被引用后的权限问题（比如，被其他应用同步锁定）。此外，snapshot state 可能被改变。查看 NotSearchableError。
- 用户代理可能决定哪些文件在网页应用内使用不安全。一个文件可能在源文件选中后被修改，导致读取无效。此外，一些文件和字典字典结构可能被底层文件系统约束；尝试读取他们可能被认为有安全风险。查看 9 安全和隐私考虑 和 SecurityError

### 7.1 抛出一个异常或者返回一个错误
这个章节是非规范的

错误条件可以在读取 File 或者 Blob 的时候出现；特别的错误条件，导致获取流失败，叫做失败原因，一个失败原因是 NotFound，UnsafeFile，TooManyReads，SnapshotState，或者 FileLock。

同步读取方法抛出表哥中的类型异常，如果错误术语一个特定的错误原因。

异步方法使用 FileReader 的 error 属性，必须返回下面表格适当的 DOMException 对象，如果错误属于一个特定的错误原因，或者返回 null。
| 类型 | 描述和失败原因 |
| - | - |
| NotFoundError | 当读取正在进行的时候，如果一个 File 或者 Blob 找不到，这是 NotFound 失败原因。对于异步读取方法，error 属性必须返回一个 NotFoundError 异常，同步读取方法必须抛出一个 NotFoundError 异常 |
| SecurityError | 如果：- 决定确定文件在网页应用内访问不安全，这就是 UnsafeFile 失败原因。- 太多读取操作在 File 或者 Blob 资源调用，这是 TooMansyReads 失败原因。对于异步读取方法，error 属性可能返回一个 SecurityError 异常和同步读取方法可能抛出一个 SecurityError 异常。这是一个安全错误不被其他错误原因覆盖 |
| NotReadableError | 如果：- 一个文件的 File 或者 Blob 的 snapshot state 和底层存储的状态不同，这是 SnapshotState 失败原因。- File 或者 Blob 不能被读取，特别是因为权限问题，在 snapshot state 被建立后发生（比如，其他应用在底层存储同步锁定），这是 FileLock 失败原因。对于异步读取方法，error 属性必须返回一个 NotReadableError 异常，同步读取方法一必须排除一个 NotReadableError 异常。 |

### 8. Blob Url 和媒体源引用
这个章节定义了一个 URL 的方案用来引用 Blob 和媒体源对象

### 8.1 介绍
这个章节是提供信息的。

Blob（或者对象）URL 是像 blob:http://example.com/550e8400-e29b-41d4-a716-446655440000 之类的 URL。这让 Blob 和 MediaSource 和其他设计只能和 URL 一起用的 API 融合称为可能，比如 <img> 元素。Blob URL 可以可以用来导航去触发下载本地生成的数据。

为了这个目的，两个静态方法在 URL 接口上暴露，createObjectURL(obj) 和 revokeObjectURL(url)。第一个方法创建了一个从 URL 到 Blob 的映射，第二个方法策撤销映射。只要映射存在，Blob 就不能被垃圾收集，所以，需要特别注意一旦引用不再需要，就立刻撤销映射。所有的 URL 都将被撤销，当创建 URL 的全局变量消失的时候。

### 8.2 模型
每一个用户代理必须维护一个 blob URL store。一个 blob URL store 是一个 map，它的 key 是一个有效的 URL 字符串，它的值是 blob URL 实体。

一个 blob URL 实体由一个 object（Blob 类型或者 MediaSource），和一个环境（一个环境设置对象）

为了生成一个新的 blob URL，执行下面的步骤：
1. 让 result 为一个空字符串
2. 拼接“blob:”字符串到 result
3. 让 settings 为当前设置对象
4. 让 origin 为 settings 的 origin
5. 让 serialized 为 origin 的 ASCII 序列化
6. 如果 serialized 是“null”，设置它为实现定义的值
7. 拼接 serialized 到 result
8. 拼接 U+0024 SOLIDUS(/)  到 result
9. 生成 UUID[RFC4122] 作为字符串并拼接到 result
10. 返回 result

给定 object，添加一个实体到 blob URL store，执行下面的步骤：
1. 让 store 为用户代理的 blob URL store
2. 让 url 为生成一个新的 blob URL 的结果
3. 让 entry 为新的 blob URL 实体，由 object 和当前设置对象组成
4. 设置 store[url] 为 entry
5. 返回 url

给定 url，从 blob URL store 移除一个实体，执行下面的步骤：
1. 让 store 为用户代理的 blob URL store
2. 让 url 字符串为 url 序列化的结果
3. 移除 store[url]

### 8.3 blob URL 不引用模型
给定 url(一个 URL)，获取一个 blob URL，执行下面步骤：
1. 断言：url 的方案是“blob”
2. 让 store 为用户代理的 blob URL store
3. 让 url string 为使用 exclude fragement flag 序列化 url 的结果
4. 如果 store[url string] 存在，返回 store[url string]；否则返回失败。

更多关于 blob URL 的转化和获取模型定义在 [URL] 和 [Fetch] 规格

### 8.3.1 blob URL 的 Origin

这个章节是提供信息的

只要 URL 还没有被撤销，blob URL 的 origin和创建 URL 的环境的一致。在 blob URL store 中查找 URL 是 [URL] 规格达成的，并使用实体返回正确的 origin。

如果 URL 被撤销，origin 的序列化将会依旧和创建 blob URL 的环境的 origin 的序列化一致，但是对于不透明的 origin，origin 本身可能是不同的。这种不同不能被观察，因为一个撤销的 blob URL 无法解析/获取。

### 8.3.2 blob URL 的声明
这个规格使用下面的步骤扩展了 unloading document cleanup steps：
1. 让 environment 为 Document 的 relevant settings object
2. 让 store 为用户代理的 blob URL store
3. 从 store 移除所有 value 的 enviroment 和 environment 相等的 实体 


### 8.4 创建和调用一个 lob URL
Blob URL 使用暴露在 URL 对象的静态方法创建和撤销。废除一个 blob URL，将会吧 blob URL 和它引用的资源解耦，如果它在撤销之后解除引用，用户代理必须表现得像一个网路哦错误发生。这个章节描述了一个 URL 规格补充的接口。表示 blob URL 创建和撤销的方法。
```
[Exposed=(Window,DedicateWorkermSharedWorker)]
partial interface URL {
    static DOMString createObjectURL(Blob or MediaSource obj)
    stateic void revokeObjectURL(DOMString url)
}
```
createObjectURL(obj) 静态方法必须返回 为 obj 添加一个实体到 blob URL store 的结果

revokeObjectURL(url) 静态方法必须执行下面的步骤：
1. 让 url record 为 转化 url 的结果
2. 如果 url record 的方案不是“blob”，返回
3. 让 origin 为 url record 的 origin
4. 让 settings 为当前设置对象
5. 如果 origin 和 settings 的 origin 是同源的，返回
6. 从 blob URL store 中移除 url 的实体

### 8.4.1 blob URL 创建和撤销的栗子
Blob url 是用来获取 blob 对象的字符串，并可以持续使用。creatObjectURL() - 8.3.2 blob URL 的生命

这个章节给出了 blob URL 创建和撤销的栗子和解释

```
url = URL.createObjectURL(blob)
img1.src = url
img2.src = url
```

```
var blobURLref = URL.createObjectURL(file)
img1 = new Image()
img2 = new Image()

img1.src = blobURLref
img2.src = blobURLref

if(img1.complete && img2.complete) {
    URL.revokeObjectURL(blobURLref)
} else {
    msg("Images cannot previewed!")
    URL.revokeObjectURL(blobURLref)
}
```
### 9. 安全和隐私考虑
这个章节是提供信息的。

这个规格允许网页内容从底层文件系统读取文件，同时提供了一种通过通过唯一标识的手段访问，因此受制于一些安全考虑。这个规格也假设主要的用户交互是 HTML 的 <input type="file"/> 元素，所有用户选择的文件使用 FileReader 对象读取。重要的安全考虑包括防止写的的文件选择攻击（选择循环）。防止访问系统敏感文件，和防备在选择之后在磁盘上发生文件修改。

- 防止选择循环：在文件选择期间，一个用户可能被 <input type="file"> 关联的文件选择器轰炸（在一个“必须选中”循环，在文件选择器消失之前强制选择），用户代理可能防止文件访问任何任何选择，通过设置 FileList 对象大小返回 0。
- 系统敏感文件：（比如，在 /usr/bin 的文件，密码文件，和其他本地操作可执行）通常不应该暴露给网页内容，并且不能通过 blob URL 访问，用户代理可能为同步方法抛出一个 SecurityError 异常，或者为异步读取返回一个 SecurityError 异常。

### 10. 需求和用例
