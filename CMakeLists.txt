PROJECT( CreateStainVectorProfile )
CMAKE_MINIMUM_REQUIRED( VERSION 3.13 )
# Enable C++17 features
SET(CMAKE_CXX_STANDARD 17)
INCLUDE(FetchContent)

#Note which version of Sedeen Viewer this plugin was last compiled and tested with
SET(SEDEENSDK_VERSION "5.4.3.20190808" CACHE STRING "Last version of Sedeen Viewer the plugin was compiled and tested with")

# Define project description variables
SET( DISPLAY_NAME_TEXT "Create Stain Vector Profile" CACHE STRING "Name of the plugin as it should be displayed in Sedeen Viewer")
SET( SUPPORT_URL_TEXT "http://pathcore.com/support/plugin/info/${PROJECT_NAME}" CACHE STRING "Location users can find help with the plugin" )
SET( DEVELOPER_TEXT "Sunnybrook Research Institute" CACHE STRING "Name of the author or organization that created the plugin" )

# Load the Sedeen dependencies
SET(PROGRAMFILESX86 "PROGRAMFILES\(X86\)")
FIND_PACKAGE( SEDEENSDK REQUIRED 
  HINTS ../../.. 
        "$ENV{${PROGRAMFILESX86}}/Sedeen Viewer SDK/v5.4.3.20190808/msvc2017"
        "$ENV{PROGRAMFILES}/Sedeen Viewer SDK/v5.4.3.20190808/msvc2017" )
        
IF(NOT BOOST_ROOT)
  SET(BOOST_ROOT "BOOST_ROOT-NOTFOUND" CACHE PATH "Preferred installation prefix of the Boost C++ library")
ENDIF()
IF(NOT BOOST_VERSION)
  SET(BOOST_VERSION "BOOST_VERSION-NOTFOUND" CACHE STRING "Boost library version number")
ENDIF()

FIND_PACKAGE(Boost ${BOOST_VERSION} )

# Fetch TinyXML2 files. Do not build as a subproject
FetchContent_Declare(
  TinyXML2
  GIT_REPOSITORY https://github.com/leethomason/tinyxml2.git
  GIT_TAG 61a4c7d507322c9f494f5880d4c94b60e4ce9590
)
# Check if the tinyxml2 files have already been populated
FetchContent_GetProperties(TinyXML2)
STRING(TOLOWER "TinyXML2" TinyXML2Name)
IF(NOT ${TinyXML2Name}_POPULATED)
  #Fetch TinyXML2 using the details from FetchContent_Declare 	
  FetchContent_Populate(TinyXML2)
ENDIF()

# Fetch the StainAnalysis-plugin repository. Do not build as a subproject
FetchContent_Declare(
  StainAnalysisplugin
  GIT_REPOSITORY https://github.com/sedeen-piip-plugins/StainAnalysis-plugin.git
  GIT_TAG 26cd478479c18dfd2031ba300e294f4c3e374b64
)
FetchContent_GetProperties(StainAnalysisplugin)
STRING(TOLOWER "StainAnalysisplugin" StainAnalysispluginName)
IF(NOT ${StainAnalysispluginName}_POPULATED)
  FetchContent_Populate(StainAnalysisplugin)
ENDIF()
SET(STAIN_ANALYSIS_DIR ${${StainAnalysispluginName}_SOURCE_DIR})

INCLUDE_DIRECTORIES( ${INCLUDE_DIRECTORIES} 
                     ${SEDEENSDK_INCLUDE_DIR} 
                     ${BOOST_ROOT} 
                     ${${TinyXML2Name}_SOURCE_DIR}
                     ${STAIN_ANALYSIS_DIR}
                     )

LINK_DIRECTORIES( ${LINK_DIRECTORIES} 
                  ${SEDEENSDK_LIBRARY_DIR} 
                  )

ADD_LIBRARY( ${PROJECT_NAME} MODULE 
             ${PROJECT_NAME}.cpp 
             ${PROJECT_NAME}.h 
             ${${TinyXML2Name}_SOURCE_DIR}/tinyxml2.h 
             ${${TinyXML2Name}_SOURCE_DIR}/tinyxml2.cpp
             ${STAIN_ANALYSIS_DIR}/StainProfile.h 
             ${STAIN_ANALYSIS_DIR}/StainProfile.cpp 
             ${STAIN_ANALYSIS_DIR}/ColorDeconvolutionKernel.h 
             ${STAIN_ANALYSIS_DIR}/ColorDeconvolutionKernel.cpp 
             ${STAIN_ANALYSIS_DIR}/StainVectorMath.h 
             ${STAIN_ANALYSIS_DIR}/StainVectorMath.cpp
             )

# Link the library against the Sedeen SDK libraries
TARGET_LINK_LIBRARIES( ${PROJECT_NAME} ${SEDEENSDK_LIBRARIES} )

# Create or update the .info file in the build directory
STRING( TIMESTAMP DATE_CREATED_TEXT "%Y-%m-%d" )
CONFIGURE_FILE( "infoTemplate.info.in" "${PROJECT_NAME}.info" )

# Set the relative directory where the plugin should be located
SET( PLUGIN_RELATIVE_DIR "plugins/cpp/piip/${PROJECT_NAME}" )

# Set the install destination directory
IF( NOT PLUGIN_DESTINATION_DIR )
  IF( ${SEDEEN_FOUND} )
	SET( TEMPPLUGINDIR "${PATHCORE_DIR}/${PLUGIN_RELATIVE_DIR}" )
  ELSE()
	SET( TEMPPLUGINDIR "PLUGIN_DESTINATION_DIR-NOTFOUND" )
	MESSAGE( SEND_ERROR "PLUGIN_DESTINATION_DIR not found. Set this to the target installation directory of the plugin within Sedeen Viewer (e.g. $ENV{PROGRAMFILES}/Sedeen Viewer/plugins/cpp/piip/${PROJECT_NAME}).")
  ENDIF()
  SET(PLUGIN_DESTINATION_DIR ${TEMPPLUGINDIR} CACHE PATH "Installation directory for the plugin within Sedeen Viewer")
ENDIF()

# Install the plugin and .info file in the PLUGIN_DESTINATION_DIR directory
IF( ${SEDEEN_FOUND} )
  INSTALL(TARGETS ${PROJECT_NAME}
      LIBRARY DESTINATION "${PLUGIN_DESTINATION_DIR}")
  INSTALL(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.info"
      DESTINATION "${PLUGIN_DESTINATION_DIR}")
ENDIF()

#For debugging: shows all variables and their values
#get_cmake_property(_variableNames VARIABLES)
#list (SORT _variableNames)
#foreach (_variableName ${_variableNames})
#    message(STATUS "${_variableName}=${${_variableName}}")
#endforeach()
