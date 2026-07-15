# bin2hex

将二进制文件转换为 Verilog `$readmemh` 可读的十六进制文件。

## 依赖

Python 3.x，无需安装第三方库。

## 用法

```
python bin2hex [-h] -i FILE -o FILE [-w N] [-d N] [-e {little,big}] [-b] [-s N]
```

## 参数说明

| 参数 | 全称 | 说明 | 默认值 |
|---|---|---|---|
| `-i` | `--input` | 输入二进制文件路径 | 必填 |
| `-o` | `--output` | 输出 hex 文件路径 | 必填 |
| `-w` | `--word-bytes` | 每个 memory word 的字节数 | `8`（64 位） |
| `-d` | `--mem-depth` | 内存深度（word 数），不足时补零行 | 不补零 |
| `-e` | `--endian` | 输入文件的字节序：`little` 或 `big` | `little` |
| `-b` | `--binary` | 输出二进制格式（用于 `$readmemb`） | 关闭（默认输出 hex） |
| `-s` | `--split` | 将输出按深度均分拆分为 N 个文件（需配合 `-d`，且 `-d` 必须能被 N 整除） | `1`（不拆分） |

## 示例

**基本用法（64 位内存，小端）：**
```bash
python bin2hex -i rom.bin -o rom_init.hex
```

**指定 32 位内存：**
```bash
python bin2hex -i rom.bin -o rom_init.hex -w 4
```

**指定内存深度，不足自动补零：**
```bash
python bin2hex -i rom.bin -o rom_init.hex -w 8 -d 65536
```

**大端字节序输入：**
```bash
python bin2hex -i rom.bin -o rom_init.hex -w 8 -d 65536 -e big
```

**输出二进制格式（用于 `$readmemb`）：**
```bash
python bin2hex -i rom.bin -o rom_init.bin -w 4 -b
```

**拆分为多个文件（深度 1024 拆成 4 个 256 深度的文件）：**
```bash
python bin2hex -i rom.bin -o rom_init.rcf -w 8 -d 1024 -s 4
```
生成 `rom_init_0.rcf`、`rom_init_1.rcf`、`rom_init_2.rcf`、`rom_init_3.rcf`，每个文件 256 行。

## 字节序说明

`$readmemh` 读取 hex 文件时，每行代表一个 word，高位在左（大端顺序）。`$readmemb` 读取二进制格式文件，格式相同但内容为二进制字符串。

- **小端（little-endian，默认）**：bin 文件中每个 word 的低字节在前，脚本会自动翻转字节顺序后写入 hex。
- **大端（big-endian）**：bin 文件中每个 word 已是高字节在前，直接写入，无需翻转。

示例：64 位小端整数 `0xDEADBEEFCAFEBABE` 在 bin 文件中的存储为：

```
BE BA FE CA EF BE AD DE   ← 小端（LSB 在前）
```

转换后 hex 输出为：

```
deadbeefcafebabe          ← $readmemh 读入正确的值
```

## 边界处理

- 若 bin 文件最后一个 word 不足 `word_bytes`，自动在**末尾**补零字节。
- 若指定了 `-d`，且 bin 数据不足深度，剩余行全部输出 `00...00`。
- 若指定了 `-d`，且 bin 数据超过深度，超出部分截断并打印警告。
- 若指定了 `-s N`（N>1），需同时指定 `-d`，且 `-d` 必须能被 N 整除；数据先补零/截断到 `-d` 深度，再均分到 N 个文件，文件名按 `<base>_<i><ext>` 顺序生成。

## 在 Verilog 中使用

**$readmemh（十六进制）：**
```verilog
reg [63:0] mem [0:65535];

initial begin
    $readmemh("rom_init.hex", mem);
end
```

**$readmemb（二进制）：**
```verilog
reg [31:0] mem [0:65535];

initial begin
    $readmemb("rom_init.bin", mem);
end
```
