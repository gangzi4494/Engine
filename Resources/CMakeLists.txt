CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
Project(ResourcePacker)

function(GroupDirectories dirPrefix groupPrefix targetDir sources)
	get_filename_component(currentDirName ${targetDir} NAME)
	string(CONCAT currentDirFullName "${dirPrefix}${currentDirName}/")
	string(CONCAT currentDirGroupName "${groupPrefix}${currentDirName}")
	file(GLOB allFiles "${targetDir}/*")
	file(GLOB allCFiles "${targetDir}/*.c")
	file(GLOB allHFiles "${targetDir}/*.h*")
	file(GLOB allCXXFiles "${targetDir}/*.c*")
	
	list(LENGTH allFiles  argv_len)
	list(LENGTH allCFiles lenC)
	list(LENGTH allCXXFiles lenCXX)
	list(LENGTH allHFiles lenH)
	
	set(allUsefulFileCount 0)
	math(EXPR allUsefulFileCount "${lenC}+${lenCXX}+${lenH}")
	if(0 LESS allUsefulFileCount)
		set(mGroups ${allCFiles} ${allCXXFiles} ${allHFiles})
		source_group(${currentDirGroupName} FILES ${mGroups})
		message(STATUS "${currentDirGroupName} c:${lenC} cxx:${lenCXX} h:${lenH} total:${argv_len}")
	endif()
    set(i 0)
    while( i LESS ${argv_len})
         list(GET allFiles ${i} argv_value)
		 if(IS_DIRECTORY ${argv_value})
			#deal with directory
			#message(STATUS "${argv_value}")
			string(CONCAT currentGroupPrefix "${groupPrefix}${currentDirName}\\")
			GroupDirectories(${currentDirFullName} ${currentGroupPrefix} ${argv_value} subGroups)
			list(APPEND mGroups ${subGroups})
		 else()
			#deal with files
			#message(STATUS "file ${argv_value}")
		 endif()
         math(EXPR i "${i} + 1")  
    endwhile()
	set(${sources} ${mGroups} PARENT_SCOPE)
endfunction()

if(WIN32)
	include_directories(${PROJECT_SOURCE_DIR}/../)
	include_directories(${PROJECT_SOURCE_DIR}/../External/Box2D/include)
	include_directories(${PROJECT_SOURCE_DIR}/../External/PBC/src)
	include_directories(${PROJECT_SOURCE_DIR}/../External/ProtoBuffer3.13/src)
	include_directories(${PROJECT_SOURCE_DIR}/../External/Lua5.1.4/src)
	include_directories(${PROJECT_SOURCE_DIR}/../External/PhysX3.4/PhysX_3.4/Include)
	include_directories(${PROJECT_SOURCE_DIR}/../External/PhysX3.4/PxShared/include)
	include_directories(${PROJECT_SOURCE_DIR}/../External/zlib)
	include_directories(${PROJECT_SOURCE_DIR}/../External/LuaSocket/src)
	include_directories(${PROJECT_SOURCE_DIR}/../External/LibZipRel/lib)
	include_directories(${PROJECT_SOURCE_DIR}/../External/LibZipRel/build)
	include_directories(${PROJECT_SOURCE_DIR}/../External/LibCurl/include)
	include_directories(${PROJECT_SOURCE_DIR}/../External/FreeType/include)
	
	set(ResourcePackerSrcs ${PROJECT_SOURCE_DIR}/../Platform/Windows/Common/PlatformWindowsSeriallizer.pb.cc)
	list(APPEND ResourcePackerSrcs ${PROJECT_SOURCE_DIR}/src/main.cpp)

	add_compile_definitions(ALICE_WIN_PLAYER _DEBUG UNICODE _UNICODE _SCL_SECURE_NO_WARNINGS _CRT_SECURE_NO_WARNINGS)

	set(CMAKE_CXX_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG")
	set(CMAKE_CXX_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1")
	set(CMAKE_CXX_FLAGS "/DWIN32 /D_WINDOWS /W3 /GR /EHsc")
	set(CMAKE_C_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG")
	set(CMAKE_C_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1")
	set(CMAKE_C_FLAGS "/DWIN32 /D_WINDOWS /W3")

	link_directories(${PROJECT_SOURCE_DIR}/../External/PrebuiltLibs/x86)
	add_executable(${PROJECT_NAME} ${ResourcePackerSrcs})
	target_link_libraries(${PROJECT_NAME} gdiplus libprotobuf Imm32)
	set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME "ResourcePacker")
	set_target_properties(${PROJECT_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}")
ENDIF()