load("@rules_foreign_cc//tools/build_defs/native_tools:native_tools_toolchain.bzl", "native_tool_toolchain")

toolchain(
    name = "built_jmake",
    toolchain = ":jmake_tool",
    toolchain_type = "@rules_foreign_cc//tools/build_defs:make_toolchain",
)

native_tool_toolchain(
    name = "jmake_tool",
    path = "jmake",
    target = ":jmake_all",
    visibility = ["//visibility:public"],
)

filegroup(
    name = "jmake_all",
    srcs = [ "jmake" ],
)
