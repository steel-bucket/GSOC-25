# GSOC 25 Report
[CCExtractor Development](https://summerofcode.withgoogle.com/programs/2025/organizations/ccextractor-development)
### Re-writing `lib_ccx` libraries to Rust

## Week‑wise Report

#### Week 1  
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

- Finished re‑writing the modules `demuxer` and `file_functions` to Rust. Wrote the new files `ctorust.rs`, `stream_functions.rs`, `hlist.rs` and `demuxer.rs`.
- Fixed all formatting and build issues, and wrote minor documentation.  
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
- Resolved CI issues, tested bitstream for both Unix and Windows, and made sure that it worked akin to the main branch, I had to fix a bunch of different errors.
- Also worked on PR [#1662](https://github.com/CCExtractor/ccextractor/pull/1662), resolving all regression issues on Windows; these were not caught in Week 1 itself because the tests never ran due to the GPAC servers being down back then.
- After the GPAC servers were back up, resolved CI issues regarding Windows, tested all Windows regressions, and fixed several issues—especially catching a large error with the handles used in Windows to describe a file.
- Completed Regression Testing on Windows.
- Added more minor documentation to both modules.

#### Week 3  
[Pull Request #1](https://github.com/steel-bucket/ccextractor/pull/1) <br>
AND  
[Commit e2d753](https://github.com/CCExtractor/ccextractor/pull/1662/commits/5184ac5bd9645658a4eb3cb7ecdacd7ce6e2d753)

- Re-Wrote the entire gxf module in Rust alongside adjusting unit tests. The GXF module comprises of the `ccx_gxf.c` and `ccx_gxf.h` files.
- Made Extern functions, wrote unit tests, and got the functions working akin to C.
- Some of the logic was already written by me in the Community Bonding period, I had to write more to finish it, and then test it and clear all warnings and regressions.
- Wrote 2 Data Transfer Modules in the `libccxr_exports` directory, for demuxer_data and ccx_gxf comprising of all incumbent enums for transferring data between C and Rust both ways.
- Made sure all of the regressions were working akin to the main branch.
- Refactored all of the `ctorust` functions in [#1662](https://github.com/CCExtractor/ccextractor/pull/1662) based on Rust Best Practice, using `impl` instead of raw functions.

#### Week 4
[Pull Request #2](https://github.com/steel-bucket/ccextractor/pull/2) <br>
[Commit 6dc3b9](https://github.com/CCExtractor/ccextractor/pull/1662/commits/8ab4f837da10be074607c92a484929814671010d) <br>
AND
[Pull Request #1706](https://github.com/CCExtractor/ccextractor/pull/1706)

- Wrote the entire MXF module in Rust  alongside Unit Testing and Formatting.
- Made the Data Transfer Module for MXFContext and made Extern functions in libccxr_exports for the MXF Module.
- This Module entails the Entire Rust Porting of the files `ccx_demuxer_mxf.c` and `ccx_demuxer_mxf.h`.
- Regression tested the MXF Rust module against the MXF file in the CCExtractor Sample Platform. [Link](https://sampleplatform.ccextractor.org/sample/057c1fbc2d9f82691ae0b2150f95765a9c9d894ce1eb297229a05a242650b31a). And confirmed it to be working
- Fixed Clippy errors introduced to the CCExtractor Codebase after Rust v1.88.
- Fixed Clippy errors in the Demuxer Branch as well and rebased that branch.
