#
# Copyright (c) Contributors to the Open 3D Engine Project.
# For complete copyright and license terms please see the LICENSE at the root of this distribution.
#
# SPDX-License-Identifier: Apache-2.0 OR MIT
#
#

ly_add_target(
    NAME ${gem_name}.Static STATIC
    NAMESPACE Gem
    FILES_CMAKE
        surfacedata_files.cmake
    INCLUDE_DIRECTORIES
        PRIVATE
            Source
        PUBLIC
            Include
    BUILD_DEPENDENCIES
        PRIVATE
            Gem::LmbrCentral
        PUBLIC
            AZ::AzCore
            AZ::AzFramework
            Gem::Atom_RPI.Public
            Gem::Atom_Feature_Common.Public
)

ly_add_target(
    NAME ${gem_name} ${PAL_TRAIT_MONOLITHIC_DRIVEN_MODULE_TYPE}
    NAMESPACE Gem
    FILES_CMAKE
        surfacedata_shared_files.cmake
    INCLUDE_DIRECTORIES
        PRIVATE
            Source
        PUBLIC
            Include
    BUILD_DEPENDENCIES
        PUBLIC
            Gem::${gem_name}.Static
            Gem::LmbrCentral.API
    RUNTIME_DEPENDENCIES
        Gem::LmbrCentral
)

# Inject the gem name into the Module source file
ly_add_source_properties(
    SOURCES
        Source/SurfaceDataModule.cpp
    PROPERTY COMPILE_DEFINITIONS
        VALUES
            O3DE_GEM_NAME=${gem_name}
            O3DE_GEM_VERSION=${gem_version})

# the above module is for use in all client/server types 
ly_create_alias(NAME ${gem_name}.Servers  NAMESPACE Gem TARGETS Gem::${gem_name})
ly_create_alias(NAME ${gem_name}.Clients  NAMESPACE Gem TARGETS Gem::${gem_name})
ly_create_alias(NAME ${gem_name}.Unified  NAMESPACE Gem TARGETS Gem::${gem_name})

if (PAL_TRAIT_BUILD_HOST_TOOLS)

    ly_add_target(
        NAME ${gem_name}.Editor GEM_MODULE

        NAMESPACE Gem
        FILES_CMAKE
            surfacedata_editor_files.cmake
        COMPILE_DEFINITIONS
            PRIVATE
                SURFACEDATA_EDITOR
        INCLUDE_DIRECTORIES
            PRIVATE
                Source
            PUBLIC
                Include
        BUILD_DEPENDENCIES
            PRIVATE
                AZ::AzToolsFramework
                Gem::${gem_name}.Static
                Gem::LmbrCentral.Editor.Static
        RUNTIME_DEPENDENCIES
            Gem::LmbrCentral.Editor
    )

    # Inject the gem name into the Module source file
    ly_add_source_properties(
        SOURCES
            Source/SurfaceDataEditorModule.cpp
        PROPERTY COMPILE_DEFINITIONS
            VALUES
                O3DE_GEM_NAME=${gem_name}
                O3DE_GEM_VERSION=${gem_version})

    # the above module is for use in dev tool situations
    ly_create_alias(NAME ${gem_name}.Builders NAMESPACE Gem TARGETS Gem::${gem_name}.Editor)
    ly_create_alias(NAME ${gem_name}.Tools    NAMESPACE Gem TARGETS Gem::${gem_name}.Editor)

endif()

################################################################################
# Tests
################################################################################
if(PAL_TRAIT_BUILD_TESTS_SUPPORTED)

    ly_add_target(
        NAME ${gem_name}.Tests ${PAL_TRAIT_TEST_TARGET_TYPE}
        NAMESPACE Gem
        FILES_CMAKE
            surfacedata_tests_files.cmake
        INCLUDE_DIRECTORIES
            PRIVATE
                .
                Tests
                Source
        BUILD_DEPENDENCIES
            PRIVATE
                AZ::AzTest
                Gem::${gem_name}.Static
                Gem::LmbrCentral
        RUNTIME_DEPENDENCIES
            Gem::LmbrCentral
    )
    ly_add_googletest(
        NAME Gem::${gem_name}.Tests
        LABELS REQUIRES_tiaf
    )
    ly_add_googlebenchmark(
        NAME Gem::${gem_name}.Benchmarks
        TARGET Gem::${gem_name}.Tests
    )
endif()
