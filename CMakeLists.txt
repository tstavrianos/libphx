cmake_minimum_required (VERSION 3.16)

if(DEFINED ENV{VCPKG_ROOT} AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
      CACHE STRING "")
endif()

if(DEFINED ENV{VCPKG_DEFAULT_TRIPLET} AND NOT DEFINED VCPKG_TARGET_TRIPLET)
  set(VCPKG_TARGET_TRIPLET "$ENV{VCPKG_DEFAULT_TRIPLET}" CACHE STRING "")
endif()

project (phx)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

find_package(freetype CONFIG REQUIRED)
find_package(SDL2 CONFIG REQUIRED)
find_package(Bullet CONFIG REQUIRED)
find_package(GLEW REQUIRED)
find_package(lz4 CONFIG REQUIRED)
find_package(LuaJIT REQUIRED)

include (script/build/Shared.cmake)

# ------------------------------------------------------------------------------

file (GLOB HEADERS "include/*.h")
file (GLOB SOURCES "src/*.cpp")

add_library (phx SHARED ${SOURCES} ${HEADERS})
phx_configure_output_dir (phx)
phx_configure_target_properties (phx)

target_compile_definitions (phx PRIVATE LIBPHX_BUILDING=1)
target_include_directories (phx PUBLIC "include")
target_include_directories (phx PUBLIC "ext/include")
target_link_directories (phx PUBLIC "ext/lib/${PLATARCH}")

set_target_properties (phx PROPERTIES
  PREFIX "lib"
  OUTPUT_NAME_DEBUG "phx${ARCH}d"
  OUTPUT_NAME_RELEASE "phx${ARCH}r"
  OUTPUT_NAME_RELWITHDEBINFO "phx${ARCH}"
  OUTPUT_NAME_MINSIZEREL "phx${ARCH}rm")

# ------------------------------------------------------------------------------

if (WINDOWS)

  # register an external shared library
  function (add_extlib name lib dll)
    add_library (${name} SHARED IMPORTED GLOBAL)
    set_property (TARGET ${name} PROPERTY IMPORTED_IMPLIB "${CMAKE_CURRENT_SOURCE_DIR}/ext/lib/${PLATARCH}/${lib}")
    set_property (TARGET ${name} PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/ext/lib/${PLATARCH}/${dll}")
  endfunction ()

  # link target to an external library; also add a post-build step to
  # copy the library DLL into the bin folder
  function (target_link_extlib target lib)
    target_link_libraries (${target} PRIVATE ${lib})
    add_custom_command (TARGET ${target} POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy_if_different
      $<TARGET_FILE:${lib}> $<TARGET_FILE_DIR:${target}>
    )
  endfunction ()

  if (ARCH EQUAL "32")
    add_extlib (fmod fmodL_vc.lib fmodL.dll)
    add_extlib (fmodstudio fmodstudioL_vc.lib fmodstudioL.dll)
  else ()
    add_extlib (fmod fmodL64_vc.lib fmodL64.dll)
    add_extlib (fmodstudio fmodstudioL64_vc.lib fmodstudioL64.dll)
  endif ()

elseif (LINUX)

  # TODO

endif ()

# ------------------------------------------------------------------------------

if (WINDOWS)

  target_link_libraries (phx PRIVATE
    opengl32.lib
    user32.lib winmm.lib Ws2_32.lib
	)

  target_link_extlib (phx fmod)
  target_link_extlib (phx fmodstudio)

elseif (LINUX)

  target_link_libraries (phx PRIVATE
    GL
    dl
    fmod fmodstudio
	)

  # Add ext/lib/{$PLATARCH} to runtime library search path
  target_link_options (phx PRIVATE "-Wl,-rpath,../ext/lib/${PLATARCH}")

endif ()

target_link_directories(phx PRIVATE ${BULLET_LIBRARY_DIRS})

target_link_libraries(phx PRIVATE
	freetype
	SDL2::SDL2 SDL2::SDL2main
    GLEW::GLEW
	lz4::lz4
	${LUAJIT_LIBRARIES}
    BulletSoftBody BulletDynamics BulletCollision Bullet3Common LinearMath
	)

target_include_directories (phx PUBLIC
        ${LUAJIT_INCLUDE_DIRS})

