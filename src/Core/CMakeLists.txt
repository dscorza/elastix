#---------------------------------------------------------------------
project( elxCore )

#---------------------------------------------------------------------
# Define lists of files in the subdirectories.

set( KernelFilesForExecutables
  Kernel/elxElastixMain.cxx
  Kernel/elxElastixMain.h
  Kernel/elxTransformixMain.cxx
  Kernel/elxTransformixMain.h
)

set( KernelFilesForComponents
  Kernel/elxElastixBase.cxx
  Kernel/elxElastixBase.h
  Kernel/elxElastixTemplate.h
  Kernel/elxElastixTemplate.hxx
)

set( InstallFilesForExecutables
  Install/elxComponentLoader.cxx
  Install/elxComponentLoader.h
)

set( InstallFilesForComponents
  Install/elxComponentDatabase.cxx
  Install/elxComponentDatabase.h
  Install/elxBaseComponent.cxx
  Install/elxBaseComponent.h
  Install/elxBaseComponentSE.h
  Install/elxBaseComponentSE.hxx
  Install/elxInstallAllComponents.h
  Install/elxInstallFunctions.h
  Install/elxMacro.h
  Install/elxIncludes.h
)

set( ConfigurationFiles
  Configuration/elxConfiguration.cxx
  Configuration/elxConfiguration.h
)

set( ComponentBaseClassFiles
  ComponentBaseClasses/elxFixedImagePyramidBase.h
  ComponentBaseClasses/elxImageSamplerBase.h
  ComponentBaseClasses/elxInterpolatorBase.h
  ComponentBaseClasses/elxMetricBase.h
  ComponentBaseClasses/elxMovingImagePyramidBase.h
  ComponentBaseClasses/elxOptimizerBase.h
  ComponentBaseClasses/elxRegistrationBase.h
  ComponentBaseClasses/elxResampleInterpolatorBase.h
  ComponentBaseClasses/elxResamplerBase.h
  ComponentBaseClasses/elxTransformBase.h
  ComponentBaseClasses/elxFixedImagePyramidBase.hxx
  ComponentBaseClasses/elxImageSamplerBase.hxx
  ComponentBaseClasses/elxInterpolatorBase.hxx
  ComponentBaseClasses/elxMetricBase.hxx
  ComponentBaseClasses/elxMovingImagePyramidBase.hxx
  ComponentBaseClasses/elxOptimizerBase.hxx
  ComponentBaseClasses/elxRegistrationBase.hxx
  ComponentBaseClasses/elxResampleInterpolatorBase.hxx
  ComponentBaseClasses/elxResamplerBase.hxx
  ComponentBaseClasses/elxTransformBase.hxx
)

set( ProgressCommandFiles
  elxProgressCommand.cxx
  elxProgressCommand.h
)

#---------------------------------------------------------------------
# Construct source groups for nice visualisation in Visual Studio.

source_group( "Install" FILES
  ${InstallFilesForComponents}
  ${InstallFilesForExecutables}
)

source_group( "Kernel" FILES
  ${KernelFilesForComponents}
  ${KernelFilesForExecutables}
)

source_group( "Configuration" FILES ${ConfigurationFiles} )
source_group( "ComponentBaseClasses" FILES ${ComponentBaseClassFiles} )
source_group( "ProgressCommand" FILES ${ProgressCommandFiles} )

#---------------------------------------------------------------------
# Create the elxCore library.

add_library( elxCore
  ${KernelFilesForComponents}
  ${InstallFilesForComponents}
  ${ConfigurationFiles}
  ${ComponentBaseClassFiles}
  ${ProgressCommandFiles}
)

#---------------------------------------------------------------------
# Link against other libraries.

target_link_libraries( elxCore
  elxCommon
  xoutlib
  param # Needed for elxConfiguration
  #  ${ITK_LIBRARIES}
)

#---------------------------------------------------------------------
# Define the mevis dcm tiff lib to which we should link.
set( mevisdcmtifflib mevisdcmtiff )

