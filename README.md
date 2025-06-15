# GSOC 25 Report
[CCExtractor Development](https://summerofcode.withgoogle.com/programs/2025/organizations/ccextractor-development)
### Re-writing `lib_ccx` libraries to Rust

## Week‑wise Report

#### Week 1  
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

- Finished re‑writing the modules `demuxer` and `file_functions` to Rust.  
- Wrote the files `ctorust.rs`, `stream_functions.rs`, `hlist.rs` and `demuxer.rs`.  
- Fixed all formatting and build issues, and wrote documentation.  
- Completed regression testing on Linux.  
- Primarily, the basic logic and unit tests part was already implemented in May and April, which is what I was working on during the entire Community Bonding Period.  
- So, the first week was on testing regressions which failed for this particular module, writing documentation, and fixing all formatting and CI issues.  
- The Demuxer module does the important work of opening a file, so now ccextractor opens files, detects the stream type, closes files, gets the file size, and performs other tasks in Rust, which is a large step forward in the Rust conversion.  
- The file functions module was also tested and is working, and it is left for use in future modules like MythTV, MXF, GXF, etc.

#### Week 2  
[Pull Request #1649](https://github.com/CCExtractor/ccextractor/pull/1649) 
[Specific Changes I made to this branch](https://github.com/CCExtractor/ccextractor/pull/1649/commits/e04b43017138c3c1e94c245e9d1dd01de293dbed)<br>
AND  
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

- Re‑wrote the logic for the entire bitstream module using an index pointing to the current location of the pos pointer of the bitstream; this was required because all the tests for the MPEG‑2 TS files were failing in the bitstream module.  
- Moved the FFI functions to the `libccxr_exports` directory, and re‑wrote the logic for the FFI using bindgen, as it was not used in the previous case.  
- Resolved CI issues, tested bitstream for both Unix and Windows, and made sure that it worked akin to the main branch.  
- Also worked on PR [#1662](https://github.com/CCExtractor/ccextractor/pull/1662), resolving all regression issues on Windows; these were not caught in Week 1 itself because the tests never ran due to the GPAC servers being down.  
- After the GPAC servers were back up, resolved CI issues regarding Windows, tested all Windows regressions, and fixed several issues—especially catching a large error with the handles used in Windows to describe a file.  
- Added minor documentation to both modules.

