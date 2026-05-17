cmake_minimum_required(VERSION 3.15)

project(kobodeluxe
    VERSION 0.6.0
    DESCRIPTION "Arcade style 2D shooter"
    LANGUAGES C CXX)

set(KOBO_ORGANIZATION "OlofsonArcade")
set(KOBO_APPLICATION  "KoboDeluxe")

set(VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(VERSION_BUILD 0)
set(VERSION_STRING "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set(PACKAGE "kobodeluxe-${VERSION_STRING}")

# ---------------------------------------------------------------------------
# Options
# ---------------------------------------------------------------------------
option(WIN32_ZIP_BUILD "Build flat Win32 ZIP archive." OFF)
set(WORKDIR "${PROJECT_SOURCE_DIR}/build" CACHE PATH "Top level build/staging directory.")
set(KOBO_EXTRA_LIBRARIES "" CACHE STRING "Extra libraries to link with.")

message(STATUS ".-----------------------------------------")
message(STATUS "|      WIN32_ZIP_BUILD: ${WIN32_ZIP_BUILD}")
message(STATUS "|              WORKDIR: ${WORKDIR}")
message(STATUS "| KOBO_EXTRA_LIBRARIES: ${KOBO_EXTRA_LIBRARIES}")
message(STATUS "'-----------------------------------------")

# ---------------------------------------------------------------------------
# Install layout
# ---------------------------------------------------------------------------
if(WIN32 AND WIN32_ZIP_BUILD)
    set(KOBO_BIN_DIR   "/")
    set(KOBO_SHARE_DIR "/")
    set(KOBO_DOC_DIR   "/")
elseif(APPLE)
    # For a macOS .app bundle the binary lives inside the bundle; a separate
    # prefix-relative bin/ is not meaningful.  Data and docs go alongside it.
    set(KOBO_BIN_DIR   ".")
    set(KOBO_SHARE_DIR ".")
    set(KOBO_DOC_DIR   ".")
else()
    set(KOBO_BIN_DIR   "bin")
    set(KOBO_SHARE_DIR "share/kobodeluxe")
    set(KOBO_DOC_DIR   "share/doc/kobodeluxe")
endif()

# ---------------------------------------------------------------------------
# Compiler flags (shared between C and C++ unless noted)
# ---------------------------------------------------------------------------
set(KOBO_FLAGS_RELEASE  "-O3 -DNDEBUG")
set(KOBO_FLAGS_MAINTAIN "-O0 -g -Wall -Wwrite-strings -Wcast-align -Wshadow -Wdisabled-optimization")

set(CMAKE_C_FLAGS_RELEASE          "${KOBO_FLAGS_RELEASE}")
set(CMAKE_CXX_FLAGS_RELEASE        "${KOBO_FLAGS_RELEASE}")
set(CMAKE_C_FLAGS_MAINTAINER       "${KOBO_FLAGS_MAINTAIN} -Wbad-function-cast -Wstrict-prototypes -Wmissing-prototypes -Wmissing-declarations -Werror")
set(CMAKE_CXX_FLAGS_MAINTAINER     "${KOBO_FLAGS_MAINTAIN} -Wmissing-declarations -Werror")
set(CMAKE_C_FLAGS_DEBUG            "${KOBO_FLAGS_MAINTAIN} -DDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG          "${KOBO_FLAGS_MAINTAIN} -DDEBUG")

# ---------------------------------------------------------------------------
# Subdirectories
# ---------------------------------------------------------------------------
add_subdirectory(src)

# ---------------------------------------------------------------------------
# Game data
# ---------------------------------------------------------------------------
if(APPLE)
    # On macOS, install data inside the bundle next to the binary so the
    # app is self-contained and can be dragged to /Applications.
    install(DIRECTORY data/gfx
        DESTINATION "kobodl.app/Contents/MacOS"
        FILES_MATCHING
            PATTERN "*.png"
            PATTERN "*.gpl"
            PATTERN "*.theme")
    install(DIRECTORY data/sfx
        DESTINATION "kobodl.app/Contents/MacOS"
        FILES_MATCHING
            PATTERN "*.a2s"
            PATTERN "*.theme")
else()
    install(DIRECTORY data/gfx
        DESTINATION "${KOBO_SHARE_DIR}"
        FILES_MATCHING
            PATTERN "*.png"
            PATTERN "*.gpl"
            PATTERN "*.theme")
    install(DIRECTORY data/sfx
        DESTINATION "${KOBO_SHARE_DIR}"
        FILES_MATCHING
            PATTERN "*.a2s"
            PATTERN "*.theme")
endif()

# ---------------------------------------------------------------------------
# Documentation
# ---------------------------------------------------------------------------
file(GLOB doc_files
    "${PROJECT_SOURCE_DIR}/*.md"
    "${PROJECT_SOURCE_DIR}/COPYING*")
list(REMOVE_ITEM doc_files "${PROJECT_SOURCE_DIR}/INSTALL.md")
install(FILES ${doc_files} DESTINATION "${KOBO_DOC_DIR}")

# ---------------------------------------------------------------------------
# Desktop integration (Linux only)
# ---------------------------------------------------------------------------
if(NOT WIN32 AND NOT APPLE)
    install(FILES "${PROJECT_SOURCE_DIR}/icons/kobodeluxe.png"
        DESTINATION "share/pixmaps"
        RENAME "kobodeluxe.png")
    install(FILES "${PROJECT_SOURCE_DIR}/icons/net.olofson.kobodeluxe.desktop"
        DESTINATION "share/applications")
endif()

# ---------------------------------------------------------------------------
# Win32 redistributables
# ---------------------------------------------------------------------------
if(WIN32)
    file(GLOB win32_redist_files "${WORKDIR}/win32-redist/*")
    install(FILES ${win32_redist_files} DESTINATION "${KOBO_BIN_DIR}")
endif()

# ---------------------------------------------------------------------------
# CPack
# ---------------------------------------------------------------------------
include(GNUInstallDirs)

set(CPACK_PACKAGE_NAME              "kobodeluxe")
set(CPACK_PACKAGE_VERSION           "${VERSION_STRING}")
set(CPACK_PACKAGE_CONTACT           "David Olofson <david@olofson.net>")
set(CPACK_RESOURCE_FILE_LICENSE     "${PROJECT_SOURCE_DIR}/COPYING")
set(CPACK_RESOURCE_FILE_README      "${PROJECT_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Arcade style 2D shooter")
set(CPACK_STRIP_FILES               TRUE)

if(WIN32)
    set(CPACK_GENERATOR "ZIP")
    set(CPACK_SET_DESTDIR               ON)
    set(CPACK_PACKAGE_RELOCATABLE       FALSE)
elseif(APPLE)
    # Bundle SDL2 and other Homebrew dylibs into the .app before packaging.
    install(CODE "
        include(BundleUtilities)
        fixup_bundle(
            \"\${CMAKE_INSTALL_PREFIX}/kobodl.app\"
            \"\"
            \"/opt/homebrew/lib;/opt/homebrew/opt/sdl2/lib;/opt/homebrew/opt/sdl2_image/lib\"
        )
    " COMPONENT Runtime)
    set(CPACK_GENERATOR                 "DragNDrop")
    set(CPACK_DMG_VOLUME_NAME           "Kobo Deluxe ${VERSION_STRING}")
    set(CPACK_PACKAGE_RELOCATABLE       TRUE)
else()
    set(CPACK_GENERATOR "DEB;TGZ")
    set(CPACK_SET_DESTDIR               ON)
    set(CPACK_PACKAGE_RELOCATABLE       FALSE)
    set(CPACK_DEBIAN_PACKAGE_SECTION    "games")
    set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS  ON)
    set(CMAKE_INSTALL_RPATH             "${CMAKE_INSTALL_PREFIX}/lib")
endif()

include(CPack)
