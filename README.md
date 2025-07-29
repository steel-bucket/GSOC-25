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

- Wrote the entire MXF module in Rust, alongside Unit Testing and Formatting.
- Made the Data Transfer Module for MXFContext and made Extern functions in libccxr_exports for the MXF Module.
- This Module entails the Entire Rust Porting of the files `ccx_demuxer_mxf.c` and `ccx_demuxer_mxf.h`.
- Regression tested the MXF Rust module against the MXF file in the CCExtractor Sample Platform. [Link](https://sampleplatform.ccextractor.org/sample/057c1fbc2d9f82691ae0b2150f95765a9c9d894ce1eb297229a05a242650b31a). And confirmed it to be working
- Fixed Clippy errors introduced to the CCExtractor Codebase after Rust v1.88.
- Fixed Clippy errors in the Demuxer Branch as well and rebased that branch.

#### Week 5
[Pull Request #3](https://github.com/steel-bucket/ccextractor/pull/3) <br>
Changes to [Bitstream](https://github.com/CCExtractor/ccextractor/pull/1649) <br>
AND
[Pull Request #1708](https://github.com/CCExtractor/ccextractor/pull/1708)

- Wrote the Entire MythTV Module in Rust, alongside Unit Testing and Formatting
- Made a Data Transfer Module for AVPacketMythTV(renamed from AVPacket) and made Extern functions in libccxr_exports for the MythTV Module.
- This Module entails the Entire Rust Re-writing of the file `myth.c`.
- Regression Tested it against the Myth TS Sample from the Sample Platform [Link](https://sampleplatform.ccextractor.org/sample/121), And confirmed it to be successfully extracting the subtitles.
- Fixed Clashes between the MythTV module and libavutil in hardsubx build.
- Made all Recommended changes by mentors in the bitstream module and got it merged after discussion and some patches.
- Audited my old changes to the decoder module and made a PR($1708) fixing errors happening in XDS and General Regression Tests.

#### Week 6
[Pull Request #1710](https://github.com/CCExtractor/ccextractor/pull/1710) <br>
Changes to Encoding Module(same pr) <br> 
AND <br>
Preview PRs [#1711](https://github.com/CCExtractor/ccextractor/pull/1711) [#1712](https://github.com/CCExtractor/ccextractor/pull/1712) [#1713](https://github.com/CCExtractor/ccextractor/pull/1713)

- Kickstarted the Development for the Encoder(Writer) module, alongside CI Issues and formatting.
- This Module Entails the porting of most helper functions in `ccx_encoders_common.c`, the TXT file encoding, and the library `ccx_encoders_g608.c`.
- Using the `get_str_basic` function(plugged in to rust), now all of the "writing" into TXT files is done through rust.
- Regression Tested against a sample, identified problems in the `encoding` module.
- Fixed the `line21_to_utf8` function in the encoding module.
- Rebased my 3 previous PRs and made 3 main repo PRs for previewing(they are now attached to each other in my repo, for previewing specific changes and also to the main repo to be seen in the main branch), as recommended in weekly meet.

#### Week 7
[Pull Request #4](https://github.com/steel-bucket/ccextractor/pull/4) OR [Pull Request #1717](https://github.com/CCExtractor/ccextractor/pull/1717)
Both are the same, one is for preview

- Kickstarted the Development of the Transport stream module, alongside CI Issues and formatting.
- This module entails porting of most of the functions in `ts_tables.c`
- I shall be adding more and more modules to Complete TS by Week 9 alongside Stream Functions.
- Resolved problems regarding `cinfo_tree`, which is a collection of linked lists and removed it from CcxDemuxer as it requires working on a shared data like in `parse_PMT` or `update_capinfo`.
- `parse_PMT` works locally but needs more testing so it should be released by next week.

#### Week 8
[Pull Request #5](https://github.com/steel-bucket/ccextractor/pull/5) OR [Pull Request #1720](https://github.com/CCExtractor/ccextractor/pull/1720)
Both are the same, one is for preview

- Worked on more of the TS library, porting the entire XMLTV library to Rust.
- This PR entails the rust migration of the entire `ts_tables_epg.c`library.
- Resolved most issues with the library's linux interface. Somehow there are some minute differences between my extracted subtitles and the original ones, particularly in the spanish é and á characters.
- Tested it to be completely working, testing against [Sample #127](https://sampleplatform.ccextractor.org/sample/127) from the sample platform.
- Resolved all CI tests, except one(FIXED!!), Also there's a weird seg fault during extracting from XMLTV files in Windows which I'll also look into(FIXED!!).
- I shall be working on these imperfections, and the failing CI test later, I couldn't do it on 1 week alone as it's quite a large library.
