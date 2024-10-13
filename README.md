# tar

Pure Blade library for creating and extracting TAR archives.

### Package Information
---

- **Name:** tar
- **Version:** 1.0.0
- **Homepage:** _Homepage goes here._
- **Tags:** tar, archive, extract, blade, gzip
- **Author:** Richard Ore <eqliqandfriends@gmail.com>
- **License:** MIT


## Installation
---

You can install the tar library with [Nyssa package manager](https://nyssa.bladelang.org )

```
nyssa install tar
```


## Important

The library exports two helper functions, `compress()` and `extract()`, which allow you to create and extract TAR archives. Note that this library does not yet support TAR archives compressed with the bzip2 algorithm (typically files ending in `.bz2` and `.tbz`).

This library supports the most popular extensions, such as `.tar.gz`, `.tar`, `.gz`, and `.tgz`.


## Extracting TAR archives
---

Use the helper function `extract()` for a quick way to extract TAR archives.

```
import tar

tar.extract('/path/to/archive.tar.gz', '/destination')
```

The destination can be omitted - in which case the archive will be extracted into the same directory as the source with a matching name, e.g. `/path/to/file.tar.gz` will extract to `/path/to/file.tar`.

_See below to learn more about the `extract()` method_


## Creating a new TAR ball
---

To quickly create a new tarball, you can use the `compress()` helper function in the library like this:

```
import tar

tar.compress('/path/to/file/or/directory', '/destination.tar.gz')
```

The compress function can be used to compress a single file or an entire directory. As with the `extract()` function, you can choose to omit the destination parameter - in which case the destination will be set to the current working directory with the same name as the target file/directory and the extension `.tar.gz`.

_See below to learn more about the `compress()` method_


## API Definition
---

For a more fine grained control and to create TAR archives from scratch while adding file and data as you wish or to get the list of files in a TAR archive, you need to make use of the library API as described below. They are self explanatory and you can check out the [examples](#) to see more details on their use.

### Constants

- `COMPRESS_AUTO`: Automatically select and detect compression type (Default).
- `COMPRESS_NONE`: Create and read archives without any compression.
- `COMPRESS_GZIP`: Create and read archives with the GZip compression method.
- `COMPRESS_BZIP`: Create and read archives with the BZip2 compression method (Not yet supported).

### Functions

- `compress(path: string, destination: string = ')`
  
  Create a new TAR ball from the file or directory in the given path and saves it to the destination
  path, or `${NAME_OF_TARGET}.tar.gz` in the current working directory if the destination is not given.

  - **@param** *string* `path` the file or directory that will be compressed.
  - **@param** *string* `destination` the destination of the compressed TAR ball.


- `extract(file: string, destination: string = ')`
  
  Extracts a TAR file to the given destination or to the same directory as the source file with the 
  same name as the TAR file (without the last extension) if destination is not given.
 
  - **@param** *string* `file` the file to be extracted
  - **@param** *string* `destination` the path to extract to (Optional).
  - **@return** *List&lt;Dictionary&gt;* of extracted items (see `contents()`).


### Class Tar

Creates or extracts TAR archives with long pathnames (> 100 characters) support in POSIX ustar and GNU longlink formats.

#### Methods

- `set_compression(level: number = 9, type: COMPRESS_*)`
  
  Set the compression level and type

  - **@param** *int* `level` Compression level (0 to 9) - Default = 9
  - **@param** *int* `type`  Type of compression to use (use COMPRESS_* constants) - Default = COMPRESS_AUTO
  - **@throws** *TarIllegalCompressionException*


- `open(file: string)`
  
  Open an existing Tar file for reading

  - **@param** *string* `file`
  - **@throws** *TarIOException*
  - **@throws** *TarIllegalCompressionException*


- `contents()`
  
  Read the contents of a Tar archive
  
  This function lists the files stored in the Tar, and returns an indexed array of FileInfo objects
  
  The Tar is closed afer reading the contents, because rewinding is not possible in bzip2 streams.
  Reopen the file with open() again if you want to do additional operations
  
  - **@return** *List&lt;Dictionary&gt;*: This dictionary contains the following keys which describe the individual files in the archive:
  
    - `filename`: The name of the file
    - `perm`: Permissions on the file (essentially POSIX file mode). (default: 0)
    - `uid`: Id of the file owner (default: 0)
    - `gid`: Id of the file group (default: 0)
    - `size`: The size of the file (default: 0)
    - `mtime`: The POSIX last modified time of the file (default: 0)
    - `typeflag`: The type of the file (`0` for regular file, `5` for directory)
    - `link`: The number of links on the file
    - `uname`: The name of the file owner
    - `gname`: The name of the file group


- `extract(outdir: string, strip: int|string = 0, exclude: string = '', include: string = '')`
  
  Extract an existing Tar archive
  
  The strip parameter allows you to strip a certain number of path components from the filenames
  found in the Tar file, similar to the --strip-components feature of GNU tar. This is triggered when
  an integer is passed as strip.
  Alternatively a fixed string prefix may be passed in strip. If the filename matches this prefix,
  the prefix will be stripped. It is recommended to give prefixes with a trailing slash.
  
  By default this will extract all files found in the Tar. You can restrict the output using the include
  and exclude parameter. Both expect a full regular expression (including delimiters and modifiers). If
  include is set, only files that match this expression will be extracted. Files that match the exclude
  expression will never be extracted. Both parameters can be used in combination. Expressions are matched against
  stripped filenames as described above.
  
  The Tar is closed afterwards. Reopen the file with open() again if you want to do additional operations
  
  - **@param** *string*     `outdir`  the target directory for extracting
  - **@param** *int|string* `strip`   either the number of path components or a fixed prefix to strip
  - **@param** *string*     `exclude` a regular expression of files to exclude
  - **@param** *string*     `include` a regular expression of files to include
  - **@throws** *TarIOException*
  - **@return** *list*


- `create(file: string = '')`
  
  Create a new Tar file
  
  If file is empty, the Tar file will be created in memory
  
  - **@param** *string* `file`
  

- `add_file(path: string, header: string|dict)`
  
  Add a file to the current Tar using an existing file in the filesystem
  
  - **@param** *string* `path`         path to the original file
  - **@param** *string|dict* `header`  either the name to use in Tar (string) or a dictionary object with all metadata - leave empty to take from original
  - **@throws** *TarIOException*
  
  **NOTE:** If the header is a dictionary, it must conform to the format defined above in `content()`.


- `add_data(data: bytes, header: string|dict)`
  
  Add a file to the current Tar using the given data as content.
  If the header is set to nil or empty string, a file called `Untitled-{CURRENT_TIMESTAMP}` will be created.
  
  - **@param** *bytes* `bytes`     binary content of the file to add
  - **@param** *string|dict* `header` either the name to us in Tar (string) or a dictionary object with all metadata
  - **@throws** *TarIOException*


- `close()`
  
  Add the closing footer to the archive if in write mode, close all file handles
  
  After a call to this function no more data can be added to the archive, for read access no reading is allowed anymore
  
  "Physically, an archive consists of a series of file entries terminated by an end-of-archive entry, which
  consists of two 512 blocks of zero bytes"
  
  - **@link** http://www.gnu.org/software/tar/manual/html_chapter/tar_8.html#SEC134
  - **@throws** *TarIOException*


- `get_archive()`
  
  Returns the created in-memory Tar data. This implicitly calls `close()` on the Tar.

  - **@throws** *TarIOException*
  - **@throws** *TarIllegalCompressionException*
  - **@returns** *bytes*


- `save(path: string)`
  
  Save the created in-memory Tar data
  
  **NOTE:** It is more memory effective to specify the filename in the create() function and
  let the library work on the new file directly.

  - **@param** *string* `path`


- `add_directory(directory: string, file_blacklist: list = [], ext_blacklist: list = [])`
  
  Adds the specified `directory` recursively to the archive and set's it path in the archive to `dir`.
  
  - **@param** *string* `directory`
  - **@param** *string* `file_blacklist` if not empty, this function will ignore every file with a matching path.
  - **@param** *list* `ext_blacklist` if not empty, this function will ignore every file with a matching extension.
  - **@throws** *TarIOException|Exception*


- `file_type(file: string)`
  
  Guesses the wanted compression from the given file
  
  Uses magic bytes for existing files, the file extension otherwise.
  
  **NOTE:** You don't need to call this yourself. It's used when you pass `COMPRESS_AUTO` somewhere.
  
  - **@param** *string* `file`
  - **@return** *int* (one of COMPRESS_BZIP, COMPRESS_GZIP or COMPRESS_NONE)
 
