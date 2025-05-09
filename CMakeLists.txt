
#
# CMake options
# 

# CMake version
cmake_minimum_required(VERSION 3.20 FATAL_ERROR)


#
# Detect type of CMake setup
#

# Use PROJECT_IS_TOP_LEVEL once we bump the required CMake version up to 3.21
get_directory_property(META_HAS_PARENT PARENT_DIRECTORY)
if(META_HAS_PARENT)
  set(META_BUILD_AS_ROOT Off)
else()
  set(META_BUILD_AS_ROOT On)
endif()


#
# Configure CMake environment
#

# Register general cmake commands
include(cmake/Custom.cmake)

# Set policies
set_policy(CMP0054 NEW) #  ENABLE CMP0054: Only interpret if() arguments as variables or keywords when unquoted.
set_policy(CMP0042 NEW) #  ENABLE CMP0042: MACOSX_RPATH is enabled by default.
set_policy(CMP0063 NEW) #  ENABLE CMP0063: Honor visibility properties for all target types.
set_policy(CMP0069 NEW) #  ENABLE CMP0069: INTERPROCEDURAL_OPTIMIZATION is enforced when enabled.
set_policy(CMP0077 NEW) #  ENABLE CMP0077: option() honors normal variables.
set_policy(CMP0120 OLD) # DISABLE CMP0120: The WriteCompilerDetectionHeader module is removed.

# Include cmake modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(GenerateExportHeader)
include(WriteCompilerDetectionHeader)

# Include custom cmake modules
include(cmake/Coverage.cmake)
include(cmake/GenerateTemplateExportHeader.cmake)
include(cmake/GetGitRevisionDescription.cmake)
include(cmake/HealthCheck.cmake)


# 
# Project configuration options
# 

# Project options
option(BUILD_SHARED_LIBS      "Build shared instead of static libraries."              ON)
option(OPTION_SELF_CONTAINED  "Create a self-contained install with all dependencies." OFF)
option(OPTION_BUILD_TESTS     "Build tests."                                           ON)
option(OPTION_BUILD_DOCS      "Build documentation."                                   OFF)
option(OPTION_BUILD_EXAMPLES  "Build examples."                                        OFF)
option(OPTION_ENABLE_COVERAGE "Add coverage information."                              OFF)

# Get git revision
if (OPTION_USE_GIT_INFORMATION AND META_BUILD_AS_ROOT)
  get_git_head_revision(GIT_REFSPEC GIT_SHA1)
  string(SUBSTRING "${GIT_SHA1}" 0 12 GIT_REV)
  if(NOT GIT_SHA1)
      set(GIT_REV "0")
  endif()
else()
  set(GIT_SHA1 "")
  set(GIT_REV "0")
endif()


# 
# Project description and (meta) information
# 

# Meta information about the project
set(META_PROJECT_NAME        "template")
set(META_PROJECT_DESCRIPTION "CMake Project Template")
set(META_AUTHOR_ORGANIZATION "CG Internals GmbH")
set(META_AUTHOR_DOMAIN       "https://github.com/cginternals/cmake-init/")
set(META_AUTHOR_MAINTAINER   "opensource@cginternals.com")
set(META_VERSION_MAJOR       "2")
set(META_VERSION_MINOR       "1")
set(META_VERSION_PATCH       "0")
set(META_VERSION_REVISION    "${GIT_REV}")
set(META_VERSION             "${META_VERSION_MAJOR}.${META_VERSION_MINOR}.${META_VERSION_PATCH}")
set(META_NAME_VERSION        "${META_PROJECT_NAME} v${META_VERSION} (${META_VERSION_REVISION})")
set(META_CMAKE_INIT_SHA      "${GIT_REV}")
set(META_CMAKE_INIT_BRANCH   "cmake-3.20")

string(MAKE_C_IDENTIFIER ${META_PROJECT_NAME} META_PROJECT_ID)
string(TOUPPER ${META_PROJECT_ID} META_PROJECT_ID)


# 
# Declare project
# 

# Generate folders for IDE targets (e.g., VisualStudio solutions)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(IDE_FOLDER "")

# Declare project
project(${META_PROJECT_NAME}
    VERSION ${META_VERSION}
    DESCRIPTION ${META_PROJECT_DESCRIPTION}
)

enable_language(C)
enable_language(CXX)

# Set output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

# Create version file
file(WRITE "${PROJECT_BINARY_DIR}/VERSION" "${META_NAME_VERSION}")


#
# Project Health Check Setup
#

# Add cmake-init template check cmake targets
add_check_template_target(${META_CMAKE_INIT_SHA} ${META_CMAKE_INIT_BRANCH})

