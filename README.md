# GSOC 25 Report
[CCExtractor Development](https://summerofcode.withgoogle.com/programs/2025/organizations/ccextractor-development)
### Re-writing `lib_ccx` libraries to Rust

## Week-wise Report
#### Week 1
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

Finish Re-writing the modules `demuxer` and `file_functions` to Rust. Wrote the `ctorust.rs`, `stream_functions.rs`, `hlist.rs` and `demuxer.rs`. Fixed all formatting and build issues, wrote documentation. Completed Regression Testing on Linux.
Primarily the basic logic and unit tests part was already implemented in May and April, which is what I was working on in the entire duration of the Community Bonding Period. So the first week was on testing regressions which failed for this particular module, writing documentation and fixing all formatting issues and CI issues.
The Demuxer module does the important work of opening a file, so now ccextractor opens files, detects the stream type, closes files, gets the file size and some other work in Rust, which is a large step forward in the rust conversion.
The file functions module was also tested and is working, it is left for use in future modules like MythTV, MXF, GXF, etc.

#### Week 2
[Pull Request #1649](https://github.com/CCExtractor/ccextractor/pull/1649)
AND
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

Re-wrote the Logic for The Entire bitstream module using an index pointing to the current location of the pos pointer of the bitstream, this was required as all the tests for the MPEG-2 TS Files were failing in the bitstream module. Also moved the FFI functions to the libccxr_exports directory, re-wrote the logic for the FFI using bindgen, as it was not used in the previous case. Resolved CI issues, Tested bitstream for both unix and windows, and made sure that it was working akin to the main branch.
Also worked on the [#1662](https://github.com/CCExtractor/ccextractor/pull/1662) PR, resolved all regression issues on Windows, this was not caught on Week 1 itself as the Tests never ran because of the gpac servers being down, after them being up, I resolved CI issues regarding windows and tested all windows regressions and fixed some, especially catching a large error with the Handles used in Windows to describe a file. Also added Minor Documentation to Both modules
