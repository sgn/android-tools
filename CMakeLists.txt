project(android-tools)
cmake_minimum_required(VERSION 3.1.0)

# Android provides it's own version of mke2fs which is incompatible with
# the version shipped by e2fsprogs. To prevent a name clash we install
# androids version of mke2fs under a different name. This name can be
# configured here.
#
# See also: https://bugs.archlinux.org/task/56955
set(ANDROID_MKE2FS_NAME "mke2fs.android")

# Version of android-tools and the version of boringssl being used.
# See: https://android.googlesource.com/platform/external/boringssl/+/platform-tools-${ANDROID_VERSION}/BORINGSSL_REVISION
set(ANDROID_VERSION 29.0.6)
set(BORINGSSL_VERSION 7f02881e96e51f1873afcf384d02f782b48967ca)

# Vendor string used in version outputs.
set(ANDROID_VENDOR android-tools)

# The CMakeLists.txt in the vendor subdirectory is responsible for
# patching, building and installing the software.
set(ANDROID_PATCH_DIR ${CMAKE_CURRENT_SOURCE_DIR}/patches)
add_subdirectory(vendor)

# CPack configuration for creating source tarballs which already include
# patched versions of the vendored dependencies.
set(CPACK_SOURCE_GENERATOR "TXZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${PROJECT_NAME}-${ANDROID_VERSION}")
set(CPACK_SOURCE_IGNORE_FILES "/patches/" "/build/" "/.git/" "/tests/"
	"/testdata/" "/extras/simpleperf/scripts/" "\\\\.so$" "\\\\.zip$"
	"/fuzz/" "\\\\.tar$" "\\\\.tar\\\\..*$" "\\\\.tgz$" "\\\\.data$")
include(CPack)