# Configure health check tools
enable_cppcheck(ON)
enable_clang_tidy(ON)
enable_coverage(${OPTION_ENABLE_COVERAGE})


# 
# Compiler settings and options
# 

include(cmake/CompileOptions.cmake)


# 
# Deployment/installation setup
# 

# Get project name
set(project ${META_PROJECT_NAME})

# Check for system dir install
set(SYSTEM_DIR_INSTALL FALSE)
if("${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr" OR "${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr/local")
    set(SYSTEM_DIR_INSTALL TRUE)
endif()

# Installation paths
if(UNIX AND SYSTEM_DIR_INSTALL)
    # Install into the system (/usr/bin or /usr/local/bin)
    set(INSTALL_ROOT      "share/${project}")       # /usr/[local]/share/<project>
    set(INSTALL_CMAKE     "share/${project}/cmake") # /usr/[local]/share/<project>/cmake
    set(INSTALL_EXAMPLES  "share/${project}")       # /usr/[local]/share/<project>
    set(INSTALL_DATA      "share/${project}")       # /usr/[local]/share/<project>
    set(INSTALL_BIN       "bin")                    # /usr/[local]/bin
    set(INSTALL_SHARED    "lib")                    # /usr/[local]/lib
    set(INSTALL_LIB       "lib")                    # /usr/[local]/lib
    set(INSTALL_INCLUDE   "include")                # /usr/[local]/include
    set(INSTALL_DOC       "share/doc/${project}")   # /usr/[local]/share/doc/<project>
    set(INSTALL_SHORTCUTS "share/applications")     # /usr/[local]/share/applications
    set(INSTALL_ICONS     "share/pixmaps")          # /usr/[local]/share/pixmaps
    set(INSTALL_INIT      "/etc/init")              # /etc/init (upstart init scripts)
else()
    # Install into local directory
    set(INSTALL_ROOT      ".")                      # ./
    set(INSTALL_CMAKE     "cmake")                  # ./cmake
    set(INSTALL_EXAMPLES  ".")                      # ./
    set(INSTALL_DATA      ".")                      # ./
    set(INSTALL_BIN       ".")                      # ./
    set(INSTALL_SHARED    "lib")                    # ./lib
    set(INSTALL_LIB       "lib")                    # ./lib
    set(INSTALL_INCLUDE   "include")                # ./include
    set(INSTALL_DOC       "doc")                    # ./doc
    set(INSTALL_SHORTCUTS "misc")                   # ./misc
    set(INSTALL_ICONS     "misc")                   # ./misc
    set(INSTALL_INIT      "misc")                   # ./misc
endif()

# Set runtime path
set(CMAKE_SKIP_BUILD_RPATH            FALSE) # Add absolute path to all dependencies for BUILD
set(CMAKE_BUILD_WITH_INSTALL_RPATH    FALSE) # Use CMAKE_INSTALL_RPATH for INSTALL
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH FALSE) # Do NOT add path to dependencies for INSTALL

if(NOT SYSTEM_DIR_INSTALL)
    # Find libraries relative to binary
    if(APPLE)
        set(EXECUTABLE_INSTALL_RPATH "@loader_path/../../../${INSTALL_LIB}")
        set(LIBRARY_INSTALL_RPATH    "@loader_path/../../../${INSTALL_LIB}")
    else()
        set(EXECUTABLE_INSTALL_RPATH "$ORIGIN/${INSTALL_LIB}")
        set(LIBRARY_INSTALL_RPATH    "$ORIGIN")
    endif()
endif()


# 
# Project modules
# 

add_subdirectory(source)
add_subdirectory(docs)
add_subdirectory(deploy)


# 
# Deployment (global project files)
# 

# Install version file
install(FILES "${PROJECT_BINARY_DIR}/VERSION" DESTINATION ${INSTALL_ROOT} COMPONENT runtime)

# Install cmake find script for the project
install(FILES ${META_PROJECT_NAME}-config.cmake DESTINATION ${INSTALL_ROOT} COMPONENT dev)

# Install the project meta files
install(FILES AUTHORS   DESTINATION ${INSTALL_ROOT} COMPONENT runtime)
install(FILES LICENSE   DESTINATION ${INSTALL_ROOT} COMPONENT runtime)
install(FILES README.md DESTINATION ${INSTALL_ROOT} COMPONENT runtime)

# Install runtime data
install(DIRECTORY ${PROJECT_SOURCE_DIR}/data DESTINATION ${INSTALL_DATA} COMPONENT runtime)
