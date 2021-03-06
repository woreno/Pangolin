# Find Pangolin (https://github.com/stevenlovegrove/Pangolin)
find_package(Pangolin REQUIRED)
include_directories(${Pangolin_INCLUDE_DIRS})
link_directories(${Pangolin_LIBRARY_DIRS})
link_libraries(${Pangolin_LIBRARIES})

find_package(CUDA QUIET)

if(CUDA_FOUND)
  cuda_include_directories(${CMAKE_CURRENT_SOURCE_DIR})

  set(CUDA_NVCC_FLAGS "-O3")
  add_definitions(-DCUDA_BUILD)

  if(CUDA_SDK_ROOT_DIR AND EXISTS("${CUDA_SDK_ROOT_DIR}/lib/libcutil_x86_64.a") )
    add_definitions(-DUSE_CUTIL)
    include_directories( ${CUDA_SDK_ROOT_DIR}/common/inc )
    link_directories( ${CUDA_SDK_ROOT_DIR}/lib )
    link_libraries(libcutil_x86_64.a)
  endif()

  cuda_add_executable(
    VBODisplay
    main.cpp kernal.cu
  )

  # In your own code, you can ommit these lines
  if( ${PANGOLIN_EXAMPLE} )
    add_dependencies(VBODisplay ${LIBRARY_NAME})
  endif()
endif()