#---------------------------------------------------------------------
# Create the elastix executable or library.
if( ELASTIX_BUILD_EXECUTABLE )
  add_executable( elastix
    Main/elastix.cxx
    Main/elastix.h
    Kernel/elxElastixMain.cxx
    Kernel/elxElastixMain.h
    ${InstallFilesForExecutables}
  )
else()
  if( ELASTIX_BUILD_SHARED_LIBS )
    add_library( elastix SHARED
      Main/elastixlib.cxx
      Main/elastixlib.h
      Kernel/elxElastixMain.cxx
      Kernel/elxElastixMain.h
      ${InstallFilesForExecutables}
    )
  else()
    add_library( elastix STATIC
      Main/elastixlib.cxx
      Main/elastixlib.h
      Kernel/elxElastixMain.cxx
      Kernel/elxElastixMain.h
      ${InstallFilesForExecutables}
    )
  endif()
endif()

#---------------------------------------------------------------------
# Create the transformix executable.

if( ELASTIX_BUILD_EXECUTABLE )
  add_executable( transformix
    Main/transformix.cxx
    Main/elastix.h
    Kernel/elxElastixMain.cxx
    Kernel/elxElastixMain.h
    Kernel/elxTransformixMain.cxx
    Kernel/elxTransformixMain.h
    ${InstallFilesForExecutables}
  )
else()
  if( ELASTIX_BUILD_SHARED_LIBS )
    add_library( transformix SHARED
      Main/transformixlib.cxx
      Main/transformixlib.h
      Kernel/elxElastixMain.cxx
      Kernel/elxElastixMain.h
      Kernel/elxTransformixMain.cxx
      Kernel/elxTransformixMain.h
      ${InstallFilesForExecutables}
    )
  else()
    add_library( transformix STATIC
      Main/transformixlib.cxx
      Main/transformixlib.h
      Kernel/elxElastixMain.cxx
      Kernel/elxElastixMain.h
      Kernel/elxTransformixMain.cxx
      Kernel/elxTransformixMain.h
      ${InstallFilesForExecutables}
    )
  endif()
endif()

#---------------------------------------------------------------------
# Link elastix against other libraries.

target_link_libraries( elastix
  param
  xoutlib
  elxCommon
  elxCore
  ${mevisdcmtifflib}
  ${AllComponentLibs}
  ${ITK_LIBRARIES}
)

#---------------------------------------------------------------------
# Link transformix against other libraries.

target_link_libraries( transformix
  param
  xoutlib
  elxCommon
  elxCore
  ${mevisdcmtifflib}
  ${AllComponentLibs}
  ${ITK_LIBRARIES}
)

if( MSVC )
  # NOTE: that linker /INCREMENTAL:NO flag makes it impossible to use
  # Debug breakpoints in Visual Studio 10.0. It is probably the Visual Studio 10.0 bug.
  # Or you have to recompile ITK with the same flag in Debug as well.
  # Or it has something to do with building it with IncrediBuild where this flag is not welcome for PDB files.
  # Therefore we are going to set it only for Visual Studio 9.0 (2008)
  if( MSVC90 OR MSVC91 )
    set( CMAKE_EXE_LINKER_FLAGS_DEBUG "/INCREMENTAL:NO" )
  endif()
endif()

#---------------------------------------------------------------------
# Define the install directory for elastix and transformix.

if( WIN32 )
  if( ELASTIX_BUILD_EXECUTABLE )
    install( TARGETS elastix transformix
      RUNTIME DESTINATION . )
      #COMPONENT core )
  else()
    # transformix is not implemented as library yet
    install( TARGETS elastix transformix
      RUNTIME DESTINATION .
      ARCHIVE DESTINATION . )
  endif()
else()
  # Tell the executables where to find the required .so files.
  set_target_properties( elastix transformix
    PROPERTIES INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib:${ITK_DIR}" )
  # Put executables in bin and libraries in lib.
  install( TARGETS elastix transformix
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION bin
    ARCHIVE DESTINATION bin)
    #COMPONENT core )
endif()

#---------------------------------------------------------------------
# The Core/Install directory contains a CMakeLists file for
# defining the elastix compilation types.

add_subdirectory( Install )

