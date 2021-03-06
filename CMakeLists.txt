#--------------------------------------------------------------------
# Inviwo OpenMesh Module
ivw_module(OpenMesh)

#--------------------------------------------------------------------
# Add OpenMesh add set correct folders
option(OPENMESH_BUILD_PYTHON_BINDINGS "" OFF)

set( BUILD_APPS OFF CACHE BOOL "Enable or disable building of apps" )

add_subdirectory(ext/OpenMesh)
if(TARGET doc)
    ivw_folder(doc ext/OpenMesh)
endif()
if(TARGET doc-install)
    ivw_folder(doc-install ext/OpenMesh)
endif()
if(TARGET winhelp)
    ivw_folder(winhelp ext/OpenMesh)
endif()

ivw_folder(OpenMeshCore ext/OpenMesh)
ivw_folder(OpenMeshTools ext/OpenMesh)



#--------------------------------------------------------------------
# Add header files
set(HEADER_FILES
    include/modules/openmesh/openmeshmodule.h
    include/modules/openmesh/openmeshmoduledefine.h
    include/modules/openmesh/openmeshreader.h
    include/modules/openmesh/openmeshwriter.h
    include/modules/openmesh/processors/meshdecimationprocessor.h
    include/modules/openmesh/processors/trianglestowireframe.h
    include/modules/openmesh/utils/meshdecimation.h
    include/modules/openmesh/utils/meshiteratorutils.h
    include/modules/openmesh/utils/meshproximity.h
    include/modules/openmesh/utils/openmeshconverters.h
)
ivw_group("Header Files" ${HEADER_FILES})

#--------------------------------------------------------------------
# Add source files
set(SOURCE_FILES
    src/openmeshmodule.cpp
    src/openmeshreader.cpp
    src/openmeshwriter.cpp
    src/processors/meshdecimationprocessor.cpp
    src/processors/trianglestowireframe.cpp
    src/utils/meshdecimation.cpp
    src/utils/meshiteratorutils.cpp
    src/utils/meshproximity.cpp
    src/utils/openmeshconverters.cpp
)
ivw_group("Source Files" ${SOURCE_FILES})


#--------------------------------------------------------------------
# Add shaders
set(SHADER_FILES
    #${CMAKE_CURRENT_SOURCE_DIR}/glsl/meshprocessor.frag
)
ivw_group("Shader Files" ${SHADER_FILES})


#--------------------------------------------------------------------
# Add Unittests
set(TEST_FILES
	#${CMAKE_CURRENT_SOURCE_DIR}/tests/unittests/mesh-test.cpp
)
ivw_add_unittest(${TEST_FILES})

#--------------------------------------------------------------------
# Create module
ivw_create_module(${SOURCE_FILES} ${HEADER_FILES} ${SHADER_FILES})

target_link_libraries(${${mod}_target} PUBLIC 
  OpenMeshCore
  OpenMeshTools
)
target_include_directories(${${mod}_target} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/ext/OpenMesh/src)

ivw_register_license_file(NAME OpenMesh VERSION 7.0 MODULE Mesh TYPE "3-clause BSD License"
    URL https://www.openmesh.org/
    FILES ${CMAKE_CURRENT_SOURCE_DIR}/ext/OpenMesh/LICENSE
)


#--------------------------------------------------------------------
# Add shader directory to pack
# ivw_add_to_module_pack(${CMAKE_CURRENT_SOURCE_DIR}/glsl)


