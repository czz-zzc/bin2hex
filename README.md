# bin2hex

将二进制文件转换为 Verilog `$readmemh` 可读的十六进制文件。

## 依赖

Python 3.x，无需安装第三方库。

## 用法

```
python bin2hex [-h] -i FILE -o FILE [-w N] [-d N] [-e {little,big}]
```

## 参数说明

| 参数 | 全称 | 说明 | 默认值 |
|---|---|---|---|
| `-i` | `--input` | 输入二进制文件路径 | 必填 |
| `-o` | `--output` | 输出 hex 文件路径 | 必填 |
| `-w` | `--word-bytes` | 每个 memory word 的字节数 | `8`（64 位） |
| `-d` | `--mem-depth` | 内存深度（word 数），不足时补零行 | 不补零 |
| `-e` | `--endian` | 输入文件的字节序：`little` 或 `big` | `little` |

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

## 字节序说明

`$readmemh` 读取 hex 文件时，每行代表一个 word，高位在左（大端顺序）。

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

## 在 Verilog 中使用

```verilog
reg [63:0] mem [0:65535];

initial begin
    $readmemh("rom_init.hex", mem);
end
```
