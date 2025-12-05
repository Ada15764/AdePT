# SPDX-FileCopyrightText: 2023 CERN
# SPDX-License-Identifier: Apache-2.0

SET(CMAKE_EXE_LINKER_FLAGS "-Wl,--disable-new-dtags")

if(NOT TARGET G4HepEm::g4HepEm)
  message(STATUS "Disabling example1 (needs G4HepEm)")
  return()
endif()

if(Geant4_FOUND)
  if(NOT Geant4_gdml_FOUND)
    message(STATUS "Disabling example1 (needs Geant4 with GDML support)")
    return()
  endif()
  if(NOT AdePT_HAS_G4VTRACKINGMANAGER)
    message(STATUS "Disabling example1 (needs G4VTrackingManger interface available starting with geant4-11-00)")
    return()
  endif()
else()
  message(STATUS "Disabling example1 (needs Geant4)")
  return()
endif()

set(sources_g4
  ${PROJECT_SOURCE_DIR}/../../examples/common/src/ParticleGun.cc
  ${PROJECT_SOURCE_DIR}/../../examples/common/src/ParticleGunMessenger.cc
  ${PROJECT_SOURCE_DIR}/../../examples/common/src/FTFP_BERT_HepEm.cc
  ${PROJECT_SOURCE_DIR}/../../examples/common/src/FTFP_BERT_AdePT.cc
  src/ATLTileCalTBActInitialization.cc
  src/ATLTileCalTBHit.cc 
  src/ATLTileCalTBStepAction.cc
  src/ATLTileCalTBDetConstruction.cc
  src/ATLTileCalTBPrimaryGenAction.cc
  src/FLUKAHadronInelasticPhysics.cc
  src/ATLTileCalTBEventAction.cc
  src/ATLTileCalTBRunAction.cc
  src/G4_CernFLUKAHadronInelastic_FTFP_BERT.cc
  src/ATLTileCalTBGeometry.cc
  src/ATLTileCalTBSensDet.cc
  src/SpectrumAnalyzer.cc
)

if(HepMC3_FOUND)
  set(sources_hepmc3
    ${PROJECT_SOURCE_DIR}/../../examples/common/src/HepMC3G4AsciiReader.cc
    ${PROJECT_SOURCE_DIR}/../../examples/common/src/HepMC3G4AsciiReaderMessenger.cc
    ${PROJECT_SOURCE_DIR}/../../examples/common/src/HepMC3G4Interface.cc
  )
endif()

# example1
add_executable(ATLAS_TileCalTB ATLAS_TileCalTB.cpp ${sources_g4} ${sources_hepmc3})

target_include_directories(ATLAS_TileCalTB
  PRIVATE
  ${PROJECT_SOURCE_DIR}/ATLAS_TileCalTB/include
  ${PROJECT_SOURCE_DIR}/ATLAS_TileCalTB
    ${PROJECT_SOURCE_DIR}/../../examples/common/include
    ${HEPMC3_INCLUDE_DIR}
)
cuda_rdc_target_link_libraries(ATLAS_TileCalTB
  PRIVATE
    ${HEPMC3_LIBRARIES}
    ${HEPMC3_FIO_LIBRARIES}
    AdePT_G4_integration
)

# Install macros and geometry file
SET(GDML ${PROJECT_BINARY_DIR}/cms2018_sd.gdml)
configure_file("macros/gui.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/gui.mac")
configure_file("macros/icons.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/icons.mac")
configure_file("macros/init_vis.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/init_vis.mac")
configure_file("macros/single.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/single.mac")
configure_file("macros/TBrun_all.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/TBrun_all.mac")
configure_file("macros/TBrun.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/TBrun.mac")
configure_file("macros/vis.mac.in" "${PROJECT_BINARY_DIR}/Tilecal/vis.mac")

# Tests

add_test(NAME TileCalTB-g4vg
  COMMAND $<TARGET_FILE:ATLAS_TileCalTB> -m ${PROJECT_BINARY_DIR}/example1.mac
)
