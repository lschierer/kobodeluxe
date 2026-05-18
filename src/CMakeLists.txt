set(sources
    gamestate.cpp
    map.cpp
    radar.cpp
    cfgform.cpp
    dashboard.cpp
    enemy.cpp
    game.cpp
    logger.c
    pfile.cpp
    score.cpp
    sound.cpp
    filemap.cpp
    myship.cpp
    random.cpp
    cfgparse.cpp
    gamectl.cpp
    kobo.cpp
    manage.cpp
    prefs.cpp
    screen.cpp
    states.cpp
    enemies.cpp
    form.cpp
    options.cpp
    scenes.cpp
    eel/e_builtin.c
    eel/e_getargs.c
    eel/e_lexer.c
    eel/e_path.c
    eel/e_register.c
    eel/e_script.c
    eel/e_symtab.c
    eel/e_util.c
    eel/eel.c
    graphics/cs.c
    graphics/display.cpp
    graphics/filters.c
    graphics/gfxengine.cpp
    graphics/region.c
    graphics/sofont.cpp
    graphics/sprite.c
    graphics/toolkit.cpp
    graphics/vidmodes.c
    graphics/window.cpp
    sound/a_agw.c
    sound/a_bus.c
    sound/a_channel.c
    sound/a_commands.c
    sound/a_control.c
    sound/a_delay.c
    sound/a_events.c
    sound/a_filters.c
    sound/a_globals.c
    sound/a_limiter.c
    sound/a_midi.c
    sound/a_midicon.c
    sound/a_midifile.c
    sound/a_midisock.c
    sound/a_patch.c
    sound/a_pitch.c
    sound/a_plugin.c
    sound/a_sequencer.c
    sound/a_struct.c
    sound/a_voice.c
    sound/a_wave.c
    sound/a_wca.c
    sound/audio.c
    sound/audiodev.c
    sound/sfifo.c
)

# ---------------------------------------------------------------------------
# Dependencies
# ---------------------------------------------------------------------------
include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckCSourceCompiles)

# SDL2 — prefer config-file package (installed by Homebrew / vcpkg / etc.);
# fall back to pkg-config so Linux distro packages work too.
find_package(SDL2 CONFIG QUIET)
if(NOT SDL2_FOUND)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(SDL2 REQUIRED sdl2)
    # Wrap into an imported target so the rest of the file stays uniform.
    add_library(SDL2::SDL2 INTERFACE IMPORTED)
    target_include_directories(SDL2::SDL2 INTERFACE ${SDL2_INCLUDE_DIRS})
    target_link_libraries(SDL2::SDL2 INTERFACE ${SDL2_LIBRARIES})
    target_link_directories(SDL2::SDL2 INTERFACE ${SDL2_LIBRARY_DIRS})
    target_compile_options(SDL2::SDL2 INTERFACE ${SDL2_CFLAGS_OTHER})
endif()

find_package(SDL2_image CONFIG QUIET)
if(NOT SDL2_image_FOUND)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(SDL2_IMAGE REQUIRED SDL2_image>=2.0.0)
    add_library(SDL2_image::SDL2_image INTERFACE IMPORTED)
    target_include_directories(SDL2_image::SDL2_image INTERFACE ${SDL2_IMAGE_INCLUDE_DIRS})
    target_link_libraries(SDL2_image::SDL2_image INTERFACE ${SDL2_IMAGE_LIBRARIES})
    target_link_directories(SDL2_image::SDL2_image INTERFACE ${SDL2_IMAGE_LIBRARY_DIRS})
    target_compile_options(SDL2_image::SDL2_image INTERFACE ${SDL2_IMAGE_CFLAGS_OTHER})
endif()

# ---------------------------------------------------------------------------
# Windows resources
# ---------------------------------------------------------------------------
set(RES_FILES "")
if(MINGW)
    set(RES_FILES "${PROJECT_SOURCE_DIR}/icons/windows/kobodl.rc")
    set(CMAKE_RC_COMPILER_INIT windres)
    enable_language(RC)
    set(CMAKE_RC_COMPILE_OBJECT
        "<CMAKE_RC_COMPILER> <FLAGS> -O coff <DEFINES> -i <SOURCE> -o <OBJECT>")
endif()

# ---------------------------------------------------------------------------
# Executable
# ---------------------------------------------------------------------------
if(APPLE)
    set(MACOS_ICON "${PROJECT_SOURCE_DIR}/icons/kobodl.icns")
    set_source_files_properties("${MACOS_ICON}" PROPERTIES
        MACOSX_PACKAGE_LOCATION "Resources")
