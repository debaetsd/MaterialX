include_directories(
    ${EXTERNAL_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/../)

file(GLOB_RECURSE materialx_source "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")
file(GLOB_RECURSE materialx_headers "${CMAKE_CURRENT_SOURCE_DIR}/*.h*")

if(NOT MATERIALX_BUILD_OIIO)
    list(REMOVE_ITEM materialx_source "${CMAKE_CURRENT_SOURCE_DIR}/OiioImageLoader.cpp")
endif()

assign_source_group("Source Files" ${materialx_source})
assign_source_group("Header Files" ${materialx_headers})

if(UNIX)
    add_compile_options(-Wno-unused-function)
endif()

add_library(MaterialXRender ${materialx_source} ${materialx_headers})

target_link_libraries(
    MaterialXRender
    MaterialXGenShader
    ${CMAKE_DL_LIBS})

if(MATERIALX_BUILD_OIIO)
    set(OPENIMAGEIO_ROOT_DIR ${MATERIALX_OIIO_DIR})
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/External/OpenImageIO")
    find_package(OpenImageIO REQUIRED)
    if(OPENIMAGEIO_FOUND)
        include_directories(${OPENIMAGEIO_INCLUDE_DIR})
        target_link_libraries(MaterialXRender ${OPENIMAGEIO_LIBRARIES})
    endif()
endif()

set_target_properties(
    MaterialXRender PROPERTIES
    OUTPUT_NAME MaterialXRender
    COMPILE_FLAGS "${EXTERNAL_COMPILE_FLAGS}"
    LINK_FLAGS "${EXTERNAL_LINK_FLAGS}"
    VERSION "${MATERIALX_LIBRARY_VERSION}"
    SOVERSION "${MATERIALX_MAJOR_VERSION}"
    DEBUG_POSTFIX "${MATERIALX_DEBUG_POSTFIX}")

install(TARGETS MaterialXRender
    EXPORT MaterialX
    ARCHIVE DESTINATION ${MATERIALX_INSTALL_LIB_PATH}
    LIBRARY DESTINATION ${MATERIALX_INSTALL_LIB_PATH})

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/"
    DESTINATION ${CMAKE_INSTALL_PREFIX}/${MATERIALX_INSTALL_INCLUDE_PATH}/MaterialXRender/ MESSAGE_NEVER
    FILES_MATCHING PATTERN "*.h*")

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/MaterialXRender.pdb"
    DESTINATION "${CMAKE_INSTALL_PREFIX}/${MATERIALX_INSTALL_LIB_PATH}/" OPTIONAL)
