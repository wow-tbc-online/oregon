# This file is part of the OregonCore Project. See AUTHORS file for Copyright information
#
# This file is free software; as a special exception the author gives
# unlimited permission to copy and/or distribute it, with or without
# modifications, as long as this notice is preserved.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY, to the extent permitted by law; without even the
# implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

# Enable precompiled headers when using the GCC compiler.
CU_RUN_HOOK(BEFORE_SCRIPTS_LIBRARY)

CU_SET_PATH("AC_SCRIPTS_DIR" "${CMAKE_CURRENT_LIST_DIR}")

if( USE_SCRIPTPCH )
  include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}
  )
endif()

file(GLOB_RECURSE scripts_custom Custom/*.cpp Custom/*.h)
file(GLOB_RECURSE scripts_easternplaguelands EasternKingdoms/*.cpp EasternKingdoms/*.h)
file(GLOB_RECURSE scripts_examples Examples/*.cpp Examples/*.h)
file(GLOB_RECURSE scripts_kalimdor Kalimdor/*.cpp Kalimdor/*.h)
file(GLOB_RECURSE scripts_outland Outland/*.cpp Outland/*.h)
file(GLOB_RECURSE scripts_world World/*.cpp World/*.h)
file(GLOB_RECURSE scripts_commands Commands/*.cpp Commands/*.h)

if( USE_SCRIPTPCH AND MSVC )
  set(scripts_STAT_SRCS
    PrecompiledHeaders/ScriptPCH.cpp
    PrecompiledHeaders/ScriptPCH.h
  )
endif()

set(scripts_STAT_SRCS
  ${scripts_STAT_SRCS}
  ${BUILDDIR}/GenLoader.cpp
  ../game/ScriptedEscortAI.cpp
  ../game/ScriptedGuardAI.cpp
  ../game/ScriptedCreature.cpp
  ../game/ScriptedFollowerAI.cpp
  ../game/ScriptedSimpleAI.cpp
)

CU_SET_PATH("CMAKE_AC_MODULE_DIR" "${CMAKE_CURRENT_LIST_DIR}")

CU_GET_GLOBAL("AC_ADD_SCRIPTS_LIST")
CU_GET_GLOBAL("AC_ADD_SCRIPTS_INCLUDE")
CU_GET_GLOBAL("AC_SCRIPTS_SOURCES")

if(SCRIPTS)
  set(scripts_STAT_SRCS
    ${AC_SCRIPTS_SOURCES}
    ${scripts_STAT_SRCS}
    ${scripts_custom}
    ${scripts_easternplaguelands}
    ${scripts_examples}
    ${scripts_kalimdor}
    ${scripts_outland}
    ${scripts_world}
    ${scripts_commands}
  )
endif()

set("AC_SCRIPTS_INCLUDES" "")

FOREACH (include ${AC_ADD_SCRIPTS_INCLUDE})
    set("AC_SCRIPTS_INCLUDES" "#include \"${include}\"\n${AC_SCRIPTS_INCLUDES}")
ENDFOREACH()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/GenLoader.cpp.cmake ${BUILDDIR}/GenLoader.cpp)

include_directories(
  ${CMAKE_BINARY_DIR}
  ${CMAKE_SOURCE_DIR}/dep/g3dlite/include
  ${CMAKE_SOURCE_DIR}/dep/SFMT
  ${CMAKE_SOURCE_DIR}/dep/mersennetwister
  ${CMAKE_SOURCE_DIR}/dep/zlib
  ${CMAKE_SOURCE_DIR}/src/framework
  ${CMAKE_SOURCE_DIR}/src/collision
  ${CMAKE_SOURCE_DIR}/src/game
  ${CMAKE_SOURCE_DIR}/dep/recastnavigation/Detour
  ${CMAKE_SOURCE_DIR}/dep/recastnavigation
  ${CMAKE_SOURCE_DIR}/src/shared
  ${CMAKE_SOURCE_DIR}/src/shared/Database
  ${CMAKE_CURRENT_SOURCE_DIR}/PrecompiledHeaders
  ${ACE_INCLUDE_DIR}
  ${MYSQL_INCLUDE_DIR}
  ${OPENSSL_INCLUDE_DIR}
)

source_group("Modules" FILES ${AC_SCRIPTS_SOURCES})

GroupSources(${CMAKE_CURRENT_SOURCE_DIR})

add_library(scripts STATIC ${scripts_STAT_SRCS})

add_dependencies(scripts revision.h)

# Generate precompiled header
if( USE_SCRIPTPCH )
  if(CMAKE_COMPILER_IS_GNUCXX)
    add_precompiled_header(scripts ${CMAKE_CURRENT_SOURCE_DIR}/PrecompiledHeaders/ScriptPCH.h)
  elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_clang_pch(scripts "${CMAKE_CURRENT_SOURCE_DIR}/PrecompiledHeaders" "ScriptPCH.h" "ScriptPCH.cpp")
  elseif(MSVC)
    add_native_precompiled_header(scripts ${CMAKE_CURRENT_SOURCE_DIR}/PrecompiledHeaders/ScriptPCH)
  endif()
endif()
