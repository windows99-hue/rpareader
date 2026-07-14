# RPAReader
A lightweight in-memory reader for Ren'Py `.rpa` archives, **no temporary files written to disk**.

## Features
- Fully memory-based extraction: Never dump any assets to local storage
- Standard file-like interface: `read()`, `open()`, `exists()`, indexing support
- Wildcard batch preload for sprites/animations (optimized for desktop pet projects)
- Context manager (`with` statement) for safe auto file closing
- Built-in path normalization, glob pattern matching via `fnmatch`
- Native compatibility with `unrpa` official parsing logic

## Use Case
Perfect for desktop pet / DokiBox style projects:
Load game sprites & animations directly from RPA archives into memory, render images with PySide6 without unpacking resources to disk.

## Installation
```bash
pip install unrpa
```

# Basic Usage

~~~python
from rpa_reader import RPAReader

# Open archive with context manager (auto close)
with RPAReader("images.rpa") as rpa:
    # List all files inside archive
    all_files = rpa.list_files()

    # Check if target asset exists
    if "sprite/monika/idle.png" in rpa:
        # Read raw bytes of image
        img_bytes = rpa.read("sprite/monika/idle.png")
        
        # Get BytesIO stream for QPixmap.loadFromData
        stream = rpa.open_stream("sprite/monika/idle.png")

# File handle auto closed after with block
~~~

### API Reference
#### Constructor
~~~python
RPAReader(archive_path: str)
~~~
 - `archive_path`: Path to target .rpa file
##### Properties
 - `.names`: List all file paths stored in archive
 - `len(rpa)`: Total number of archived files
##### Methods
1. `.list_files() -> list[str]`
Return sorted list of all asset paths.
2. `.exists(name: str) -> bool`
Check if asset path exists in archive.
3. `.read(name: str) -> bytes`
Read full raw binary data of target file.
Raise FileNotFoundError if missing.
4. `.open(name: str) -> io.BytesIO`
Return in-memory binary stream of asset.
5. `.open_stream(name: str) -> io.BytesIO`
Alias of .open(), semantic for graphic rendering.
6. `.preload(patterns: list[str]) -> dict[str, bytes]`
Batch load assets matching prefix / wildcard patterns (fnmatch syntax).
7. `.close()`
Manually close underlying archive file stream.
#### Magic Methods
 - `__contains__`: `if "asset.png" in rpa:`
 - `__getitem__`: `data = rpa["sprite/char.png"]`
 - Context manager support (`with RPAReader(...)`)
#### Key Design Advantage
All extracted asset data lives only in RAM. No temporary images/audio files generated on disk, suitable for lightweight desktop pet tools to avoid messy unpack cache.
#### Limitations
1. Only supports standard Ren'Py RPA3 archives (official unrpa compatibility)
2. Large archives (GB-scale) may consume high RAM if fully preloaded
3. Does not support write / modify RPA archives (read-only only)
#### License
This project using `MIT` License.
