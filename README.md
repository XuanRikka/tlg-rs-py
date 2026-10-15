# tlg-rs-py

[tlg-rs](https://github.com/XuanRikka/tlg-rs) 的 Python 绑定（PyO3），提供 KrKr2 tlg 图片格式（tlg5 / tlg6）的编解码能力。

Python 包名是 `tlg-rs`，导入名是 `tlg`。

## 构建

需要 Rust 工具链和 [maturin](https://github.com/PyO3/maturin)：

```bash
pip install maturin

# 开发安装（当前解释器环境）
maturin develop --release

# 或者只出 wheel
maturin build --release
```

> `tlg_core` 依赖 crates.io 上已发布的 `tlg-rs`，需要核心库已发布包含本绑定所需 API 的版本。
> 想对着 tlg-rs 源码联调时，给 `Cargo.toml` 里的 `tlg_core` 临时加回 `path = "../tlg-rs"`（两个仓库放同一父目录）即可。

## 用法

```python
import io
import tlg
from tlg.tlg import TlgType, TlgReader, TlgWriter, Tlg5Encoder, Tlg5Decoder

width, height = 4, 2
rgb = bytes([255, 0, 0] * (width * height))  # RGB888 逐行排列

# TlgWriter/TlgReader：支持 tags，自动识别 tlg5/tlg6
buffer = io.BytesIO()
with TlgWriter.from_rgb(rgb, {"title": "hello"}, width, height, TlgType.Tlg5) as writer:
    writer.write_to(buffer)

with TlgReader(io.BytesIO(buffer.getvalue())) as reader:
    data, info, tags = reader.read()  # -> (bytes, ImageInfo, dict[str, str])

# Tlg5Encoder/Tlg5Decoder、Tlg6Encoder/Tlg6Decoder：不处理 tags，读写裸 tlg 数据
raw = Tlg5Encoder.from_rgb(rgb, width, height).encode()
with Tlg5Decoder(io.BytesIO(raw)) as decoder:
    data, info = decoder.decode()

print(tlg.get_tlg_type(io.BytesIO(raw)))  # TlgType.Tlg5
```

## 接口一览

| 类 | 构造 | 主要方法 |
| --- | --- | --- |
| `TlgReader` | `TlgReader(file)`、`from_reader(file)` | `read() -> (bytes, ImageInfo, dict[str, str])` |
| `TlgWriter` | `from_gray/from_rgb/from_rgba(data, tags, width, height, tlg_type)`、`from_raw(...)` | `write() -> bytes`、`write_to(file)` |
| `Tlg5Encoder` / `Tlg6Encoder` | `from_gray/from_rgb/from_rgba(data, width, height)`、`from_raw(data, width, height, pixel_layout)` | `encode() -> bytes`、`encode_to(file)` |
| `Tlg5Decoder` / `Tlg6Decoder` | `Tlg5Decoder(file)`、`from_reader(file)` | `decode() -> (bytes, ImageInfo)` |

- 六个类都支持 `with`。
- `data` 参数接受任何支持缓冲区协议的对象（`bytes` / `bytearray` / numpy 数组等）。
- `get_tlg_type(file)` 用于探测 tlg 版本，非 tlg 返回 `None`。
- `TlgReader.read()`、解码器的 `decode()`、编码器的 `encode()` / `encode_to(...)` 都会消耗掉对象本身，之后再调用会抛 `RuntimeError`。
- `TlgReader` / `TlgWriter` 处理的是可能带 tags 的完整文件；四个直接编解码器只处理裸 tlg 数据。

## 鸣谢
- KrKr2引擎开发者

## 许可证

MIT
