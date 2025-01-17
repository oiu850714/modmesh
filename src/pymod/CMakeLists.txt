# Copyright (c) 2019, Yung-Yu Chen <yyc@solvcon.net>
# BSD-style license; see COPYING

cmake_minimum_required(VERSION 3.16)
project(pymod)

set(MODMESH_SOURCES
    modmesh.cpp
)
pybind11_add_module(
    _modmesh
    ${MODMESH_SOURCES}
    ${MODMESH_HEADERS}
    ${MODMESH_PY_HEADERS}
)
if(HIDE_SYMBOL)
    set_target_properties(_modmesh PROPERTIES CXX_VISIBILITY_PRESET "hidden")
else()
    set_target_properties(_modmesh PROPERTIES CXX_VISIBILITY_PRESET "default")
endif()

target_compile_options(
    _modmesh PRIVATE
    ${COMMON_COMPILER_OPTIONS}
    -Wno-unused-value # for PYBIND11_EXPAND_SIDE_EFFECTS in pybind11.h
    -Wno-noexcept-type # GCC
)

if(CLANG_TIDY_EXE AND USE_CLANG_TIDY)
    set_target_properties(
        _modmesh PROPERTIES
        CXX_CLANG_TIDY "${DO_CLANG_TIDY}"
    )
endif()

execute_process(
    COMMAND python3-config --extension-suffix
    OUTPUT_VARIABLE PYEXTSUFFIX)

add_custom_target(_modmesh_py
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:_modmesh> ${MODMESH_PY_DIR}/_modmesh${PYEXTSUFFIX}
    DEPENDS _modmesh)

message(STATUS "CMAKE_INSTALL_INCLUDEDIR: ${CMAKE_INSTALL_INCLUDEDIR}")
install(DIRECTORY ${MODMESH_INCLUDE_DIR}/modmesh DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

add_custom_target(flake8)
flake8("flake8")

# vim: set ff=unix fenc=utf8 nobomb et sw=4 ts=4 sts=4:
