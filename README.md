# PixelPet

[![build](https://github.com/Prof9/PixelPet/actions/workflows/build.yaml/badge.svg)](https://github.com/Prof9/PixelPet/actions)

PixelPet 是一个用于复古游戏修改的图像处理工具。它可以自动化地将常见的图像文件（如 PNG）转换为复古主机支持的二进制格式。你也可以反向操作，使用 PixelPet 从游戏 ROM 中提取和渲染二进制图像。

目前，PixelPet 主要面向 Game Boy Advance 和 Nintendo DS 系列系统，但也很容易扩展到其他系统。

请注意，PixelPet 仍在开发中，新的功能会根据需要不断添加。因此，以下描述的信息可能会发生变化。

## 工作台（Workbench）

PixelPet 内部有一个“工作台”，用于保存正在处理的主要对象。PixelPet 支持的每个命令都作用于一个或多个这些对象。从这个意义上说，PixelPet 模仿了复古主机的视频系统。

工作台中包含的对象有：

 *  调色板集合（Set of palettes）
 *  位图图像（Bitmap image）
 *  字节流（Stream of bytes）
 *  图块集（Tileset）
 *  图块地图（Tilemap）

例如，加载的位图图像可以被转换为图块集 + 图块地图的组合，然后可以输出为字节流。反之，也可以从字节流读取图块集和图块地图，然后用来渲染新的位图图像。

### 调色板（Palettes）

PixelPet 中的每个调色板都分配有唯一的调色板槽编号。每当添加新调色板时，它会获得当前未被占用的最小槽编号（从 0 开始），当然也可以手动指定调色板槽编号。通过这种机制，PixelPet 完全支持生成和渲染多调色板的图块集和图块地图。

调色板可以包含任意数量的颜色，但可以设置最大颜色数；如果尝试添加超出最大数量的颜色，则会抛出错误。颜色可以采用任何支持的格式，如 24 位和 15 位，并可通过特定命令在格式间转换。

### 位图（Bitmap）

位图就是正在操作的完整图像。在内部，这是一组整数数组，每个像素一个整数。位图可以从多种文件格式导入；不过尚未经过广泛测试，因此建议使用标准的 24 位 RGB 或 32 位 RGBA 格式的 PNG 图像。你也可以将位图导出为标准 PNG 图像文件。

### 字节流（Bytestream）

字节流是一系列字节，工作台中的其他对象都可以序列化为字节流，或从字节流反序列化。它也可以从二进制文件导入或写入二进制文件。

### 图块集（Tileset）

图块集保存了一组图块，可以与图块地图结合使用来表示图像。每个图块集有特定的图块尺寸和颜色格式。默认情况下，图块集的图块尺寸为 8x8 像素，使用 32 位 RGBA 颜色格式。

每当向图块集添加图块时，PixelPet 会先检查该图块是否已存在于图块集中（包括水平或垂直翻转，或两者都翻转的情况）。如果已存在，则不会添加该图块。当然，也可以禁用此行为，以便有意添加重复图块。

图块集可以是非索引的，即每个图块中的像素值就是实际显示的颜色值；也可以是索引的，即每个像素值代表调色板中的索引，显示时会用调色板中对应的颜色值。

### 图块地图（Tilemap）

图块地图保存了一组图块显示条目，可以与图块集结合使用来表示图像。每个条目包含图块编号、是否需要水平/垂直翻转，以及渲染该图块时应使用哪个调色板槽（如果图块集是索引的）。

在 PixelPet 中，图块地图没有特定的预定义宽度或高度（即每行/每列的图块数）。可以无限制地向图块地图添加条目，然后可以以不同的宽度和高度进行渲染。

## 颜色格式（Color formats）

PixelPet 目前支持以下颜色格式。颜色格式描述了每个颜色分量（红、绿、蓝，以及可选的 Alpha）的范围，以及它们在内存中的存储方式。颜色格式的命名遵循“字顺序”（word order），例如 BGR555 表示蓝色分量存储在高位，红色分量存储在低位。

 *  **2BPP** - 通用 2 位灰度，`0x0` 为纯黑，`0x3` 为纯白。
 *  **4BPP** - 通用 4 位灰度，`0x0` 为纯黑，`0xF` 为纯白。
 *  **8BPP** - 通用 8 位灰度，`0x00` 为纯黑，`0xFF` 为纯白。
 *  **GB** - Game Boy 使用的 2 位灰度，亮度取反。`0x0` 为纯白，`0x3` 为纯黑。
 *  **GBA** / **BGR555** - Game Boy Advance 使用的 15 位颜色，红、绿、蓝分量各 5 位。`0x0000` 为纯黑，`0x7FFF` 为纯白。
 *  **NDS** / **ABGR5551** - Nintendo DS 使用的 16 位颜色，红、绿、蓝分量各 5 位，Alpha 分量 1 位。`0x8000` 为纯黑，`0xFFFF` 为纯白。
 *  **24BPP** / **RGB888** - 通用 24 位颜色，红、绿、蓝分量各 8 位。`0x000000` 为纯黑，`0xFFFFFF` 为纯白。
 *  **32BPP** / **ARGB8888** - 通用 32 位颜色，红、绿、蓝、Alpha 分量各 8 位。`0xFF000000` 为纯黑，`0xFFFFFFFF` 为纯白。
 *  **RGB555** - 15 位颜色，红、绿、蓝分量各 5 位。`0x0000` 为纯黑，`0x7FFF` 为纯白。此格式中蓝色分量占低位，红色分量占高位。

## 图块地图格式（Tilemap formats）

PixelPet 目前支持以下图块地图格式。图块地图格式包含了关于图块地图（以及图块集）如何存储的信息。它描述了图块尺寸、是否为索引图像（即是否需要调色板才能正确显示）、图块使用的颜色格式、图块地图条目在内存中的存储方式，以及平台的能力（如是否支持图块翻转）。

 *  **GB** - Game Boy 使用的每像素 2 位图块地图格式，一对 2 字节共同表示 8 个像素。
 *  **GBA-4BPP** / **NDS-4BPP** - Game Boy Advance 和 Nintendo DS 使用的每像素 4 位图块地图格式。该格式使用索引色，需要调色板才能正确显示。如果没有加载合适的调色板，渲染时将使用通用 `4BPP` 灰度格式。
 *  **GBA-8BPP** / **NDS-8BPP** - Game Boy Advance 和 Nintendo DS 使用的每像素 8 位图块地图格式。该格式使用索引色，需要调色板才能正确显示。如果没有加载合适的调色板，渲染时将使用通用 `8BPP` 灰度格式。

## 用法（Usage）

PixelPet 目前支持以下命令。每个命令作为单独的命令行参数指定，后跟其参数，也作为命令行参数。例如，要加载输入图像 `input.png` 并将其另存为 `output.png`，可以在你喜欢的控制台中运行以下命令：

```
PixelPet Import-Bitmap "input.png" Export-Bitmap "output.png"
```

或者，也可以将以下内容写入一个基本文本文件 `script.txt`（扩展名不限）：

```
Import-Bitmap "input.png"
Export-Bitmap "output.png"
```

然后，在控制台中运行该脚本：

```
PixelPet Run-Script "script.txt"
```

## 命令（Commands）

### Help
```
Help
```

打印 PixelPet 支持的命令及其参数列表。

### Run-Script
```
Run-Script <path> [--recursive/-r]
```

运行指定 `<path>` 的基本文本文件中的脚本。该脚本还可以包含更多 `Run-Script` 命令，以运行其他文件中的脚本。

默认情况下，不允许递归包含当前堆栈上已存在的脚本。例如，如果名为 `script.txt` 的脚本包含命令 `Run-Script script.txt`，则会抛出错误。此外，如果 `script1.txt` 调用 `script2.txt`，而 `script2.txt` 又调用 `script1.txt`，也会抛出错误。不过，可以通过指定 `--recursive` 或 `-r` 标志来禁用此限制。

**示例用法：**
```
Run-Script "script.txt"
```

### Set-Variable
```
Set-Variable <name> <value>
```

将名为 `<name>` 的变量设置为字符串 `<value>`。如果变量不存在，则会创建。`<name>` 只能包含字母、数字和下划线（`[A-Za-z0-9_]`）。

设置变量后，可以在任何其他命令（包括另一个 `Set-Variable`）中通过 `<变量名>` 的方式使用该变量。如果向命令传递包含变量的参数，执行命令前会展开参数中的所有变量。展开是通过将 `<var>`（其中 var 是变量名）替换为该变量的字符串值来完成的。可以嵌套参数，例如 `<var1<var2>>`。

使用 `Run-Script` 命令时，所有变量也会传递给脚本。

**示例用法：**
```
Set-Variable map_top "map-top.png"
Set-Variable map_bottom "map-bottom.png"
Set-Variable layer "top"
Import-Bitmap "<map_<layer>>"
```

将图像 `map-top.png` 导入到工作台。

### Import-Bitmap
```
Import-Bitmap <path> [--format/-f <format>]
```

将指定 `<path>` 的图像文件导入到工作台位图。之前的工作台位图会被丢弃。建议使用标准的 24 位或 32 位 PNG 图像。

如果未指定 `--format` 选项，则所有像素按原样导入。如果指定，则所有像素将按指定的颜色格式解释。

**示例用法：**
```
Import-Bitmap "input.png"
```
将图像 `input.png` 导入到工作台。

### Export-Bitmap
```
Export-Bitmap <path> [--format/-f <format>]
```

将工作台位图导出到指定 `<path>` 的文件。输出位图始终以 32 位图像存储。

如果未指定 `--format` 选项，则导出时所有像素会转换为 32 位色（不会修改工作台位图）。如果指定，则像素会转换为给定格式。注意，由于图像本身以 32 位格式存储，这意味着图像可能无法正确显示，但比导出为字节再用十六进制编辑器检查更方便。

不过，如果 `--format` 选择的颜色格式不包含 alpha 通道，`Export-Bitmap` 会在转换后将所有像素的 32 位 alpha 通道设置为 255。

**示例用法：**
```
Export-Bitmap "output.png"
```
将工作台中的位图导出为 `output.png`。

### Import-Bytes
```
Import-Bytes <path> [--append/-a] [--offset/-o <count>] [--length/-l <count>]
```

将指定 `<path>` 的二进制文件导入到工作台字节流。如果指定 `--append`，则二进制文件会追加到之前的工作台字节流；否则，之前的字节流会被丢弃。

可选地，可以使用 `--offset` 和/或 `--length` 参数指定从文件中读取字节的偏移量（默认为 0）和最大读取字节数（默认为不限制）。字节会一直读取到文件末尾或达到最大长度，以先到者为准。

**示例用法：**
```
Import-Bytes "input.bin" --offset 0x4 --length 0x20
```
从文件 `input.bin` 的偏移 4（0x4）处开始导入 32（0x20）字节。

### Export-Bytes
```
Export-Bytes <path>
```

将整个工作台字节流导出到指定 `<path>` 的文件。

**示例用法：**
```
Export-Bytes "output.bin"
```
将工作台字节流导出为 `output.bin`。

### Clear-Palettes
```
Clear-Palettes
```

丢弃所有工作台调色板。

### Clear-Tileset
```
Clear-Tileset [--tile-size/-s <width> <height>]
```

丢弃工作台图块集，并初始化一个新的。可选地，`--tile-size` 参数可用于指定新图块集的图块宽度和高度（像素）。默认是 8x8 像素。

```
Clear-Tileset --tile-size 16 16
```
初始化一个 16x16 像素的图块集。

### Clear-Tilemap
```
Clear-Tilemap
```

丢弃工作台图块地图，并初始化一个新的。

### Extract-Palettes
```
Extract-Palettes [--append/-a] [--palette-number/-pn <number>] [--palette-size/-ps <count>] [--palette-count/-pc <count>] [--x/-x <pixels>] [--y/-y <pixels>] [--width/-w <pixels>] [--height/-h <pixels>] [--tile-size/-s <width> <height>]
```

从工作台位图中提取调色板。

首先将位图分割为图块。对于每个图块，取出其中的颜色值，尽可能添加到现有调色板（去除重复色），如果无法加入则新建调色板。

PixelPet 尝试最小化使用的调色板数量，但采用贪心算法，因此不保证一定是最少的。如果对此有要求，建议手动创建模板并通过 `Read-Palettes` 命令加载。

如果指定 `--append`，则提取的调色板会添加到当前工作台调色板，否则会先丢弃当前调色板。

`--palette-number` 可指定新建调色板的初始槽编号。该编号会递增直到找到空槽。如果未指定，新调色板会获得已加载调色板中最高编号 + 1。

`--palette-size` 可指定新建调色板的最大大小。之前已创建的调色板不受影响。默认最大大小不受限制。

`--palette-count` 可指定最多可添加的调色板数量。如果位图需要更多调色板，则会抛出错误。默认不限制数量。

`--x`、`--y`、`--width` 和 `--height` 可只处理位图的一部分。位图本身不会改变，但会按裁剪后的区域处理，第一块图块从裁剪后位图的左上角开始。默认处理整个位图。

`--tile-size` 指定每个图块的像素大小。如果省略，则使用当前工作台图块集的图块大小。特殊情况：`--tile-size` 的宽度和/或高度为 0 时，会使用整个位图的宽度和/或高度，例如 `--tile-size 0 0` 会将整个位图视为一个“图块”。

**示例用法：**
```
Import-Bitmap "input.png"
Extract-Palettes --palette-size 16
```
从工作台位图中提取 16 色调色板。

### Read-Palettes
```
Read-Palettes [--append/-a] [--palette-number/-pn <number>] [--palette-size/-ps <count>] [--x/-x <pixels>] [--y/-y <pixels>] [--width/-w <pixels>] [--height/-h <pixels>]
```

从加载到工作台位图的*调色板图像*中读取调色板。读取的颜色会加载到新调色板中。

该命令可用于加载预定义的调色板集合。调色板以*调色板图像*的形式存储：由 8x8 像素的纯色块组成的图像，每个色块代表调色板中的一种颜色。例如，16 色调色板可用 128x8 像素的调色板图像表示。色块按从左到右、从上到下读取。如果某个 8x8 色块不是完全相同的颜色，则会抛出错误。

该命令与 `Extract-Palettes` 不同，用户可显式指定调色板，因此可以添加重复色（`Extract-Palettes` 会去除重复色）。

如果指定 `--append`，则读取的调色板会添加到当前工作台调色板，否则会先丢弃当前调色板。

`--palette-number` 可指定新建调色板的初始槽编号。该编号会递增直到找到空槽。如果未指定，新调色板会获得已加载调色板中最高编号 + 1。

`--palette-size` 可指定新建调色板的最大大小。达到最大后会新建调色板，因此可一次读取多个调色板。如果未指定，所有颜色会放入一个不受限制的调色板。

`--x`、`--y`、`--width` 和 `--height` 可只处理位图的一部分。位图本身不会改变，但会按裁剪后的区域处理，第一块图块从裁剪后位图的左上角开始。默认处理整个位图。

**示例用法：**
```
Import-Bitmap "128x16.png"
Read-Palettes --palette-size 16
```
导入一个 128x16 像素的调色板图像，表示两个 16 色调色板，然后从调色板图像中读取这两个调色板。

### Render-Palettes
```
Render-Palettes [--colors-per-row/-cw <count>]
```

将工作台调色板渲染为*调色板图像*，并写入工作台位图。原有位图会被丢弃。这与 `Read-Palettes` 命令使用的调色板图像相同。每个已加载的调色板占据一行，每种颜色渲染为 8x8 像素的色块。调色板图像以 32 位色渲染。

`--colors-per-row` 可指定每行渲染的颜色数。如果颜色数超过一行，则自动换行。如果一行未满但调色板已结束，则直接换行，剩余部分留空。该选项可用于将 256 色调色板渲染为 16x16 的调色板图像，而不是 256x1。未指定时，调色板图像宽度为最大调色板的宽度。

**示例用法：**
```
Import-Bitmap "input-16-color.png"
Extract-Palettes --palette-size 16
Render-Palettes
Export-Bitmap "palette.png"
```
导入 16 色图像，生成 16 色调色板，渲染为调色板图像，并写入 `palette.png`。

### Render-Tileset
```
Render-Tileset [--tiles-per-row/-tw <count>] [--format/-f <format>]
```

将工作台图块集渲染为图像，并写入工作台位图。原有位图会被丢弃。

如果图块集使用索引色，则渲染时会用工作台调色板。如果有与原索引调色板编号相同的调色板，则用该调色板渲染；否则用当前已加载的第一个调色板；如果没有调色板，则用通用灰度调色板渲染。

如果图块集不使用索引色，则每个像素按原样渲染。

`--tiles-per-row` 可指定每行渲染的图块数。默认是 32 个图块。对于 8x8 像素的图块集，渲染图像宽度为 256 像素，高度自动计算。

如果指定 `--format`，则用指定颜色格式渲染，否则用 32 位色渲染。

**示例用法：**
```
Import-Bitmap "input.png"
Extract-Palettes
Generate-Tilemap GBA-4BPP
Render-Tileset --tiles-per-row 16
Export-Bitmap "tileset.png"
```
导入图像，生成调色板，生成图块地图+图块集，将图块集渲染为每行 16 个图块的图像，并写入 `tileset.png`。

### Render-Tilemap
```
Render-Tilemap <tiles-per-row> <tiles-per-column>
```

将工作台图块地图和图块集一起渲染为图像，并写入工作台位图。原有位图会被丢弃。图块地图以 32 位色渲染。

与 `Render-Tileset` 不同，必须同时指定 `<tiles-per-row>` 和 `<tiles-per-column>`，分别表示每行和每列渲染的图块数。例如，`Render-Tilemap 30 20` 且图块为 8x8 像素时，结果为 240x160 像素的位图。

图块地图条目按从左到右、从上到下渲染。

**示例用法：**
``` 
Import-Bytes "tilemap.bin"
Deserialize-Tilemap
Import-Bytes "tileset.bin"
Deserialize-Tileset GBA-4BPP
Render-Tilemap 30 20
Export-Bitmap "tilemap.png"
```
导入并反序列化二进制文件中的图块地图和图块集，然后渲染为 240x160 像素的图像，并写入 `tilemap.png`。

### Crop-Bitmap
```
Crop-Bitmap [--x/-x <pixels>] [--y/-y <pixels>] [--width/-w <pixels>] [--height/-h <pixels>]
```

使用指定参数裁剪工作台位图。`--x` 和 `--y` 指定左上角的 x、y 偏移，`--width` 和 `--height` 指定新位图的宽度和高度。如果参数超出原位图范围，会自动调整到位图边界。

**示例用法：**
```
Crop-Bitmap --width 32 --height 32
```
将工作台位图裁剪为左上角 32x32 像素。

### Convert-Bitmap
```
Convert-Bitmap <format> [--sloppy/-s]
```

将工作台位图转换为指定颜色格式。

默认情况下，大多数命令生成的工作台位图为 32 位色。通常在进一步处理以适配复古主机显示前，需要先转换为合适的颜色格式。

对于每个像素的红、绿、蓝、Alpha 分量，按如下方式缩放：

```
out = (in + (in_max / 2) * out_max) / in_max
```
每次除法向下取整。

大致等价于：
```
out = in * out_max / in_max
```
每次除法按正确方向取整。

例如，将 5 位色分量 `31` 转为 8 位色分量：
```
31 * 255 / 31 = 255
```

但许多模拟器和图像处理工具使用了错误的色彩缩放算法，即直接左移并用零填充。例如，同样将 5 位色分量 `31` 转为 8 位：
```
31 << 3 = 248
```

反之亦然：
```
248 >> 3 = 31
```

指定 `--sloppy` 选项时，PixelPet 会采用这种位移缩放算法。可用于处理从不准确模拟器提取的图像。

**示例用法：**
```
Import-Bitmap "input.png"
Convert-Bitmap GBA --sloppy
Convert-Bitmap 32BPP
Export-Bitmap "output.png"
```
加载一张从不准确模拟器提取的 32 位图像 `input.png`，用 `--sloppy` 选项转换回原始 GBA 色彩格式，再用正确缩放转换回 32 位色，最终写入 `output.png`，实现“修复”图像。

### Convert-Palettes
```
Convert-Palettes <format> [--sloppy/-s]
```

将所有工作台调色板从当前颜色格式转换为指定颜色格式。

与 `Convert-Bitmap` 类似，但操作对象为调色板中的每个颜色。通常比 `Convert-Bitmap` 更快。

### Deduplicate-Palettes
```
Deduplicate-Palettes [--global/-g]
```

将所有工作台调色板中的重复颜色替换为新的唯一颜色。

该命令使用二分查找算法，复杂度为 `O(m log 2^n)`，其中 `m` 为需生成的新颜色数，`n` 为色彩空间的位数，保证生成的颜色唯一（只要总调色板大小不超过色彩空间大小）。

`Deduplicate-Palettes` 生成的新颜色是随机的，不保证一致性，但在同一 PixelPet 二进制文件的多次运行中应保持一致。

通常，去重在每个调色板内单独进行。如果指定 `--global`，则对所有调色板整体去重。例如，若有两个调色板有完全相同的颜色，`--global` 时其中一个会被替换。`--global` 要求所有调色板颜色格式一致。

### Pad-Palettes
```
Pad-Palettes <width> [--color/-c <value>] [--palette-size/-ps <count>]
```

通过添加颜色将所有工作台调色板填充到至少指定宽度。

`--color` 可指定添加的颜色值，默认是 0。

如果工作台没有调色板，会新建一个并填充到指定宽度。`--palette-size` 指定新建调色板的最大大小，未指定则不限制。

**示例用法：**
```
Import-Bitmap "input.png"
Extract-Palettes --palette-size 16
Pad-Palettes 16
```
导入图像 `input.png`，提取调色板并填充到 16 色。

### Pad-Tileset
```
Pad-Tileset <width> [--color/-c <value>] [--tile-size/-s <width> <height>]
```

通过添加图块将工作台图块集填充到至少指定数量。

`--color` 指定填充图块的颜色值，省略则为 0。

`--tile-size` 指定每个图块的像素大小，省略则用当前图块集的大小。如果当前图块集非空，指定的大小必须与其一致。

### Generate-Tilemap
```
Generate-Tilemap <format> [--append/-a] [--no-reduce/-nr] [--x/-x <pixels>] [--y/-y <pixels>] [--width/-w <pixels>] [--height/-h <pixels>] [--tile-size/-s <width> <height>]
```

从工作台位图生成图块地图和图块集。位图按从左到右、从上到下处理。

`<format>` 参数指定使用的图块地图格式，也决定生成的图块是否为索引色。注意，生成索引色图块前需先加载或提取合适的调色板。

如果指定 `--append`，则新生成的图块地图和图块会追加到当前工作台，否则会先清空。

默认情况下，PixelPet 会尝试通过去除已存在（包括翻转）的图块来减少图块数量（前提是格式支持）。可用 `--no-reduce` 禁用此行为，强制每个图块都加入。

`--x`、`--y`、`--width` 和 `--height` 可只处理位图的一部分。位图本身不会改变，但会按裁剪后的区域处理，第一块图块从裁剪后位图的左上角开始。默认处理整个位图。

`--tile-size` 指定每个图块的像素大小，省略则用当前图块集的大小。如果当前图块集非空，指定的大小必须与其一致。

**示例用法：**
```
Import-Bitmap "input-256-colors.png"
Extract-Palettes --palette-size 256
Generate-Tilemap GBA-8BPP
```
导入 `input-256-colors.png`，提取 256 色调色板，然后用该调色板生成优化后的 `GBA-8BPP` 索引图块地图和图块集。

### Deserialize-Palettes
```
Deserialize-Palettes <format> [--append/-a] [--palette-number/-pn <number>] [--palette-size/-ps <count>] [--palette-count/-pc <count>] [--offset/-o <count>]
```

从工作台字节流反序列化一系列调色板。`<format>` 指定调色板的颜色格式。

默认从字节流起始读到末尾，生成一个不受限制的调色板。`--palette-size` 可指定每个调色板的大小，达到后新建调色板，直到读完。`--palette-count` 可限制调色板数量，提前停止。

指定 `--append` 时，新调色板会追加到当前工作台，否则会先清空。

`--palette-number` 可指定新建调色板的初始槽编号，递增直到空槽。未指定则为已加载调色板中最高编号 + 1。

`--offset` 可指定从字节流的偏移处开始读取。

**示例用法：**
```
Import-Bytes "rom.gba"
Deserialize-Palettes GBA --palette-size 16 --palette-count 2 --offset 0x600000
```
导入 ROM `rom.gba`，并从偏移 `0x600000` 处读取两个 16 色 GBA 格式调色板。

### Serialize-Palettes
```
Serialize-Palettes [--append/-a]
```

将当前所有工作台调色板序列化到字节流。每种颜色按当前格式序列化，例如 15 位色调色板每色 2 字节。调色板按添加顺序序列化。

指定 `--append` 时，序列化结果追加到当前字节流，否则会先清空。

### Deserialize-Tileset
```
Deserialize-Tileset <tilemap-format> [--append/-a] [--tile-count/-tc <count>] [--offset/-o <count>] [--tile-size/-s <width> <height>]
```

从工作台字节流反序列化图块集。

指定 `--append` 时，图块追加到当前图块集，否则会先清空。

`<tilemap-format>` 参数指定使用的图块地图格式。默认从字节流起始读到末尾，可用 `--offset` 指定起始地址，`--tile-count` 指定读取图块数，达到后停止。

`--tile-size` 指定每个图块的像素大小，省略则用当前图块集的大小。如果当前图块集非空，指定的大小必须与其一致。

**示例用法：**
```
Import-Bytes "rom.gba"
Deserialize-Tileset GBA-4BPP --tile-count 0x20 --offset 0x700000
```
导入 ROM `rom.gba`，并从偏移 `0x700000` 处读取 32（0x20）个 GBA-4BPP 格式图块。

### Serialize-Tileset
```
Serialize-Tileset [--append/a]
```

将整个工作台图块集序列化到字节流。每个图块按当前格式序列化，例如 8x8 像素 4BPP 图块为 32 字节。图块按添加顺序序列化。

指定 `--append` 时，序列化结果追加到当前字节流，否则会先清空。

### Deserialize-Tilemap
```
Deserialize-Tilemap <tilemap-format> [--append/-a] [--base-tile/-bt <index>] [--tile-count/-tc <count>] [--offset/-o <count>]
```

从工作台字节流反序列化图块地图。

`<tilemap-format>` 参数指定序列化图块地图的格式。

指定 `--append` 时，条目追加到当前图块地图，否则会先清空。

`--base-tile` 可指定与图块集第一个图块对应的编号，反序列化时会从条目编号中减去该值。默认是 0。

默认从字节流起始读到末尾，可用 `--offset` 指定起始地址，`--tile-count` 指定读取条目数，达到后停止。

**示例用法：**
```
Import-Bytes "rom.gba"
Deserialize-Tilemap --tile-count 600 --offset 0x800000
```
导入 ROM `rom.gba`，并从偏移 `0x800000` 处读取 600 个图块地图条目。例如对应 30x20 的图块地图。

### Serialize-Tilemap
```
Serialize-Tilemap [--append/-a] [--base-tile/-bt <index>] [--first-tile/-ft <tilemap-entry>]
```

将整个工作台图块地图序列化到字节流。目前该命令硬编码为 GBA 图块地图格式。

指定 `--append` 时，序列化结果追加到当前字节流，否则会先清空。

`--base-tile` 可指定与图块集第一个图块对应的编号，序列化时会加到条目编号上。默认是 0。

`--first-tile` 可将所有引用第一个图块的条目替换为指定值。例如，可以为图块集设置 `--base-tile`，但仍用 0 号图块作为透明图块，此时可用 `--first-tile` 指定条目值。

**示例用法：**
```
Import-Bitmap "input-16-colors.png"
Extract-Palettes --palette-size 16
Generate-Tilemap GBA-4BPP
Serialize-Tilemap --base-tile 31 --first-tile 0
```
导入 `input-16-colors.png`，生成 16 色调色板，生成图块集+图块地图，序列化时所有图块编号加 31，引用第一个图块的条目用 0。

### Deserialize-Bitmap
```
Deserialize-Bitmap <format> <width> <height> [--offset/-o <count>]
```

从工作台字节流反序列化位图到工作台位图。

`<format>` 参数指定位图的颜色格式。

`<width>` 和 `<height>` 参数指定位图的像素宽度和高度。

默认从字节流起始读 `<width>*<height>` 个像素，可用 `--offset` 指定起始地址。

**示例用法：**
```
Import-Bytes "rom.gba"
Deserialize-Bitmap GBA 256 256 --offset 0x800000
```
导入 ROM `rom.gba`，并从偏移 `0x800000` 处反序列化 256x256 位图。

### Serialize-Bitmap
```
Serialize-Bitmap [--append/-a]
```

将工作台位图序列化到字节流，使用当前位图的颜色格式。

指定 `--append` 时，序列化结果追加到当前字节流，否则会先清空。

**示例用法：**
```
Import-Bitmap "input.png"
Convert-Bitmap GBA
Serialize-Bitmap
```
导入位图 `input.png`，转换为 GBA 色彩，再序列化到字节流。

### Apply-Palette-Bitmap
```
Apply-Bitmap-Palette [--palette-number/-pn <number>]
```

将调色板应用到整个工作台位图。为此，位图必须为可用作调色板索引的颜色格式（如 8BPP）。命令执行后，位图中的颜色索引会被调色板中的实际颜色替换。

如果指定 `--palette-number`，则用该编号的调色板，否则用编号最低且颜色数足够的调色板。

只能对整个位图应用单一调色板。如需多调色板，请用 `Render-Tileset` 或 `Render-Tilemap` 渲染索引图块。

**示例用法：**
```
Import-Bytes "rom.gba"
Deserialize-Bitmap GBA-4BPP 256 256 0x800000
Deserialize-Palettes GBA --offset 0x700000 --length 0x20
Apply-Palette-Bitmap
```
导入 ROM `rom.gba`，从 `0x800000` 反序列化 256x256 4BPP 图像，从 `0x700000` 反序列化 16 色调色板，然后将调色板应用到位图。

### Quantize-Bitmap
```
Quantize-Bitmap <format> [--palette-number/-pn <number>]
```

使用调色板对工作台位图进行量化。命令执行后，位图中的实际颜色会被调色板索引替换。

`<format>` 参数指定量化后像素的颜色格式。序列化或渲染时，若未加载调色板，将用此格式。例如，16 色调色板通常用 4BPP。

如果指定 `--palette-number`，则用该编号的调色板，否则用包含所有位图颜色的编号最低的调色板。

**示例用法：**
```
Import-Bytes "rom.gba"
Import-Bitmap "input.png"
Deserialize-Palettes GBA --offset 0x700000 --length 0x20
Quantize-Bitmap GBA-4BPP
```
导入 ROM `rom.gba` 和位图 `input.png`，从 `0x700000` 反序列化 16