endif()

add_executable(kobodl WIN32 MACOSX_BUNDLE ${sources} ${RES_FILES}
    $<$<BOOL:${APPLE}>:${MACOS_ICON}>)

set_target_properties(kobodl PROPERTIES
    OUTPUT_NAME "kobodl"
    # macOS bundle metadata
    MACOSX_BUNDLE_BUNDLE_NAME        "KoboDeluxe"
    MACOSX_BUNDLE_BUNDLE_VERSION     "${VERSION_STRING}"
    MACOSX_BUNDLE_SHORT_VERSION_STRING "${VERSION_STRING}"
    MACOSX_BUNDLE_GUI_IDENTIFIER     "net.olofson.kobodeluxe"
    MACOSX_BUNDLE_COPYRIGHT          "Copyright © 1995-2018 David Olofson"
    MACOSX_BUNDLE_ICON_FILE          "kobodl.icns"
    # C++ standard
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(kobodl PRIVATE
    "${PROJECT_SOURCE_DIR}"
    "${PROJECT_SOURCE_DIR}/src"
    "${PROJECT_SOURCE_DIR}/src/graphics"
    "${PROJECT_SOURCE_DIR}/src/eel"
    "${PROJECT_SOURCE_DIR}/src/sound"
    "${PROJECT_SOURCE_DIR}/data/sfx"
    "${CMAKE_BINARY_DIR}/include"
)

target_link_libraries(kobodl PRIVATE
    SDL2::SDL2
    SDL2_image::SDL2_image
    ${KOBO_EXTRA_LIBRARIES}
)

if(WIN32)
    target_link_libraries(kobodl PRIVATE winmm dxguid)
endif()

# ---------------------------------------------------------------------------
# Feature detection
# ---------------------------------------------------------------------------
set(CMAKE_EXTRA_INCLUDE_FILES stdio.h)
check_function_exists(snprintf   KOBO_HAVE_SNPRINTF)
check_function_exists(_snprintf  KOBO_HAVE__SNPRINTF)
check_function_exists(vsnprintf  KOBO_HAVE_VSNPRINTF)
check_function_exists(_vsnprintf KOBO_HAVE__VSNPRINTF)

set(CMAKE_EXTRA_INCLUDE_FILES unistd.h)
check_function_exists(getegid KOBO_HAVE_GETEGID)
check_function_exists(setgid  KOBO_HAVE_SETGID)

set(CMAKE_EXTRA_INCLUDE_FILES sys/types.h sys/stat.h unistd.h)
check_function_exists(stat  KOBO_HAVE_STAT)
check_function_exists(lstat KOBO_HAVE_LSTAT)

set(CMAKE_EXTRA_INCLUDE_FILES)

# ---------------------------------------------------------------------------
# Runtime paths
# ---------------------------------------------------------------------------
if(NOT WIN32)
    set(KOBO_DATADIR    "${CMAKE_INSTALL_PREFIX}/share/kobodeluxe")
    set(KOBO_USERDIR    "HOME>>.kobodeluxe")
    set(KOBO_SYSCONFDIR "/etc")
endif()
set(KOBO_CONFIGFILE "kobodeluxe.cfg")

# ---------------------------------------------------------------------------
# Signal handler return type probe
# ---------------------------------------------------------------------------
check_c_source_compiles(
    "#include <sys/types.h>
     #include <signal.h>
     int main(void) { return *(signal(0,0))(0) == 1; }"
    RETSIGTYPE_INT
)
if(RETSIGTYPE_INT)
    set(RETSIGTYPE int)
else()
    set(RETSIGTYPE void)
endif()
message(STATUS "Signal handler return type: ${RETSIGTYPE}")

# ---------------------------------------------------------------------------
# Generated header
# ---------------------------------------------------------------------------
set(KOBODELUXE_AUTO_MESSAGE "This file is automatically generated. Do not edit!")
configure_file(
    "${CMAKE_SOURCE_DIR}/buildconfig.h.cmake"
    "${CMAKE_BINARY_DIR}/include/buildconfig.h"
)

# ---------------------------------------------------------------------------
# Install
# ---------------------------------------------------------------------------
install(TARGETS kobodl
    BUNDLE  DESTINATION "${KOBO_BIN_DIR}"
    RUNTIME DESTINATION "${KOBO_BIN_DIR}"
)
