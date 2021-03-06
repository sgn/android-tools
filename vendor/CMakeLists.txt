# TODO: https://cmake.org/cmake/help/v3.0/module/ExternalProject.html
# To patch the submodule

add_subdirectory(boringssl EXCLUDE_FROM_ALL)
set(android-vendored
	avb
	core
	extras
	selinux
	f2fs-tools
	e2fsprogs)

if(EXISTS "${ANDROID_PATCH_DIR}/")
	execute_process(COMMAND git submodule --quiet update)
	foreach(v ${android-vendored})
		file(GLOB patches ${ANDROID_PATCH_DIR}/${v}/*.patch)
		if(patches)
			message(STATUS "Applying patches for: ${v}")
			execute_process(COMMAND git -C
				${CMAKE_CURRENT_SOURCE_DIR}/${v} am ${patches}
				RESULT_VARIABLE ret)
			if(NOT "${ret}" STREQUAL "0")
				message(FATAL_ERROR "Couldn't apply patches for ${v}")
			endif()
		endif(patches)
	endforeach(v)
endif()

include(CMakeLists.adb.txt)
include(CMakeLists.sparse.txt)
include(CMakeLists.fastboot.txt)
include(CMakeLists.mke2fs.txt)

# Various C++20 features are used across the codebase, e.g.
# std::string_view.starts_with. Additionally, GNU extension
# (such as typeof) are used occasionally.
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu11")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++2a")

# Android seems to use various attributes supported by clang but not by
# GCC which causes it to emit lots of warnings. Since these attributes
# don't seem to effect runtime behaviour simply disable the warnings.
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-attributes")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-attributes")

# libfsmgr (required by fastboot) requires a 64-bit off_t for lseek. On
# 32-bit glibc platforms this is not the case by default.
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FILE_OFFSET_BITS=64")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_FILE_OFFSET_BITS=64")

# Targets which should be installed by `make install`.
install(TARGETS adb fastboot "${ANDROID_MKE2FS_NAME}"
	simg2img img2simg append2simg DESTINATION bin)
