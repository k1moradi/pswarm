cmake_minimum_required(VERSION 3.15)
project(pswarm C Fortran)

# === Options ===
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)        # Default to static
option(BUILD_PYTHON     "Build the Python interface" OFF)
option(BUILD_MATLAB     "Build the MATLAB interface" OFF)
option(USE_AMPL_SOLVER  "Link against AMPL solver library for AMPL interface targets" OFF)

# === Dependencies ===
find_package(MPI REQUIRED)
find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)

# === AMPL Solver ===
if(USE_AMPL_SOLVER)
  set(AMPL_SOLVER "${CMAKE_SOURCE_DIR}/libs/amplsolver.a" CACHE FILEPATH "AMPL solver library")
  if(NOT EXISTS ${AMPL_SOLVER})
    message(FATAL_ERROR "USE_AMPL_SOLVER=ON but AMPL_SOLVER (${AMPL_SOLVER}) does not exist! Set the correct path with -DAMPL_SOLVER=...")
  endif()
endif()

# === Platform/compiler flags ===
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
  set(CMAKE_C_FLAGS_RELEASE   "-O3 -DNDEBUG -fPIC -Wall -Wextra" CACHE STRING "Release flags" FORCE)
  set(CMAKE_C_FLAGS_DEBUG     "-g -O0 -fPIC -Wall -Wextra -DDEBUG" CACHE STRING "Debug flags" FORCE)
  set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -DNDEBUG -fPIC" CACHE STRING "" FORCE)
  set(CMAKE_Fortran_FLAGS_DEBUG   "-g -O0 -fPIC -DDEBUG" CACHE STRING "" FORCE)
endif()

if(MSVC)
  set(CMAKE_C_FLAGS_RELEASE   "/O2 /DNDEBUG /W3" CACHE STRING "Release flags" FORCE)
  set(CMAKE_C_FLAGS_DEBUG     "/Zi /Od /W3 /DDEBUG" CACHE STRING "Debug flags" FORCE)
  set(CMAKE_Fortran_FLAGS_RELEASE "/O2 /DNDEBUG" CACHE STRING "" FORCE)
  set(CMAKE_Fortran_FLAGS_DEBUG   "/Zi /Od /DDEBUG" CACHE STRING "" FORCE)
endif()

if(NOT WIN32)
  set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

# === Sources ===
set(SRC        pattern.c pswarm.c pswarm_main.c cache.c user.c)
set(SRC_LINEAR mve_solver.c mve_presolve.c)
set(LIBSRC     pattern.c pswarm.c)
set(RSRC       pswarm_r.c)
set(PYSRC      pswarm_py.c)

if(WIN32)
  set(PLATFORM_LIBS ws2_32)
else()
  set(PLATFORM_LIBS dl m)
endif()

# === Core Libraries ===
add_library(pswarm        ${LIBSRC})
add_library(pswarm_linear ${LIBSRC} ${SRC_LINEAR})

set_target_properties(pswarm pswarm_linear PROPERTIES
  POSITION_INDEPENDENT_CODE ON
  WINDOWS_EXPORT_ALL_SYMBOLS ON
  OUTPUT_NAME "pswarm"
)

target_link_libraries(pswarm        PRIVATE ${BLAS_LIBRARIES} ${PLATFORM_LIBS})
target_link_libraries(pswarm_linear PRIVATE ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES} ${PLATFORM_LIBS})

# === Main Executables ===
add_executable(pswarm_serial          ${SRC})
add_executable(pswarm_serial_linear   ${SRC} ${SRC_LINEAR})
add_executable(pswarm_parallel        ${SRC})
add_executable(pswarm_parallel_linear ${SRC} ${SRC_LINEAR})

target_link_libraries(pswarm_serial          PRIVATE pswarm         ${PLATFORM_LIBS})
target_link_libraries(pswarm_serial_linear   PRIVATE pswarm_linear  ${PLATFORM_LIBS} ${LAPACK_LIBRARIES})

