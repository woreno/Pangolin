# Find Pangolin (https://github.com/stevenlovegrove/Pangolin)
find_package(Pangolin REQUIRED)
include_directories(${Pangolin_INCLUDE_DIRS})
link_directories(${Pangolin_LIBRARY_DIRS})
link_libraries(${Pangolin_LIBRARIES})

add_executable(SimpleMultiDisplay main.cpp)

# In your own code, you can ommit these lines
if( ${PANGOLIN_EXAMPLE} )
  add_dependencies(SimpleMultiDisplay ${LIBRARY_NAME})
endif()
