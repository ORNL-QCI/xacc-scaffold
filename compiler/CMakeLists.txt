if (NOT LLVM_ROOT)
   # By default, users should install scaffold package from
   # us, so it will be in this location
   set (LLVM_ROOT /usr/local/scaffold/build/Release+Asserts)
endif()

find_package(ScaffoldClang)

if (CLANG_FOUND AND LLVM_FOUND)

   set (LIBRARY_NAME xacc-scaffold)

   file (GLOB HEADERS *.hpp)
   file (GLOB SRC *.cpp)

   set(CMAKE_CXX_FLAGS "-D_DEBUG -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS -O3 -w -fomit-frame-pointer -fvisibility-inlines-hidden -fPIC -Woverloaded-virtual -Wcast-qual -rdynamic")

   include_directories(${CLANG_INCLUDE_DIRS})
   include_directories(${CLANG_INCLUDE_DIRS}/extra-tools)

   # Set up dependencies to resources to track changes
   usFunctionGetResourceSource(TARGET ${LIBRARY_NAME} OUT SRC)
   # Generate bundle initialization code
   usFunctionGenerateBundleInit(TARGET ${LIBRARY_NAME} OUT SRC)

   add_library(${LIBRARY_NAME} SHARED ${SRC})

   set(_bundle_name xacc_scaffold)

 set_target_properties(${LIBRARY_NAME} PROPERTIES
     # This is required for every bundle
     COMPILE_DEFINITIONS US_BUNDLE_NAME=${_bundle_name}
     # This is for convenience, used by other CMake functions
     US_BUNDLE_NAME ${_bundle_name}
     )

   if(BUILD_SHARED_LIBS)
     set_target_properties(${LIBRARY_NAME} PROPERTIES
       CXX_VISIBILITY_PRESET hidden
       VISIBILITY_INLINES_HIDDEN 1
      )
   endif()

   # Embed meta-data from a manifest.json file
   usFunctionEmbedResources(TARGET ${LIBRARY_NAME}
     WORKING_DIRECTORY
       ${CMAKE_CURRENT_SOURCE_DIR}
     FILES
       manifest.json
     )

   link_directories(${CLANG_LIBRARY_DIRS})
   link_directories(${LLVM_LIBRARY_DIRS})

   target_link_libraries(${LIBRARY_NAME} ${CLANG_LIBS} ${LLVM_LIBS} ${Boost_LIBRARIES} ${XACC_LIBRARIES})

   install(TARGETS ${LIBRARY_NAME} DESTINATION ${XACC_ROOT}/lib/plugins/compilers/)
 # Gather tests
   file (GLOB test_files tests/*.cpp)
   add_tests("${test_files}" "${CMAKE_CURRENT_SOURCE_DIR}" "${LIBRARY_NAME};${CLANG_LIBS};${LLVM_LIBS};dl;pthread")
endif()
