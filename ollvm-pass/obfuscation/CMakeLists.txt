add_library(ollvm SHARED 
    ConstantFPEncryption.cpp
    ConstantIntEncryption.cpp
    CryptoUtils.cpp
    Utils.cpp
    ObfuscationPassManager.cpp
    ObfuscationOptions.cpp
    IndirectBranch.cpp
    IndirectCall.cpp
    IndirectGlobalVariable.cpp
    Flattening.cpp
    StringEncryption.cpp
    LegacyLowerSwitch.cpp
    obfuscation.def
    )

target_include_directories(ollvm PRIVATE ${CMAKE_SOURCE_DIR}/obfuscation)

add_dependencies(ollvm intrinsics_gen LLVMLinker)

llvm_map_components_to_libnames(llvm_libs support core irreader linker)
target_link_libraries(ollvm PRIVATE ${llvm_libs})

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set_target_properties(ollvm PROPERTIES
        LINK_FLAGS "-static -static-libgcc -Wl,-Bstatic,--whole-archive -lwinpthread -lstdc++ -Wl,--no-whole-archive -Wl,-Bdynamic"
    )
endif()

# Add a post-build command to move the DLL to the root directory
add_custom_command(TARGET ollvm POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E rename
            $<TARGET_FILE:ollvm>
            ${CMAKE_SOURCE_DIR}/../$<TARGET_FILE_NAME:ollvm>
    COMMENT "Moving ollvm.dll to the root directory"
)
