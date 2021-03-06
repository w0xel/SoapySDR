########################################################################
## Feature registration
########################################################################
include(FeatureSummary)
include(CMakeDependentOption)
cmake_dependent_option(ENABLE_LIBRARY "Enable runtime library" ON "TRUE" OFF)
add_feature_info(Library ENABLE_LIBRARY "runtime library v${SOAPY_SDR_VERSION}")
if (NOT ENABLE_LIBRARY)
    return()
endif()

########################################################################
# Soapy SDR sources and dependencies
########################################################################
list(APPEND SOAPY_SDR_SOURCES
    Device.cpp
    Factory.cpp
    Registry.cpp
    Types.cpp
    NullDevice.cpp
    Logger.cpp
    Errors.cpp
    Formats.cpp
    ConverterRegistry.cpp
    DefaultConverters.cpp
)

#dl libs used by dlopen in unix
list(APPEND SOAPY_SDR_LIBRARIES
    ${CMAKE_DL_LIBS}
)

if (WIN32)
    set(MODULE_EXT "dll")
elseif (UNIX)
    set(MODULE_EXT "so")
else()
    message(FATAL_ERROR "not win32 or unix")
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
    #support threaded client code
    #notice that -pthread is not the same as -lpthread
    list(APPEND SOAPY_SDR_LIBRARIES -pthread)

    #symbols are only exported from libraries/modules explicitly
    add_compile_options(-fvisibility=hidden)
    add_compile_options(-fvisibility-inlines-hidden)
endif()

include_directories(${PROJECT_SOURCE_DIR}/include)

list(APPEND SOAPY_SDR_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/Modules.cpp)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/Modules.in.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/Modules.cpp
@ONLY)

list(APPEND SOAPY_SDR_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/Version.cpp)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/Version.in.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/Version.cpp
@ONLY)

#C API support sources
list(APPEND SOAPY_SDR_SOURCES
    TypesC.cpp
    ModulesC.cpp
    VersionC.cpp
    DeviceC.cpp
    FactoryC.cpp
    LoggerC.cpp
    TimeC.cpp
    ErrorsC.cpp
    FormatsC.cpp
)

#disable MSVC warnings
if (MSVC)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
endif ()

########################################################################
# Build the library
########################################################################
add_library(SoapySDR SHARED ${SOAPY_SDR_SOURCES})
if (SOAPY_SDR_LIBRARIES)
    target_link_libraries(SoapySDR ${SOAPY_SDR_LIBRARIES})
endif()
set_target_properties(SoapySDR PROPERTIES SOVERSION ${SOAPY_SDR_ABI_VERSION})
set_target_properties(SoapySDR PROPERTIES VERSION ${SOAPY_SDR_LIBVER})
set_target_properties(SoapySDR PROPERTIES DEFINE_SYMBOL "SOAPY_SDR_DLL_EXPORTS")

########################################################################
# Install the library
########################################################################
install(TARGETS SoapySDR
    LIBRARY DESTINATION lib${LIB_SUFFIX} # .so file
    ARCHIVE DESTINATION lib${LIB_SUFFIX} # .lib file
    RUNTIME DESTINATION bin              # .dll file
)

########################################################################
# Build pkg config file
########################################################################
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/SoapySDR.in.pc
    ${CMAKE_CURRENT_BINARY_DIR}/SoapySDR.pc
@ONLY)

install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/SoapySDR.pc
    DESTINATION lib${LIB_SUFFIX}/pkgconfig
)
