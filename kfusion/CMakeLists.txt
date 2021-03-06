cmake_minimum_required(VERSION 2.8)

set(appname kfusion)

find_package(TooN   REQUIRED)
find_package(OpenCL )
find_package(SYCL   )
find_package(CUDA   )
find_package(GLUT   )
find_package(OpenGL )
find_package(OpenNI2)
find_package(PAPI)

find_package(PkgConfig)
find_package(Qt5Core)
find_package(Qt5Widgets)
find_package(Qt5Gui)
find_package(Qt5OpenGL)
find_package(Qt5PrintSupport)

include_directories(./include ${TOON_INCLUDE_PATHS} ./thirdparty)


if (CUDA_FOUND) 
 SET( CUDA_PROPAGATE_HOST_FLAGS OFF )
endif(CUDA_FOUND)

 # ---- PREPARE COMMON DEPENDENCIES  ------------ 
if(APPLE)
    set(common_libraries     stdc++ pthread)
else(APPLE)
    set(common_libraries     rt stdc++ pthread)
endif(APPLE)


set(common_compile_flags "")


 # ----------------- SYCL VERSION ----------------- 

if    (SYCL_FOUND)    # For this, CXX must also be set to syclcc
  add_library(${appname}-sycl  src/sycl/kernels.cpp)
  target_link_libraries(${appname}-sycl   ${common_libraries})	
  add_definitions( -DSYCL )
  add_version(${appname} sycl "" "") # Paths 
endif (SYCL_FOUND)

 # ----------------- CPP VERSION ----------------- 

add_library(${appname}-cpp  src/cpp/kernels.cpp)
target_link_libraries(${appname}-cpp   ${common_libraries})	
add_version(${appname} cpp "" "")

 # ----------------- OPENMP VERSION ----------------- 

add_library(${appname}-openmp  src/cpp/kernels.cpp)
target_link_libraries(${appname}-openmp   ${common_libraries})	
SET_TARGET_PROPERTIES(${appname}-openmp PROPERTIES COMPILE_FLAGS "-fopenmp")
add_version(${appname} openmp "-fopenmp" "-fopenmp")


 #  ----------------- OCL VERSION ----------------- 
 
if (OPENCL_FOUND) 
   include_directories(${OPENCL_INCLUDE_DIRS})
    add_library(${appname}-opencl  src/opencl/kernels.cpp src/opencl/common_opencl.cpp)
    target_link_libraries(${appname}-opencl   ${common_libraries} ${OPENCL_LIBRARIES})	
    add_version(${appname} opencl "" "")
endif(OPENCL_FOUND)


 #  ----------------- CUDA VERSION ----------------- 
if (CUDA_FOUND) 


set(kfusion_cuda_srcs
	src/cuda/kernels.cu
	thirdparty/kfusion.h
	)
	
    cuda_add_library(${appname}-cuda    STATIC  ${kfusion_cuda_srcs} OPTIONS -O3 )
        target_link_libraries(${appname}-cuda   ${common_libraries})	
    add_version(${appname} cuda    "" "")

endif(CUDA_FOUND)

#  ----------------- thirdparty ----------------- 

add_subdirectory( thirdparty    )