target_link_libraries(pswarm_parallel        PRIVATE MPI::MPI_C pswarm         ${PLATFORM_LIBS})
target_link_libraries(pswarm_parallel_linear PRIVATE MPI::MPI_C pswarm_linear ${PLATFORM_LIBS} ${LAPACK_LIBRARIES})

# === Optional: AMPL Executables ===
if(USE_AMPL_SOLVER)
  add_executable(pswarm_ampl        ${SRC})
  add_executable(pswarm_ampl_linear ${SRC} ${SRC_LINEAR})
  target_link_libraries(pswarm_ampl        PRIVATE pswarm        ${AMPL_SOLVER} ${PLATFORM_LIBS})
  target_link_libraries(pswarm_ampl_linear PRIVATE pswarm_linear ${AMPL_SOLVER} ${PLATFORM_LIBS} ${LAPACK_LIBRARIES})
endif()

# === Optional: MATLAB/MEX interface (auto-detects latest MATLAB) ===
if(BUILD_MATLAB)
  find_package(Matlab COMPONENTS MX_LIBRARY MEX_COMPILER REQUIRED)
  if(Matlab_FOUND)
    add_library(pswarm_matlab SHARED ${RSRC}) # Replace with your actual MEX code if needed
    target_include_directories(pswarm_matlab PRIVATE ${Matlab_INCLUDE_DIRS})
    target_link_libraries(pswarm_matlab PRIVATE pswarm ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES} ${PLATFORM_LIBS} ${Matlab_MEX_LIBRARY})
    set_target_properties(pswarm_matlab PROPERTIES
      PREFIX ""
      SUFFIX ".mex${Matlab_MEX_EXTENSION}"
    )
    message(STATUS "MATLAB found: ${Matlab_ROOT_DIR} (${Matlab_VERSION_STRING})")
  else()
    message(FATAL_ERROR "MATLAB requested but not found! Please specify MATLAB_ROOT, or install MATLAB.")
  endif()
endif()

# === Optional: Python 3.x interface (auto-detects latest Python/NumPy) ===
if(BUILD_PYTHON)
  find_package(Python3 COMPONENTS Interpreter Development NumPy REQUIRED)
  if(Python3_FOUND AND Python3_NumPy_FOUND)
    add_library(pswarm_py MODULE ${PYSRC})
    target_include_directories(pswarm_py PRIVATE
      ${Python3_INCLUDE_DIRS}
      ${Python3_NumPy_INCLUDE_DIRS}
    )
    target_link_libraries(pswarm_py PRIVATE pswarm ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES} ${PLATFORM_LIBS} ${Python3_LIBRARIES})
    # For Py3: .cpython-XYm.so, fallback to .so if SOABI not defined
    if(Python3_SOABI)
      set_target_properties(pswarm_py PROPERTIES PREFIX "" SUFFIX ".${Python3_SOABI}.so")
    else()
      set_target_properties(pswarm_py PROPERTIES PREFIX "" SUFFIX ".so")
    endif()
    message(STATUS "Python found: ${Python3_EXECUTABLE} (${Python3_VERSION})")
    message(STATUS "Python NumPy include: ${Python3_NumPy_INCLUDE_DIRS}")
  else()
    message(FATAL_ERROR "Python 3 and NumPy required for Python interface but not found! Please check your Python installation.")
  endif()
endif()

# === Output Directories ===
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# === Clean Target ===
add_custom_target(distclean
  COMMAND ${CMAKE_COMMAND} -E rm -rf ${CMAKE_BINARY_DIR}/bin ${CMAKE_BINARY_DIR}/lib
)

# === Info ===
message(STATUS "BLAS: ${BLAS_LIBRARIES}")
message(STATUS "LAPACK: ${LAPACK_LIBRARIES}")
message(STATUS "MPI: ${MPI_C_LIBRARIES}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
if(USE_AMPL_SOLVER)
  message(STATUS "AMPL solver: ${AMPL_SOLVER}")
endif()
