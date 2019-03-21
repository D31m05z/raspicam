INCLUDE_DIRECTORIES(.)
SET(private_hdrs_base "private/private_types.h  private/private_impl.h exceptions.h private/threadcondition.h ")
SET(private_still_hdrs_base "private_still/private_still_types.h  private/private_still_impl.h")

SET(public_hdrs_base raspicamtypes.h raspicam.h)

SET(srcs_base raspicam.cpp raspicam_still.cpp  private/private_impl.cpp private/threadcondition.cpp  private_still/private_still_impl.cpp)
if(NOT( ${CMAKE_SYSTEM_PROCESSOR} MATCHES arm*) )#in a pc, adds fake dependencies to mmal functions to enable compilation
SET(srcs_base ${srcs_base} private/fake_mmal_dependencies.cpp)
endif()
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR} )

find_package(Threads REQUIRED)

add_library(${PROJECT_NAME} ${hdrs_base} ${srcs_base} )
add_library(::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX d)

SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES 				# create *nix style library versions + symbolic links
	DEFINE_SYMBOL DSO_EXPORTS
	VERSION ${PROJECT_VERSION}
	SOVERSION ${PROJECT_SOVERSION}
	CLEAN_DIRECT_OUTPUT 1 										# allow creating static and shared libs without conflicts
	OUTPUT_NAME "${PROJECT_NAME}${PROJECT_DLLVERSION}" 	# avoid conflicts between library and binary target names
)
TARGET_LINK_LIBRARIES(${PROJECT_NAME}  ${REQUIRED_LIBRARIES}  Threads::Threads)

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>
    $<INSTALL_INTERFACE:include>
)

configure_file(${PROJECT_NAME}-config.cmake.in ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config.cmake @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config.cmake DESTINATION lib/cmake/${PROJECT_NAME})
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}-targets DESTINATION lib)
install(EXPORT ${PROJECT_NAME}-targets NAMESPACE :: DESTINATION lib/cmake/${PROJECT_NAME})

#Add opencv component if required
#Opencv Lib
IF (${OpenCV_FOUND})
message(STATUS "Adding cv library")
SET(hdrs_cv raspicam_cv.h raspicam_still_cv.h)
SET(srcs_cv raspicam_cv.cpp raspicam_still_cv.cpp)
ADD_LIBRARY(${PROJECT_NAME}_cv ${hdrs_cv} ${srcs_cv} ${srcs_base})
SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES 				# create *nix style library versions + symbolic links
	DEFINE_SYMBOL DSO_EXPORTS
	VERSION ${PROJECT_VERSION}
	SOVERSION ${PROJECT_SOVERSION}
	CLEAN_DIRECT_OUTPUT 1 										# allow creating static and shared libs without conflicts
	OUTPUT_NAME "${PROJECT_NAME}${PROJECT_DLLVERSION}" 	# avoid conflicts between library and binary target names
)
TARGET_LINK_LIBRARIES(${PROJECT_NAME}_cv  ${REQUIRED_LIBRARIES}  ${OpenCV_LIBS} )
INSTALL(TARGETS ${PROJECT_NAME}_cv
	RUNTIME DESTINATION bin COMPONENT main
	LIBRARY DESTINATION lib PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE COMPONENT main
	ARCHIVE DESTINATION lib COMPONENT main)
ENDIF()


#Installation of all header files
INSTALL(FILES ${public_hdrs_base} ${hdrs_cv}
	DESTINATION include/${PROJECT_NAME}
	COMPONENT main)
