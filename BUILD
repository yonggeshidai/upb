load(
    ":build_defs.bzl",
    "generated_file_staleness_test",
    "licenses",  # copybara:strip_for_google3
    "lua_binary",
    "lua_cclibrary",
    "lua_library",
    "lua_test",
    "make_shell_script",
    "map_dep",
    "upb_amalgamation",
    "upb_proto_library",
    "upb_proto_reflection_library",
    "upb_proto_srcs",
)

licenses(["notice"])  # BSD (Google-authored w/ possible external contributions)

exports_files([
    "LICENSE",
    "build_defs",
])

CPPOPTS = [
    # copybara:strip_for_google3_begin
    "-Werror",
    "-Wno-long-long",
    # copybara:strip_end
]

COPTS = CPPOPTS + [
    # copybara:strip_for_google3_begin
    "-pedantic",
    "-std=c89",
    # copybara:strip_end
]

config_setting(
    name = "darwin",
    values = {"cpu": "darwin"},
    visibility = ["//visibility:public"],
)

# Public C/C++ libraries #######################################################

cc_library(
    name = "upb",
    srcs = [
        "upb/decode.c",
        "upb/encode.c",
        "upb/msg.c",
        "upb/port_def.inc",
        "upb/port_undef.inc",
        "upb/table.c",
        "upb/table.int.h",
        "upb/upb.c",
    ],
    hdrs = [
        "upb/decode.h",
        "upb/encode.h",
        "upb/generated_util.h",
        "upb/msg.h",
        "upb/upb.h",
    ],
    copts = COPTS,
    visibility = ["//visibility:public"],
)

upb_proto_library(
    name = "descriptor_upbproto",
    deps = ["@com_google_protobuf//:descriptor_proto"],
)

cc_library(
    name = "reflection",
    srcs = [
        "upb/def.c",
        "upb/msgfactory.c",
    ],
    hdrs = [
        "upb/def.h",
        "upb/msgfactory.h",
    ],
    copts = COPTS,
    visibility = ["//visibility:public"],
    deps = [
        ":descriptor_upbproto",
        ":upb"
    ],
)

# Internal C/C++ libraries #####################################################

cc_library(
    name = "table",
    hdrs = ["upb/table.int.h"],
    deps = [":upb"],
)

# Legacy C/C++ Libraries (not recommended for new code) ########################

cc_library(
    name = "legacy_msg_reflection",
    srcs = [
        "upb/legacy_msg_reflection.c",
    ],
    hdrs = ["upb/legacy_msg_reflection.h"],
    deps = [":upb"],
    copts = COPTS,
)

cc_library(
    name = "handlers",
    srcs = [
        "upb/handlers.c",
        "upb/handlers-inl.h",
        "upb/sink.c",
    ],
    hdrs = [
        "upb/handlers.h",
        "upb/sink.h",
    ],
    copts = COPTS,
    deps = [
        ":reflection",
        ":upb",
    ],
)

cc_library(
    name = "upb_pb",
    srcs = [
        "upb/pb/compile_decoder.c",
        "upb/pb/decoder.c",
        "upb/pb/decoder.int.h",
        "upb/pb/encoder.c",
        "upb/pb/textprinter.c",
        "upb/pb/varint.c",
        "upb/pb/varint.int.h",
    ],
    hdrs = [
        "upb/pb/decoder.h",
        "upb/pb/encoder.h",
        "upb/pb/textprinter.h",
    ],
    copts = COPTS,
    deps = [
        ":handlers",
        ":table",
        ":upb",
    ],
)

cc_library(
    name = "upb_json",
    srcs = [
        "upb/json/parser.c",
        "upb/json/printer.c",
    ],
    hdrs = [
        "upb/json/parser.h",
        "upb/json/printer.h",
    ],
    copts = COPTS,
    deps = [
        ":upb",
        ":upb_pb",
    ],
)

cc_library(
    name = "upb_cc_bindings",
    hdrs = [
        "upb/bindings/stdc++/string.h",
    ],
    deps = [":upb"],
)

# upb compiler #################################################################

cc_library(
    name = "upbc_generator",
    srcs = [
        "upbc/generator.cc",
        "upbc/message_layout.cc",
        "upbc/message_layout.h",
    ],
    hdrs = ["upbc/generator.h"],
    deps = [
        map_dep("@absl//absl/base:core_headers"),
        map_dep("@absl//absl/strings"),
        map_dep("@com_google_protobuf//:protobuf"),
        map_dep("@com_google_protobuf//:protoc_lib"),
    ],
    copts = CPPOPTS,
)

