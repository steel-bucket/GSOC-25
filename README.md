# Google Summer of Code 2025 - Final Submission Report

## Re-writing `lib_ccx` Libraries to Rust

**Organization:** [CCExtractor Development](https://summerofcode.withgoogle.com/programs/2025/organizations/ccextractor-development)  
**Mentors:** Prateek Sunal, Carlos Fernandez Sanz, and Willem Van Iseghem   
**Contributor:** Deepnarayan Sett  
**Contact:** 22bec038@iiitdmj.ac.in

---

## Table of Contents

1. [Synopsis](#synopsis)
2. [Modules I Worked On](#modules-i-worked-on)
3. [Other Contributions](#other-contributions)
4. [Work Reports](#work-reports)
5. [Week-wise Report](#week-wise-report)
6. [Key Technologies Used](#key-technologies-used)
7. [Work That's Left](#work-thats-left)
8. [Acknowledgments](#acknowledgments)

---

## Synopsis

This project encompasses the work behind rewriting the `lib_ccx` libraries, which are currently written in C, in Rust for the sake of memory safety, better documentation, new features, and removing redundant code. A significant amount of C code has been rewritten in Rust, including CI, unit, and regression testing alongside substantial additional work. Over 15 libraries have been ported to Rust, encompassing 25 C files and header files (including open PRs). Three of the largest open PRs have been resolved and rewritten to completion along with comprehensive testing.

---

## Modules I Worked On

### 1. File Functions Module

**Files Ported:** `file_functions.c`, `file_buffer.h`

- This module is the backbone of the demuxer class. It can open unopened files, move incoming bytes back into the demuxer's file buffer, perform reading and seeking operations, and much more.
- It has been extensively unit tested. Tests and code can be found [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/file_functions/file.rs).

---

### 2. Demuxer/Stream Functions Module

**Files Ported:** `ccx_demuxer.c`, `ccx_demuxer.h`, `stream_functions.c`

- This library detects the stream type, reads PES headers, opens and closes files, gets their file size, and much more for most file types (except MP4 files, which have their own suite).
- This module, alongside file functions, makes up the backbone for TS, GXF, MXF, MythTV, WTV demuxing libraries and many more.
- This module includes data transfer libraries [1](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/libccxr_exports/demuxer.rs) [2](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/ctorust.rs) [3](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/common.rs) [4](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/libccxr_exports/demuxerdata.rs) for FFI between C and Rust.
- There's also a large-scale hyperlinked list library, code [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/hlist.rs). This is essentially a way to safely deal with multiple data streams in a single file.
- Code and extensive unit tests can be found [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/demuxer/demux.rs) and [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/demuxer/stream_functions.rs).

**High level Workflow of Demuxer, Stream Functions and File Functions:**

<img width="10762" height="2826" alt="demuxer" src="https://github.com/user-attachments/assets/4614c3a3-4b5c-4258-a0a0-5fcdeeba4be6" />

**Testing During Development** - These two modules are called in almost every file type (except MP4) for opening and switching to multiple files, so there isn't any specific file on which it was extensively checked. It was tested on multiple file types from the [sample platform](https://sampleplatform.ccextractor.org/sample).

---

### 3. Bitstream Module

**Files Ported:** `cc_bitstream.c`, `cc_bitstream.h`

- Primarily deals with bitstreams for MPEG-2 ES streams.
- A bitstream, in our context, is essentially a subtitle stream which is mapped to a packet in an ES stream extracted from an MPEG-2 TS file.

> **Source:** https://www.sciencedirect.com/topics/engineering/bitstream
> 
> The bitstream is organized as a succession of layers, each one being formed by a collection of packets. The layer gathers sets of compressed partial bit plane data from all the code-blocks of the different subbands and components of a tile.
> 
> The program stream is designed for applications where errors are unlikely. It contains audio, video, and data bitstreams (also called elementary bitstreams) all merged into a single bitstream. The program stream, as well as each of the elementary bitstreams, may be a fixed or variable bit-rate. DVDs and SVCDs use program streams, carrying the DVD- and SVCD-specific data in private data streams interleaved with the video and audio streams.
> 
> The transport stream, using fixed-size packets of 188 bytes, is designed for applications where data loss is likely. Also containing audio, video, and data bitstreams all merged into a single bitstream, multiple programs can be carried. The ARIB, ATSC, DVB and Open-Cable™ standards use transport streams.

- This module reads these packets, aligns them, factors in Golomb file compression, performs bit/byte-level reading, and much more.
- This module also includes a data transfer [library](https://github.com/CCExtractor/ccextractor/blob/master/src/rust/src/libccxr_exports/bitstream.rs) for FFI between C and Rust.
- Code and comprehensive unit tests can be found [here](https://github.com/CCExtractor/ccextractor/blob/master/src/rust/lib_ccxr/src/common/bitstream.rs).
- A few parts of this module were already implemented by a previous contributor @mrswastik-robot. I had to remake a significant portion of it (mostly editing the way the bitstream is traversed) and create the data transfer module.

---

### 4. Elementary Stream (ES) Module

**Files Ported:** `es_functions.c`, `es_userdata.c`

- This module extracts subtitles from an ES stream with help from the bitstream functions.
- The demuxer passes PES payloads to the ES processing entry, wrapped by the bitstream library for bit-level reading.
- The parser scans for MPEG start codes (sequence, GOP, picture, user data) and dispatches to corresponding handlers.
- Code can be found [here](https://github.com/steel-bucket/ccextractor/tree/migration-es-module/src/rust/src/es).

**High level Workflow of Bitstream and ES Functions:**

<img width="4904" height="2300" alt="es" src="https://github.com/user-attachments/assets/7c5aede7-215c-4149-ab57-2d0a57de9440" />

**Bitstream and ES Functions Were Primarily Tested During Development On:**

- Sample [#b22260](https://sampleplatform.ccextractor.org/sample/b22260d065ab537899baaf34e78a5184671f4bcb2df0414d05e6345adfd7812f)
- Sample [#127](https://sampleplatform.ccextractor.org/sample/127)
- Later, all other MPEG-2 TS libraries in the [sample platform](https://sampleplatform.ccextractor.org/sample)

---

### 5. GXF Module

**Files Ported:** `ccx_gxf.c`, `ccx_gxf.h`

- This library extracts subtitles from [General Exchange Format](https://en.wikipedia.org/wiki/General_Exchange_Format) files.

> **GXF Guide:** https://pub.smpte.org/pub/st360/st0360-2009_stable2016.pdf
> 
> The library first starts by probing the incoming bytes for the GXF packet 1MB starting array. If detected, it reads a fixed 16-byte packet header, which validates the packet type. The packet is then dispatched to one of five handlers: MAP, MEDIA, FLT, UMF, or EOS (essentially the 5 parts of a GXF stream). MAP packets contain file structure and track information, with the parser extracting media names, track start/end fields, and size, while allocating structures for ancillary and MPEG video tracks. These structures include frame rates, packet sizes, field counts, and timebase information for timestamping. MEDIA packets, containing track headers and payloads, are parsed differently based on track type. For ancillary tracks, the parser extracts SMPTE CDP caption words (CEA-708/608 paths) from AD packets, while MPEG video tracks are read into a buffer for parsing. In cases of errors, it attempts to skip remaining bytes and returns error codes (InvalidArgument, EOF, Retry). EOF and EOS are passed to the caller for file switching or demuxer stopping. Frame rate and timebase utilities convert track descriptors into timestamps for subtitle alignment.

- Code and comprehensive unit tests can be found [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/gxf_demuxer/gxf.rs).
- This module also includes a data transfer [library](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/libccxr_exports/gxf.rs) for FFI between C and Rust.

**Workflow of GXF Demuxing:**

<img width="8254" height="2418" alt="gxf" src="https://github.com/user-attachments/assets/4dfce685-8495-48fc-a5fd-5a7a3e51cb97" />

---

### 6. MXF Module

**Files Ported:** `ccx_demuxer_mxf.c`, `ccx_demuxer_mxf.h`

- This library extracts subtitles from [Material Exchange Format](https://en.wikipedia.org/wiki/Material_Exchange_Format) files.

> **MXF Guide:** https://tech.ebu.ch/docs/techreview/trev_2010-Q3_MXF-2.pdf
> 
> The library first starts by probing the incoming bytes(1mb) handed over by the demuxer. If it detects an MXF UL prefix, it reads the header partition pack to understand the file's structure and KLV (Key-Length-Value) alignment rules. It then reads the primer pack to learn how keys and local tags map to metadata sets, applying KLV Alignment Grid (KAG) rules to maintain proper alignment and skip filler. Next, it parses header metadata—such as preface, content storage, and tracks—to build a track map that links track IDs to essence streams and captures timing information. Index segments and the packets are read when available for fast seeking; if missing, partition offsets are used instead. The main loop scans for KLV packets, identifies essence types, and routes them to appropriate decoders. Subtitle data is extracted from essence or ancillary packets and timestamped.

- Code and comprehensive unit tests can be found [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/mxf_demuxer/mxf.rs).
- This module also includes a data transfer [library](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/libccxr_exports/mxf.rs) for FFI between C and Rust.

**Workflow of MXF Demuxing:**

<img width="3406" height="1978" alt="mxf" src="https://github.com/user-attachments/assets/1a59cdac-d86c-4b02-8d8f-a3e45480c4e7" />

**MXF Demuxer Was Primarily Tested During Development On:**

- Sample [#8f5d97](https://sampleplatform.ccextractor.org/sample/8f5d97c755b1bfb01e5fa736f54a600cb378e652ea1001d3fc7981fe9293c1d6)
- Sample [#057c1f](https://sampleplatform.ccextractor.org/sample/057c1fbc2d9f82691ae0b2150f95765a9c9d894ce1eb297229a05a242650b31a)
- Later, all other MPEG-2 TS libraries in the [sample platform](https://sampleplatform.ccextractor.org/sample)

---

### 7. MythTV Module

**Files Ported:** `myth.c`

> As usual, we start with probing the first 1MB and detecting that this stream is from a MythTV card. The MythTV test scans the probe buffer for repeated "tv0" or "TV0" sequences, treating the file as MythTV if matches exceed a threshold (e.g., 10 occurrences). This heuristic runs when auto-detection is enabled and prior checks leave the stream as elementary or program. Then the subtitles are extracted in the TS functions suite.

- Code and comprehensive unit tests can be found [here](https://github.com/steel-bucket/ccextractor/blob/migration-ts-core/src/rust/src/mythtv/myth.rs).

**Workflow of MythTV Detection:**

<img width="3220" height="2122" alt="myth" src="https://github.com/user-attachments/assets/8bf31539-ce45-49c3-a0ff-bc642f6ad475" />

**MythTV Was Primarily Tested During Development On:**

- Sample [#121](https://sampleplatform.ccextractor.org/sample/121)
- Later, all other MPEG-2 TS libraries in the [sample platform](https://sampleplatform.ccextractor.org/sample)

---

### 8. Encoder Module

**Files Ported:** `ccx_encoders_common.c`, `ccx_encoders_g608.c`

- An encoder in our context is a writer—a library that writes a demultiplexed subtitle stream into a file.
- I have kickstarted the work on the encoder library, creating the encoder for G608 and SimpleXML and fully porting the TXT encoder to Rust.
- I also implemented large parts of the base encoder in Rust, creating the foundation for future encoder libraries like SRT and MCC.
- Code can be found [here](https://github.com/CCExtractor/ccextractor/pull/1710/files).

---

### 9. Transport Stream (TS) Module

**Files Ported:** `ts_functions.c`, `ts_functions.h`, `ts_tables.c`, `ts_tables_epg.c`, `ts_info.c`

- This library ports 3 libraries to Rust. It's primarily for extracting subtitles from Transport Stream files, making up one of the most complex parts of CCExtractor. It took 3 weeks to complete.

> **From Wikipedia:** https://en.wikipedia.org/wiki/MPEG_transport_stream
> 
> Each stream is chopped into (at most) 188-byte sections and interleaved together. Due to the tiny packet size, streams can be interleaved with less latency and greater error resilience compared to program streams and other common containers such as AVI, MOV/MP4, and MKV, which generally wrap each frame into one packet. This is particularly important for videoconferencing, where large frames may introduce unacceptable audio delay.
> 
> TS files are mostly outputs of TV recordings, so working with them is more complex than other file types.
> 
> ![File:MPEG Transport Stream HL.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/b/ba/MPEG_Transport_Stream_HL.svg/800px-MPEG_Transport_Stream_HL.svg.png)
> 
> Our transport stream demuxer takes in a fixed 188-byte packet, locates the sync byte, and parses packet headers and adaptation fields to obtain the PID, continuity counter, payload unit start, and PCR timing. Payload bytes are routed by PID into two main reassembly paths: PSI section reassembly for tables (PMT tables, PAT tables, etc.) and PES reassembly for media. The PSI logic assembles sections (which we call PSI buffers), validates CRCs, parses PAT to find PMT PIDs, and parses PMTs to build a PID to elementary stream mapping including descriptors that flag subtitles or private data. PES reassembly uses payload unit start and continuity checking to form PES packets; PES headers yield PTS and DTS (essentially timing data). PCR samples from adaptation fields define a system clock used to correct PTS jitter and drift and to align audio and video. Subtitle extraction runs on PES or ancillary streams and supports CEA 608, CEA 708, DVB teletext and DVB subtitles, including CDP parsing when present. EPG is extracted from SI tables such as EIT and SDT, while XMLTV ingestion is a separate alternate workflow that bypasses PSI reassembly.

- Code can be found [here](https://github.com/steel-bucket/ccextractor/tree/migration-ts-core/src/rust/src/transportstream).

**High level Workflow of TS (excluding XMLTV which is separate):**

<img width="5422" height="1858" alt="ts" src="https://github.com/user-attachments/assets/372e4871-e5bc-4d7e-92e4-f1b43ab2104f" />

**TS Was Primarily Tested During Development On:**

- Sample [#96efd2](https://sampleplatform.ccextractor.org/sample/96efd279cfa1dddcb1d7d38ecc5ebd6d870a661452c6480804c30a9896037336)
- Sample [#b2260x](https://sampleplatform.ccextractor.org/sample/b22260d065ab537899baaf34e78a5184671f4bcb2df0414d05e6345adfd7812f)
- Sample [#c83f765](https://sampleplatform.ccextractor.org/sample/c83f765c661595e1bfa4750756a54c006c6f2c697a436bc0726986f71f0706cd)
- A [Hauppauge sample](https://www.dropbox.com/scl/fi/4b1y86efag39sjnmm65hs/all_in_with_chris_hayes_20250326_1958.ts?rlkey=tyid6blj5hvsbyhg1mxs9nvr8&st=557jrkq8&dl=0) submitted in the open issues by a user

---

### 10. Network Module

**Files Ported:** `networking.c`, `networking.h`

- This module mostly takes in a video stream from a TCP or UDP source instead of a file. It's primarily for real-time subtitle extraction (sort of like a socket). Much of the net module was created by previous contributors and had been in development for 2 years before this. I have completed it. The module was broken in both main and the old code. I had to remake the UDP implementation and fix numerous areas, and now the net module is working and fully ported to Rust. I've also written instructions in my PR for anyone who would work on this further.
- Code can be found [here](https://github.com/CCExtractor/ccextractor/pull/1725).

---

### 11. AVC Functions Module

**Files Ported:** `avc_functions.c`, `avc_functions.h`

- This module extracts subtitles from [Advanced Video Coding](https://en.wikipedia.org/wiki/Advanced_Video_Coding) streams. It was broken for the last few years because of a problem in the decoder module. I have fixed that and fully ported it to Rust.

> **From Wikipedia:**
> 
> Specified in Annex G of H.264/AVC, SVC allows the construction of bitstreams that contain layers of sub-bitstreams that also conform to the standard, including one such bitstream known as the "base layer" that can be decoded by an H.264/AVC codec that does not support SVC. For temporal bitstream scalability (i.e., the presence of a sub-bitstream with a smaller temporal sampling rate than the main bitstream), complete access units are removed from the bitstream when deriving the sub-bitstream. In this case, high-level syntax and inter-prediction reference pictures in the bitstream are constructed accordingly. On the other hand, for spatial and quality bitstream scalability (i.e., the presence of a sub-bitstream with lower spatial resolution/quality than the main bitstream), the NAL (Network Abstraction Layer) is removed from the bitstream when deriving the sub-bitstream.

- The primary subtitle extraction takes place after an SEI unit is extracted from an individual NAL unit; after that, it's simply data extraction from the SEI unit and storing the data in a buffer.
- Code can be found [here](https://github.com/steel-bucket/ccextractor/tree/migration-avc-module/src/rust/src/avc).
- This module also includes a data transfer [library](https://github.com/steel-bucket/ccextractor/blob/migration-avc-module/src/rust/src/avc/common_types.rs) for FFI.

**Workflow of AVC Functions:**

<img width="11552" height="2032" alt="avc" src="https://github.com/user-attachments/assets/010c3745-89da-4361-a7d3-b103d1509ac0" />

***Note*** - Videos of the demuxing and file processing modules working can be found below in the Flutter Report. 
---

## Other Contributions

- Removed the share module. This module was dysfunctional and outdated for quite some time. After discussion on Zulip, I removed it from all iterations of its use, the CI, and all build files. PR [#1737](https://github.com/CCExtractor/ccextractor/pull/1737)
- Removed all redundant C code and `DISABLE_RUST` options after discussion on Zulip. PR [#1738](https://github.com/CCExtractor/ccextractor/pull/1738)
- Remade the [Mac build script](https://github.com/steel-bucket/ccextractor/blob/removal-c-code/mac/build.command) as it was always building on Mac through `DISABLE_RUST`.
- Fixed regressions - PR [#1708](https://github.com/CCExtractor/ccextractor/pull/1708), fixed 134 codes in XDS and General.
- Fixed Clippy errors after the update to Rust v1.88. PR [#1706](https://github.com/CCExtractor/ccextractor/pull/1706)
- Fixed Rust unit tests failing after the update to Rust v1.86. PR [#1694](https://github.com/CCExtractor/ccextractor/pull/1694)
- Wrote 5 reports for mentors and future contributors (see below).

---
## Pull Requests

| Sno.       | Pull Request                                                  | Name                                                                    | Status     |
| ---------- | ------------------------------------------------------------- | ----------------------------------------------------------------------- | ---------- |
| 1          | [#1738](https://github.com/CCExtractor/ccextractor/pull/1738) | \[FEAT] Removed C code already ported to Rust                           | Open       |
| 2          | [#1737](https://github.com/CCExtractor/ccextractor/pull/1737) | \[FEAT] Removed the share module                                        | **Merged** |
| 3          | [#1736](https://github.com/CCExtractor/ccextractor/pull/1736) | \[Rust]Ported ES Module to Rust                                         | Open       |
| 4          | [#1730](https://github.com/CCExtractor/ccextractor/pull/1730) | \[Rust]Ported AVC Module to Rust                                        | Open       |
| 5          | [#1725](https://github.com/CCExtractor/ccextractor/pull/1725) | \[Rust] Fixes to Net Module                                             | Open       |
| 6          | [#1724](https://github.com/CCExtractor/ccextractor/pull/1724) | \[Rust] Finished the TS module                                          | Open       |
| 7          | [#1720](https://github.com/CCExtractor/ccextractor/pull/1720) | \[Rust]Added XMLTV library to Transport Stream module                   | Open       |
| 8          | [#1717](https://github.com/CCExtractor/ccextractor/pull/1717) | \[Rust]Added Transport Stream Module                                    | Open       |
| 9          | [#1713](https://github.com/CCExtractor/ccextractor/pull/1713) | \[Rust]Added MythTV Module                                              | Open       |
| 10         | [#1712](https://github.com/CCExtractor/ccextractor/pull/1712) | \[Rust] Added MXF module                                                | Open       |
| 11         | [#1711](https://github.com/CCExtractor/ccextractor/pull/1711) | \[Rust] Added GXF Module                                                | Open       |
| 12         | [#1710](https://github.com/CCExtractor/ccextractor/pull/1710) | \[FEAT]\[Rust] Added Encoder Module                                     | **Merged** |
| 13         | [#1708](https://github.com/CCExtractor/ccextractor/pull/1708) | \[FIX] 134 Codes in XDS and General Tests                               | **Merged** |
| 14         | [#1706](https://github.com/CCExtractor/ccextractor/pull/1706) | Fixed Clippy Errors on 1.88                                             | **Merged** |
| 15         | [#1694](https://github.com/CCExtractor/ccextractor/pull/1694) | \[FIX] Fixed Unit Test Rust based on the new changes on Rust 1.86.0     | **Merged** |
| 16         | [#1662](https://github.com/CCExtractor/ccextractor/pull/1662) | \[FEAT] added demuxer and file\_functions module                        | Open       |

---

## Work Reports

### Flutter GUI

This report examines whether there would be any caveats caused to the [CCExtractor Flutter GUI](https://github.com/CCExtractor/ccextractorfluttergui) due to my GSoC project. The final conclusion is that mostly no changes are needed—just a small change to the `ccextractor.dart` file in the GUI repository is required to get everything working.

https://docs.google.com/document/d/1bqLfD1yzIQndVWhsfFUxAeivDdHxhm4hz8xdCJZfN-U/edit?tab=t.0

### Open PRs Report

This report analyzes the open PRs as of August 2025. I checked most locally; for some, the regression tests were sufficient to draw a conclusion.

https://docs.google.com/document/d/1jOlBw3P3rUQlqZn2mifTJDFl8HREnXBHPqurFxknsik/edit?tab=t.0

Here's an older report I made a few months before:

https://docs.google.com/document/d/1UXfxaQ4wa9Ln9jjUo8pl6gnWVhLTCJcG7UfzsrmIuNU/edit?tab=t.0

### Issue Reports

I created these issue reports to make it easier for future contributors to work on the project and to ensure a smoother release.

#### Issue Report Jan 2024 - May 2025

https://docs.google.com/document/d/1f-TXDT4n4JIldaK1YI4kUFxPpKrJqXUpgIRAf4gEOog/edit?tab=t.0

#### Issue Report Jan 2023 - Dec 2023

https://docs.google.com/document/d/15VRTVIuigNVM990UVh_lc9o-zZNcBogBOPIGrV7o9M0/edit?tab=t.0

---

## Week-wise Report
These short reports were written along the way every week.

#### Week 1  
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

- Finished re‑writing the modules `demuxer` and `file_functions` to Rust. Wrote the new files `ctorust.rs`, `stream_functions.rs`, `hlist.rs` and `demuxer.rs`.
- Fixed all formatting and build issues, and wrote minor documentation.  
- Completed regression testing on Linux.
- Primarily, the basic logic and unit tests part was already implemented in May and April, which is what I was working on during the entire Community Bonding Period.  
- So, the first week was on testing regressions which failed for this particular module, writing documentation, and fixing all formatting and CI issues.  
- The Demuxer module does the important work of opening a file, so now ccextractor opens files, detects the stream type, closes files, gets the file size, and performs other tasks in Rust, which is a large step forward in the Rust conversion.  
- The file functions module was also tested and is working, and it is left for use in future modules like MythTV, MXF, GXF, etc.

#### Week 2  
[Pull Request #1649](https://github.com/CCExtractor/ccextractor/pull/1649) 
[Specific Changes I made to this branch](https://github.com/CCExtractor/ccextractor/pull/1649/commits/e04b43017138c3c1e94c245e9d1dd01de293dbed)<br>
AND  
[Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

- Re‑wrote the logic for the entire bitstream module using an index pointing to the current location of the pos pointer of the bitstream; this was required because all the tests for the MPEG‑2 TS files were failing in the bitstream module.
- Moved the FFI functions to the `libccxr_exports` directory, and re‑wrote the logic for the FFI using bindgen, as it was not used in the previous case.
- Resolved CI issues, tested bitstream for both Unix and Windows, and made sure that it worked akin to the main branch, I had to fix a bunch of different errors.
- Also worked on PR [#1662](https://github.com/CCExtractor/ccextractor/pull/1662), resolving all regression issues on Windows; these were not caught in Week 1 itself because the tests never ran due to the GPAC servers being down back then.
- After the GPAC servers were back up, resolved CI issues regarding Windows, tested all Windows regressions, and fixed several issues—especially catching a large error with the handles used in Windows to describe a file.
- Completed Regression Testing on Windows.
- Added more minor documentation to both modules.

#### Week 3  
[Pull Request #1](https://github.com/steel-bucket/ccextractor/pull/1) <br>
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

#### Week 9
[Pull Request #6](https://github.com/steel-bucket/ccextractor/pull/6) OR [Pull Request #1724](https://github.com/CCExtractor/ccextractor/pull/1724)<br>
Both are the same, one is for preview<br>
Some changes to [Pull Request #5](https://github.com/steel-bucket/ccextractor/pull/5)<br>
AND<br>
Review Changes to [Pull Request #1710](https://github.com/CCExtractor/ccextractor/pull/1710)

- Finally Finished the TS library, now all subtitles from TS files can be Extracted using Rust.
- Fixed Mac CI tests failing in TS module. Made sure all the CI tests are passing with Formatting and build issues.
- The TS library was tested against 4 TS files, 3 from the sample platform Sample [#96efd2](https://sampleplatform.ccextractor.org/sample/96efd279cfa1dddcb1d7d38ecc5ebd6d870a661452c6480804c30a9896037336), [#b2260x](https://sampleplatform.ccextractor.org/sample/b22260d065ab537899baaf34e78a5184671f4bcb2df0414d05e6345adfd7812f), [#c83f765](https://sampleplatform.ccextractor.org/sample/c83f765c661595e1bfa4750756a54c006c6f2c697a436bc0726986f71f0706cd) and a [hauppauge sample from dropbox](https://www.dropbox.com/scl/fi/4b1y86efag39sjnmm65hs/all_in_with_chris_hayes_20250326_1958.ts?rlkey=tyid6blj5hvsbyhg1mxs9nvr8&st=557jrkq8&dl=0)
- There are some issues still pertaining here. There are some weird symbols being printed out with the xmltv files(but the subs are extracted correctly and the exit code is the same as main) and the (-debug) release on linux sometimes gives drops the file descriptor. The other releases work fine.
- Finished the leftover functions from the stream_functions module from my original Demuxer PR.
- The last 3 week's work focuses on porting the files `ts_tables.c`,`ts_functions.c`,`ts_functions.h`,`ts_info.c`(some parts of it I didn't require),`ts_tables_epg.c`, and `stream_functions.c` to Rust.
- Cross Platform Tested on Windows as well and resolved type issues with the TS module.
- Note - The TS module is a bit slow because of the copying back and forth between rust and C every few chunks(basically the get_more_data implementation which works on most types of files), so maybe we could unplug it in production. We could replug it back to Rust after the core lib_ccx is made in Rust. 
- Fixed 2 of the Major issues with Week 8's PR, particularly the segfault with xmltv files for windows and the failing Mac test.
- Resolved all pertaining review comments in the Encoder PR, collaborating with my Mentor.

#### Week 10
[Pull Request #1725](https://github.com/CCExtractor/ccextractor/pull/1725) <br>
AND<br>
Review Changes to [Pull Request #1662](https://github.com/CCExtractor/ccextractor/pull/1662)

- Fixed the Net Module, it was dysfunctional in the main branch. I started by making the changes that were needed to be made in the main branch.
- Resolved all problems and failing regressions in the Rust port of the net module, which was in-development for the last 2 years.
- The Rust TCP implementation was fine, after the changes to main. But the UDP implementation needed to be re-written.
- Rebased the old PR, fixed all formatting and cross-platform issues in the CI. Made sure the regressions are akin to main.
- Made review changes to [#1662](https://github.com/CCExtractor/ccextractor/pull/1662), collaborating with my Mentor.

#### Week 11
[Pull Request #1730](https://github.com/CCExtractor/ccextractor/pull/1730)

- Fixed the AVC Functions Module, It was dysfunctional in the main branch and 0.94 because of a null pointer dereference in the rust decoder library.
- The `encoder_ctx` variable being empty was not taken into consideration which I fixed using a pre-check.
- Then I made the Rust port of the AVC functions module. Which was successfully ported to Rust.
- Resolved all failing CI, cross platform compatibility, and failing regressions in the new Rust port.
- The Rust port of the AVC functions module was tested against [This sample](https://drive.google.com/file/d/1GsxXE3EW9r9UfsRkoLlP2k03p6gLGbLe/view) which is an AVC stream.

#### Week 12
[Pull Request #1736](https://github.com/CCExtractor/ccextractor/pull/1736) <br>
[Pull Request #1737](https://github.com/CCExtractor/ccextractor/pull/1737) <br>
[Pull Request #1738](https://github.com/CCExtractor/ccextractor/pull/1738) <br>
AND <br>
[Pull Requests Report](https://docs.google.com/document/d/1jOlBw3P3rUQlqZn2mifTJDFl8HREnXBHPqurFxknsik/edit?usp=sharing)

- Ported the entire ES Functions module to Rust.
- This Rust module ports the C libraries `es_functions.c` and `es_userdata.c` to Rust.
- Resolved all failing CI, maintained cross platform compatibility, and failing regressions in the new Rust port.
- The Rust port of the ES functions module was tested against [This sample](https://sampleplatform.ccextractor.org/sample/b22260d065ab537899baaf34e78a5184671f4bcb2df0414d05e6345adfd7812f) which is an MPEG-2 ES stream.
- Resolved the share module - This module was used to make ccextractor work with cctranslate, it's been not updated for about 10 years, it was not working in both the ccextractor main and previous releases. So we decided to remove it. There's a rust port [which I made](https://github.com/CCExtractor/ccextractor/pull/1737/commits/90578eaf9dfe3dd0a3c469819317df449a2f97e1#diff-69ee6a0b9519e3ccd3b8b3dd12ec842d07f0b17de169836acb11ddcf573179cc), which could be merged into rust later if needed.
- Resolved all failing CI, and regressions which appeared while removing the share module.
- Made a PR to remove all redundant C code as discussed on zulip. Now all Rust ports are the functional ones.
- Re-made the entire Mac build process for CCExtractor. It always compiled on Mac with DISABLE_RUST on, this was caught during making the Removal PR and fixed accordingly.
- Reviewed all 11 Open Pull Requests from outside of the core team by running them locally and then compiled all the results in a [combined report](https://docs.google.com/document/d/1jOlBw3P3rUQlqZn2mifTJDFl8HREnXBHPqurFxknsik/edit?usp=sharing).

### Key Technologies Used

-   **Language**: Rust, C, Github Actions, 
-   **FFI**: Custom bindgen implementations
-   **Test Suite**: Unit Testing + Testing on Sample Platform(GCloud + Python)
-   **CI/CD**: GitHub Actions with support across Linux, Windows and Mac across all option permutations

## Work that's left
Primarily I have resolved every milestone in my Proposal. I still have to resolve merge conflicts that have accumulated in my PRs over time. The full Rust port is still a long way to go, but I have made a significantly large part of it. The next step for future contributors would be to work on WTV and the rest of the Encoder library. I'll also make a C to Rust conversion guide soon.

## Acknowledgments

I extend my heartfelt gratitude to my mentor Prateek Sunal, our org-admin Carlos Fernandez Sanz, my colleagues Hridesh MG and Vatsal Keshav and sample platform maintainer Willem Van Iseghem for all the help along the way. The experience working at CCExtractor Development was truly amazing and memorable.


_This report summarizes 12 weeks of intensive development, testing, and collaboration in migrating CCExtractor's core libraries to Rust. The project continues to evolve, and I remain committed to its success beyond the GSoC program._
