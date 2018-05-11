#--------------------------------------------------------------------
# Inviwo OpenMesh Module
ivw_module(OpenMesh)

#--------------------------------------------------------------------
# Add OpenMesh add set correct folders
option(OPENMESH_BUILD_PYTHON_BINDINGS "" OFF)
add_subdirectory(ext/OpenMesh)
ivw_folder(doc ext/OpenMesh)
ivw_folder(doc-install ext/OpenMesh)
if(TARGET winhelp)
    ivw_folder(winhelp ext/OpenMesh)
endif()
ivw_folder(OpenMeshCore ext/OpenMesh)
ivw_folder(OpenMeshTools ext/OpenMesh)



#--------------------------------------------------------------------
# Add header files
set(HEADER_FILES
    #${CMAKE_CURRENT_SOURCE_DIR}/meshprocessor.h
    ${CMAKE_CURRENT_SOURCE_DIR}/openmeshreader.h
    ${CMAKE_CURRENT_SOURCE_DIR}/openmeshwriter.h
    ${CMAKE_CURRENT_SOURCE_DIR}/processors/meshdecimationprocessor.h
    ${CMAKE_CURRENT_SOURCE_DIR}/processors/trianglestowireframe.h
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/meshdecimation.h
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/meshiteratorutils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/meshproximity.h
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/openmeshconverters.h
)
ivw_group("Header Files" ${HEADER_FILES})

#--------------------------------------------------------------------
# Add source files
set(SOURCE_FILES
    #${CMAKE_CURRENT_SOURCE_DIR}/meshprocessor.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/openmeshreader.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/openmeshwriter.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/processors/meshdecimationprocessor.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/processors/trianglestowireframe.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/meshdecimation.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/meshiteratorutils.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/meshproximity.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/utils/openmeshconverters.cpp
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