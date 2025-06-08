# GSOC 25 Report
[CCExtractor Development](https://summerofcode.withgoogle.com/programs/2025/organizations/ccextractor-development)
### Re-writing `lib_ccx` libraries to Rust

## Week-wise Report
#### Week 1
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

Finish Re-writing the modules `demuxer` and `file_functions` to Rust. Wrote the ctorust.rs, stream_functions.rs, hlist.rs and demuxer.rs. Fixed all formatting and build issues, wrote documentation. Completed Regression Testing on Linux.
Primarily the logic part was already implemented in May and April, which is what I was working on in the entire duration of the Community Bonding Period.
The Demuxer module does the important work of opening a file, so now ccextractor opens files, detects the stream type, closes files, gets the file size and some other work in Rust, which is a large step forward in the rust conversion.
The file functions module was also tested and is working, it is left for use in future modules like MythTV, MXF, GXF, etc.
