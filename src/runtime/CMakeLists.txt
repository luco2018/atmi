#
#MIT License 
#
#Copyright © 2016 Advanced Micro Devices, Inc.  
#
#Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software
#without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
#persons to whom the Software is furnished to do so, subject to the following conditions:
#
#The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
#THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
#PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
#OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#

if(ATMI_DEP_LIBHSA_FOUND)
    if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86_64)|(ppc64le)$" AND CMAKE_SYSTEM_NAME MATCHES "Linux")
        # Enable support for C++11.
        add_definitions(-std=c++11)
        
        # If building this library in debug mode, we define a macro to enable
        # dumping progress messages at runtime.
        string( TOLOWER "${CMAKE_BUILD_TYPE}" ATMI_CMAKE_BUILD_TYPE)
        if(ATMI_CMAKE_BUILD_TYPE MATCHES debug)
          add_definitions(-DDEBUG)
          add_definitions(-g)
          add_definitions(-O0)
        else()
          add_definitions(-g)
          add_definitions(-O2)
        endif()
        add_definitions(-c)
        add_definitions(-fpic)

        add_library(atmi_runtime SHARED
            atl_bindthread.c 
            atl_cputask.c    
            atl_profile.c    
            atl_rt.c         
            atl_synckernel.c 
            ATLData.cpp      
            ATLMachine.cpp   
            ATLQueue.cpp
            )

        include_directories(${ATMI_DEP_LIBHSA_INCLUDE_DIRS})
        include_directories(${CMAKE_CURRENT_SOURCE_DIR})
        include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
        include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../../include)

        # Install plugin under the lib destination folder.
        if(CMAKE_BUILD_TYPE MATCHES Debug)
          install(TARGETS atmi_runtime LIBRARY DESTINATION "lib-debug" COMPONENT runtime )
        else()
          install(TARGETS atmi_runtime LIBRARY DESTINATION "lib" COMPONENT runtime )
        endif()
        
        ## CPack component info
        set(CPACK_COMPONENT_RUNTIME_DISPLAY_NAME "ATMI Runtime Library")
        set(CPACK_COMPONENT_RUNTIME_REQUIRED TRUE )

        INSTALL(FILES 
                ${CMAKE_CURRENT_SOURCE_DIR}/../../include/atmi.h
                ${CMAKE_CURRENT_SOURCE_DIR}/../../include/atmi_runtime.h
                DESTINATION "include"
                COMPONENT runtime
               )

        target_link_libraries(
          atmi_runtime
          "-Wl,--as-needed"
          -L${ATMI_DEP_LIBHSA_LIBRARIES_DIRS} -lhsa-runtime64 
          )
    else()
        libatmi_runtime_say("Not building ATMI Runtime: only support ATMI in Linux x86_64 or ppc64le hosts.")
    endif()
else()
    libatmi_runtime_say("Not building ATMI Runtime: libhsa-runtime64 not found")
endif()
