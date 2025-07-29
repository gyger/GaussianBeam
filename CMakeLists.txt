cmake_minimum_required(VERSION 3.16)
project(GaussianBeam VERSION 0.5.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set C standard for C files
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Define mathematical constants for Windows
if(WIN32)
    add_definitions(-D_USE_MATH_DEFINES)
endif()

# Enable testing
enable_testing()

# Set output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Find Qt5 components
find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets Xml XmlPatterns LinguistTools)

# Platform-specific settings
if(APPLE)
    set(CMAKE_OSX_DEPLOYMENT_TARGET 10.15)
    set(CMAKE_OSX_ARCHITECTURES "x86_64;arm64")
endif()

if(WIN32)
    set(CMAKE_WIN32_EXECUTABLE ON)
endif()

# Compiler-specific flags
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        add_compile_options(-Wno-long-long)
    endif()
endif()

# Source files
set(SOURCES
    src/GaussianBeam.cpp
    src/Optics.cpp
    src/OpticsBench.cpp
    src/GaussianFit.cpp
    src/Function.cpp
    src/OpticsFunction.cpp
    src/Cavity.cpp
    src/Utils.cpp
    src/lmmin.c
)
set_source_files_properties(src/lmmin.c PROPERTIES LANGUAGE C)

set(GUI_SOURCES
    gui/GaussianBeamWidget.cpp
    gui/OpticsView.cpp
    gui/OpticsWidgets.cpp
    gui/GaussianBeamDelegate.cpp
    gui/GaussianBeamModel.cpp
    gui/GaussianBeamWindow.cpp
    gui/Unit.cpp
    gui/Names.cpp
    gui/GaussianBeamSave.cpp
    gui/GaussianBeamLoad.cpp
    gui/main.cpp
)

set(UI_FILES
    gui/GaussianBeamWidget.ui
    gui/GaussianBeamWindow.ui
    gui/OpticsViewProperties.ui
)

set(HEADERS
    gui/GaussianBeamDelegate.h
    gui/GaussianBeamModel.h
    gui/GaussianBeamWidget.h
    gui/GaussianBeamWindow.h
    gui/OpticsView.h
    gui/OpticsWidgets.h
)

set(RESOURCE_FILES
    gui/GaussianBeam.qrc
)

# Create executable
add_executable(gaussianbeam
    ${SOURCES}
    ${GUI_SOURCES}
    ${UI_FILES}
    ${HEADERS}
    ${RESOURCE_FILES}
)

# Set target properties
set_target_properties(gaussianbeam PROPERTIES
    OUTPUT_NAME "GaussianBeam"
    WIN32_EXECUTABLE TRUE
)

# Link Qt libraries
target_link_libraries(gaussianbeam PRIVATE
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::Xml
    Qt5::XmlPatterns
)

# Include directories
target_include_directories(gaussianbeam PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# Enable automoc, autorcc, and autouic
set_target_properties(gaussianbeam PROPERTIES
    AUTOMOC ON
    AUTORCC ON
    AUTOUIC ON
)

# Handle Windows icon
if(WIN32)
    if(MINGW)
        add_custom_command(
            OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/GaussianBeamIco.o
            COMMAND windres.exe -I${CMAKE_CURRENT_SOURCE_DIR} -i${CMAKE_CURRENT_SOURCE_DIR}/gui/GaussianBeam.rc -o ${CMAKE_CURRENT_BINARY_DIR}/GaussianBeamIco.o
            DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/gui/GaussianBeam.rc
        )
        target_sources(gaussianbeam PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/GaussianBeamIco.o)
    else()
        target_sources(gaussianbeam PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/gui/GaussianBeam.rc)
    endif()
endif()

# Translations
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/po/GaussianBeam_fr.ts)
    qt5_create_translation(QM_FILES ${CMAKE_CURRENT_SOURCE_DIR}/gui ${CMAKE_CURRENT_SOURCE_DIR}/po/GaussianBeam_fr.ts)
    add_custom_target(translations DEPENDS ${QM_FILES})
    add_dependencies(gaussianbeam translations)
    
    # Copy translation file to po directory for resource file
    add_custom_command(
        TARGET translations POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_CURRENT_BINARY_DIR}/GaussianBeam_fr.qm
        ${CMAKE_CURRENT_SOURCE_DIR}/po/GaussianBeam_fr.qm
    )
    
    # Copy the .qm file to the gui/po directory for the resource file
    add_custom_command(
        TARGET translations POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_CURRENT_BINARY_DIR}/GaussianBeam_fr.qm
        ${CMAKE_CURRENT_SOURCE_DIR}/gui/po/GaussianBeam_fr.qm
    )
endif()

# Install rules
install(TARGETS gaussianbeam
    RUNTIME DESTINATION bin
    BUNDLE DESTINATION .
)

# Install additional files
install(FILES
    README.md
    LICENSE
    COPYING
    AUTHORS
    DESTINATION share/doc/gaussianbeam
)

# CPack configuration
set(CPACK_PACKAGE_NAME "GaussianBeam")
set(CPACK_PACKAGE_VENDOR "GaussianBeam Team")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "GUI software for simulating Gaussian laser beams")
set(CPACK_PACKAGE_CONTACT "Jérôme Lodewyck (jerome dot lodewyck at normalesup dot org)")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

# Platform-specific packaging
if(WIN32)
    set(CPACK_GENERATOR "NSIS;ZIP")
    set(CPACK_NSIS_DISPLAY_NAME "GaussianBeam")
    set(CPACK_NSIS_PACKAGE_NAME "GaussianBeam")
    set(CPACK_NSIS_CONTACT "${CPACK_PACKAGE_CONTACT}")
    set(CPACK_PACKAGE_EXECUTABLES "gaussianbeam" "GaussianBeam")
elseif(APPLE)
    set(CPACK_GENERATOR "DragNDrop;TGZ")
    set(CPACK_BUNDLE_NAME "GaussianBeam")
    set(CPACK_BUNDLE_ICON "${CMAKE_CURRENT_SOURCE_DIR}/gui/images/gaussianbeam128.png")
    set(CPACK_BUNDLE_PLIST "${CMAKE_CURRENT_BINARY_DIR}/Info.plist")
else()
    set(CPACK_GENERATOR "DEB;RPM;TGZ")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_CONTACT}")
    set(CPACK_DEBIAN_PACKAGE_SECTION "science")
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libqt5core5a, libqt5gui5, libqt5widgets5, libqt5xml5, libqt5xmlpatterns5")
    set(CPACK_RPM_PACKAGE_LICENSE "GPL")
    set(CPACK_RPM_PACKAGE_GROUP "Applications/Scientific")
endif()

include(CPack)
