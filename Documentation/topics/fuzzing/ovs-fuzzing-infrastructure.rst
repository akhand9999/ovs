..
      Copyright (c) 2016, Stephen Finucane <stephen@that.guru>

      Licensed under the Apache License, Version 2.0 (the "License"); you may
      not use this file except in compliance with the License. You may obtain
      a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
      WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
      License for the specific language governing permissions and limitations
      under the License.

      Convention for heading levels in Open vSwitch documentation:

      =======  Heading 0 (reserved for the title in a document)
      -------  Heading 1
      ~~~~~~~  Heading 2
      +++++++  Heading 3
      '''''''  Heading 4

      Avoid deeper levels because they do not render well.

==========================
OVS Fuzzing Infrastructure
==========================

Open vSwitch is enrolled in the oss-fuzz program. oss-fuzz is a free continuous
fuzzing service and infrastructure offered by Google. An enrolled project is
continually fuzzed and bug reports are sent to maintainers as and when they
are generated.

The technical requirements to enrol OvS in oss-fuzz program are:
  * Dockerfile (hosted in Google's ossfuzz GitHub repo)
  * Bash build script (hosted in Google's ossfuzz GitHub repo)

Each of these requirements is explained in the following paragraphs.

-----------
Dockerfile
-----------

Dockerfile defines the box in which OvS will be built by oss-fuzz builder bots.
This file must be self sufficient in that it must define a build environment in
which OvS can be built from source code.

The build environment comprises

  * Linux box provided by Google (Ubuntu)

  * Packages required to build OvS (e.g., libssl-dev python etc.)

  * Source code of OvS (fetched by oss-fuzz builder bots from OvS'
    GitHub mirror on a daily basis)

  * Build script written in Bash

The Dockerfile definition for OvS is located in the
`projects/openvswitch/Dockerfile` sub-directory of Google's oss-fuzz repo.

Build script
------------

The build script defines steps required to compile OvS from source code.
The (Linux) box used by oss-fuzz builder bots (defined by Dockerfile) is
different from the box in which fuzzing actually happens. It follows that
the build script must ensure that fuzzing binaries are linked statically so
that no assumption is made about packages available in the fuzzing box.

OvS contains a make target called `oss-fuzz-targets` for compiling and linking
OvS fuzzer binaries. The line of bash script responsible for building
statically linked OvS fuzzing binaries is the following::

  ./boot.sh && ./configure && make -j$(nproc) && make oss-fuzz-targets

The oss-fuzz build environment assumes that OvS build system respects
compiler/linker flags defined via standard bash environment variables called
`CFLAGS`, `CXXFLAGS` etc. The oss-fuzz builder bot defines these flags so
that OvS fuzzing binaries are correctly instrumented.

oss-fuzz expects all fuzzing binaries, and optionally, configuration and
seed inputs to be located in a hard-coded directory, referenced by the bash
variable `$OUT`, in the root filesystem of the build box.

OvS source repo contains configuration for the oss-fuzz fuzzers in the
`tests/oss-fuzz/config` sub-directory. There are two types of configuration
files:

  * <fuzzer_target_name>.options: Defines configuration options for the fuzzer
    that apply while fuzzing `fuzzer_target_name`

  * <name>.dict: Defines a dictionary to be used for some `fuzzer_target_name`

Fuzzer configuration parameters of relevance to OvS are:

  * dict: names the dictionary file to be used for fuzzing

  * close_fd_mask: defines a file descriptor mask that filters console output
    generated by OvS fuzzing binaries
