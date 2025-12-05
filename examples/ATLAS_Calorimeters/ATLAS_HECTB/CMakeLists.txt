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
  src/ATLHECTBActionInitialization.cc
  src/ATLHECTBDetectorConstruction.cc
  src/ATLHECTBEventAction.cc
  src/ATLHECTBSteppingAction.cc
  src/ATLHECTBSignalCalculator.cc
  src/ATLHECTBPrimaryGeneratorAction.cc
  src/ATLHECTBRunAction.cc
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
add_executable(ATLAS_HECTB ATLAS_HECTB.cpp ${sources_g4} ${sources_hepmc3})

message(PROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR})

target_include_directories(ATLAS_HECTB
  PRIVATE
  ${PROJECT_SOURCE_DIR}/ATLAS_HECTB/include
  ${PROJECT_SOURCE_DIR}/ATLAS_HECTB
    ${PROJECT_SOURCE_DIR}/../../examples/common/include
    ${HEPMC3_INCLUDE_DIR}
)
cuda_rdc_target_link_libraries(ATLAS_HECTB
  PRIVATE
    ${HEPMC3_LIBRARIES}
    ${HEPMC3_FIO_LIBRARIES}
    AdePT_G4_integration
)

# Install macros and geometry file

configure_file("macros/ATLHECTB_gui.mac.in" "${PROJECT_BINARY_DIR}/HECTB/ATLHECTB_gui.mac")
configure_file("macros/ATLHECTB_init_vis.mac.in" "${PROJECT_BINARY_DIR}/HECTB/ATLHECTB_init_vis.mac")
configure_file("macros/ATLHECTB_run.mac.in" "${PROJECT_BINARY_DIR}/HECTB/ATLHECTB_run.mac")

# Tests

add_test(NAME ATLAS_HECTB-g4vg
  COMMAND $<TARGET_FILE:ATLAS_HECTB> -m ${PROJECT_BINARY_DIR}/ATLHECTB_run.mac
)