cc_binary(
    name = "protoc-gen-upb",
    srcs = ["upbc/main.cc"],
    deps = [
        ":upbc_generator",
        map_dep("@com_google_protobuf//:protoc_lib"),
    ],
    copts = CPPOPTS,
    visibility = ["//visibility:public"],
)

# We strip the tests and remaining rules from google3 until the upb_proto_library()
# and upb_proto_reflection_library() rules are fixed.

# copybara:strip_for_google3_begin

# C/C++ tests ##################################################################

cc_library(
    name = "upb_test",
    testonly = 1,
    srcs = [
        "tests/testmain.cc",
    ],
    hdrs = [
        "tests/test_util.h",
        "tests/upb_test.h",
    ],
    copts = CPPOPTS,
)

cc_test(
    name = "test_varint",
    srcs = ["tests/pb/test_varint.c"],
    deps = [
        ":upb_pb",
        ":upb_test",
    ],
    copts = COPTS,
)

proto_library(
    name = "test_decoder_proto",
    srcs = [
        "tests/pb/test_decoder.proto",
    ],
)

upb_proto_reflection_library(
    name = "test_decoder_upbproto",
    deps = ["test_decoder_proto"],
)

cc_test(
    name = "test_decoder",
    srcs = ["tests/pb/test_decoder.cc"],
    deps = [
        ":test_decoder_upbproto",
        ":upb_pb",
        ":upb_test",
    ],
    copts = CPPOPTS,
)

upb_proto_reflection_library(
    name = "descriptor_upbreflection",
    deps = ["@com_google_protobuf//:descriptor_proto"],
)

cc_test(
    name = "test_encoder",
    srcs = ["tests/pb/test_encoder.cc"],
    deps = [
        "descriptor_upbreflection",
        ":upb_cc_bindings",
        ":upb_pb",
        ":upb_test",
    ],
    copts = CPPOPTS,
)

proto_library(
    name = "test_cpp_proto",
    srcs = [
        "tests/test_cpp.proto",
    ],
)

upb_proto_reflection_library(
    name = "test_cpp_upbproto",
    deps = ["test_cpp_proto"],
)

cc_test(
    name = "test_cpp",
    srcs = ["tests/test_cpp.cc"],
    deps = [
        ":test_cpp_upbproto",
        ":upb",
        ":upb_pb",
        ":upb_test",
    ],
    copts = CPPOPTS,
)

cc_test(
    name = "test_table",
    srcs = ["tests/test_table.cc"],
    deps = [
        ":upb",
        ":upb_test",
    ],
    copts = CPPOPTS,
)

proto_library(
    name = "test_json_enum_from_separate",
    srcs = ["tests/json/enum_from_separate_file.proto"],
    deps = [":test_json_proto"],
)

proto_library(
    name = "test_json_proto",
    srcs = ["tests/json/test.proto"],
)

upb_proto_reflection_library(
    name = "test_json_upbprotoreflection",
    deps = ["test_json_proto"],
)

upb_proto_library(
    name = "test_json_enum_from_separate_upbproto",
    deps = [":test_json_enum_from_separate"],
)

upb_proto_library(
    name = "test_json_upbproto",
    deps = [":test_json_proto"],
)

cc_test(
    name = "test_json",
    srcs = [
        "tests/json/test_json.cc",
    ],
    deps = [
        ":test_json_upbprotoreflection",
        ":test_json_upbproto",
        ":upb_json",
        ":upb_test",
    ],
    copts = CPPOPTS,
)

upb_proto_library(
    name = "conformance_proto_upb",
    deps = ["@com_google_protobuf//:conformance_proto"],
)

upb_proto_library(
    name = "test_messages_proto3_proto_upb",
    deps = ["@com_google_protobuf//:test_messages_proto3_proto"],
)

cc_binary(
    name = "conformance_upb",
    srcs = [
        "tests/conformance_upb.c",
    ],
    copts = COPTS + ["-Ibazel-out/k8-fastbuild/bin"],
    deps = [
        ":conformance_proto_upb",
        ":test_messages_proto3_proto_upb",
        ":upb",
    ],
)

make_shell_script(
    name = "gen_test_conformance_upb",
    out = "test_conformance_upb.sh",
    contents = "$(rlocation com_google_protobuf/conformance_test_runner) $(rlocation upb/conformance_upb)",
)

sh_test(
    name = "test_conformance_upb",
    srcs = ["test_conformance_upb.sh"],
    data = [
        "tests/conformance_upb_failures.txt",
        ":conformance_upb",
        "@bazel_tools//tools/bash/runfiles",
        "@com_google_protobuf//:conformance_test_runner",
    ],
)

# Amalgamation #################################################################

py_binary(
    name = "amalgamate",
    srcs = ["tools/amalgamate.py"],
)

