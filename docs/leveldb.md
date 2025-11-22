# Using Snappy with LevelDB

This guide walks through building Snappy from this repository and enabling Snappy compression in [LevelDB](https://github.com/google/leveldb).

## Build and install Snappy

Build Snappy with CMake (matching the version noted in [CMakeLists.txt](../CMakeLists.txt)):

```bash
git submodule update --init
mkdir -p build
cd build && cmake -DSNAPPY_BUILD_TESTS=OFF -DSNAPPY_BUILD_BENCHMARKS=OFF ..
cmake --build .
sudo cmake --install .
```

Notes:
- Installing makes Snappy's headers and libraries discoverable by other projects.
- If you prefer not to install system-wide, set `CMAKE_INSTALL_PREFIX` to a local directory and point LevelDB to it via `CMAKE_PREFIX_PATH`.

## Build LevelDB with Snappy support

LevelDB automatically enables Snappy when it finds the library and headers. A typical CMake-based build looks like:

```bash
git clone https://github.com/google/leveldb.git
cd leveldb
cmake -S . -B build \
  -DLEVELDB_BUILD_TESTS=OFF \
  -DLEVELDB_BUILD_BENCHMARKS=OFF \
  -DLEVELDB_SNAPPY=ON
cmake --build build
```

If Snappy is installed in a non-standard location, add:

```bash
-DCMAKE_PREFIX_PATH="/path/to/snappy/install"
```

to the CMake configuration command.

## Using Snappy compression in your application

Once LevelDB is built with Snappy, compression can be toggled per database via `Options` when opening a `leveldb::DB` instance:

```c++
#include <leveldb/db.h>
#include <leveldb/options.h>

leveldb::Options options;
options.create_if_missing = true;
options.compression = leveldb::kSnappyCompression;

leveldb::DB* db = nullptr;
leveldb::Status status = leveldb::DB::Open(options, "./mydb", &db);
```

- Read and write operations automatically compress/decompress values.
- Use `leveldb::kSnappyCompression` for default compression. To disable compression for a specific column family (in forks that support it), choose `leveldb::kNoCompression`.

## Troubleshooting

- **Snappy not detected**: Re-run CMake for LevelDB with `-DLEVELDB_SNAPPY=ON` and ensure `snappy.h` and the Snappy library are on LevelDB's include/library search paths.
- **Linker errors**: Verify that the Snappy install directory is part of `CMAKE_PREFIX_PATH` or platform-specific library paths (`LD_LIBRARY_PATH` on Linux, `DYLD_LIBRARY_PATH` on macOS).
- **Runtime issues**: Confirm that your LevelDB binary can locate the Snappy shared library at runtime, or link LevelDB statically against Snappy.
