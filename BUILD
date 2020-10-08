load("@rules_foreign_cc//tools/build_defs/native_tools:native_tools_toolchain.bzl", "native_tool_toolchain")

toolchain(
    name = "built_sjsmake",
    toolchain = ":sjsmake_tool",
    toolchain_type = "@rules_foreign_cc//tools/build_defs:make_toolchain",
)

native_tool_toolchain(
    name = "sjsmake_tool",
    path = "sjsmake",
    target = ":sjsmake_all",
    visibility = ["//visibility:public"],
)

filegroup(
    name = "sjsmake_all",
    srcs = [ "sjsmake" ],
)