upb_proto_srcs(
    name = "descriptor_upbproto_srcs",
    deps = ["@com_google_protobuf//:descriptor_proto"],
)

upb_amalgamation(
    name = "gen_amalgamation",
    outs = [
        "upb.c",
        "upb.h",
    ],
    amalgamator = ":amalgamate",
    libs = [
        ":upb",
        ":descriptor_upbproto_srcs",
        ":reflection",
        ":handlers",
        ":upb_pb",
        ":upb_json",
    ],
)

cc_library(
    name = "amalgamation",
    srcs = ["upb.c"],
    hdrs = ["upb.h"],
    copts = COPTS,
)

# Lua libraries. ###############################################################

lua_cclibrary(
    name = "lua/upb_c",
    srcs = [
        "upb/bindings/lua/def.c",
        "upb/bindings/lua/msg.c",
        "upb/bindings/lua/upb.c",
    ],
    hdrs = [
        "upb/bindings/lua/upb.h",
    ],
    deps = [
        "legacy_msg_reflection",
        "upb",
        "upb_pb",
    ],
)

lua_library(
    name = "lua/upb",
    srcs = ["upb/bindings/lua/upb.lua"],
    luadeps = ["lua/upb_c"],
    strip_prefix = "upb/bindings/lua",
)

lua_cclibrary(
    name = "lua/upb/pb_c",
    srcs = ["upb/bindings/lua/upb/pb.c"],
    luadeps = ["lua/upb_c"],
    deps = ["upb_pb"],
)

lua_library(
    name = "lua/upb/pb",
    srcs = ["upb/bindings/lua/upb/pb.lua"],
    luadeps = [
        "lua/upb",
        "lua/upb/pb_c",
    ],
    strip_prefix = "upb/bindings/lua",
)

# Lua tests. ###################################################################

lua_test(
    name = "lua/test_upb",
    luadeps = ["lua/upb"],
    luamain = "tests/bindings/lua/test_upb.lua",
)

lua_test(
    name = "lua/test_upb_pb",
    luadeps = ["lua/upb/pb"],
    luamain = "tests/bindings/lua/test_upb.pb.lua",
)

# Test the CMake build #########################################################

filegroup(
    name = "cmake_files",
    srcs = glob([
        "CMakeLists.txt",
        "generated_for_cmake/**/*",
        "google/**/*",
        "upbc/**/*",
        "upb/**/*",
        "tests/**/*",
    ])
)

make_shell_script(
    name = "gen_run_cmake_build",
    out = "run_cmake_build.sh",
    contents = "find . && mkdir build && cd build && cmake .. && make -j8 && make test",
)

sh_test(
    name = "cmake_build",
    srcs = ["run_cmake_build.sh"],
    data = [
        ":cmake_files",
        "@bazel_tools//tools/bash/runfiles",
    ],
)

# Generated files ##############################################################

exports_files(["tools/staleness_test.py"])

py_library(
    name = "staleness_test_lib",
    testonly = 1,
    srcs = ["tools/staleness_test_lib.py"],
)

py_binary(
    name = "make_cmakelists",
    srcs = ["tools/make_cmakelists.py"],
)

genrule(
    name = "gen_cmakelists",
    srcs = [
        "BUILD",
        "WORKSPACE",
        ":cmake_files",
    ],
    outs = ["generated-in/CMakeLists.txt"],
    cmd = "$(location :make_cmakelists) $@",
    tools = [":make_cmakelists"],
)

genrule(
    name = "generate_json_ragel",
    srcs = ["upb/json/parser.rl"],
    outs = ["upb/json/parser.c"],
    cmd = "$(location @ragel//:ragel) -C -o upb/json/parser.c $< && mv upb/json/parser.c $@",
    tools = ["@ragel"],
)

genrule(
    name = "copy_json_ragel",
    srcs = ["upb/json/parser.c"],
    outs = ["generated-in/generated_for_cmake/upb/json/parser.c"],
    cmd = "cp $< $@",
)

genrule(
    name = "copy_protos",
    srcs = [":descriptor_upbproto_srcs"],
    outs = [
        "generated-in/generated_for_cmake/google/protobuf/descriptor.upb.c",
        "generated-in/generated_for_cmake/google/protobuf/descriptor.upb.h",
    ],
    cmd = "cp $(SRCS) $(@D)/generated-in/generated_for_cmake/google/protobuf",
)

generated_file_staleness_test(
    name = "test_generated_files",
    outs = [
        "CMakeLists.txt",
        "generated_for_cmake/upb/json/parser.c",
        "generated_for_cmake/google/protobuf/descriptor.upb.c",
        "generated_for_cmake/google/protobuf/descriptor.upb.h",
    ],
    generated_pattern = "generated-in/%s",
)

# copybara:strip_end
