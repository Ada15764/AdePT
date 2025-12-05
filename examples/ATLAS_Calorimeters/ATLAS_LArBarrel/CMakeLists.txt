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
  src/ATLLArBarrelActIni.cc
  src/ATLLArBarrelConstruction.cc
  src/ATLLArBarrelEventAction.cc
  src/ATLLArBarrelHit.cc
  src/ATLLArBarrelPriGenAct.cc
  src/ATLLArBarrelRunAction.cc
  src/ATLLArBarrelSensDet.cc 
  src/ATLLArBarrelTBConstants.cc
  src/ATLLArBarrelTrackingAction.cc
  src/LArBarrelConstruction.cc
  src/LArBarrelPresamplerConstruction.cc
  src/LArTBCryostatConstruction.cc
)

if(HepMC3_FOUND)
  set(sources_hepmc3
    ${PROJECT_SOURCE_DIR}/../../examples/common/src/HepMC3G4AsciiReader.cc
    ${PROJECT_SOURCE_DIR}/../../examples/common/src/HepMC3G4AsciiReaderMessenger.cc
    ${PROJECT_SOURCE_DIR}/../../examples/common/src/HepMC3G4Interface.cc
  )
endif()

# example1
add_executable(ATLAS_LArBarrel ATLAS_LArBarrel.cpp ${sources_g4} ${sources_hepmc3})

target_include_directories(ATLAS_LArBarrel
  PRIVATE
  ${PROJECT_SOURCE_DIR}/ATLAS_LArBarrel/include
  ${PROJECT_SOURCE_DIR}/ATLAS_LArBarrel
    ${PROJECT_SOURCE_DIR}/../../examples/common/include
    ${HEPMC3_INCLUDE_DIR}
)
cuda_rdc_target_link_libraries(ATLAS_LArBarrel
  PRIVATE
    ${HEPMC3_LIBRARIES}
    ${HEPMC3_FIO_LIBRARIES}
    AdePT_G4_integration
)

# Install macros and geometry file
configure_file("macros/gui.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/gui.mac")
configure_file("macros/icons.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/icons.mac")
configure_file("macros/init.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/init.mac")
configure_file("macros/init_vis.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/init_vis.mac")
configure_file("macros/run.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/run.mac")
configure_file("macros/single.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/single.mac")
configure_file("macros/vis.mac.in" "${PROJECT_BINARY_DIR}/LArBarrel/vis.mac")

# Tests

add_test(NAME ATLAS_LArBarrel-g4vg
  COMMAND $<TARGET_FILE:ATLAS_LArBarrel> -m ${PROJECT_BINARY_DIR}/example1.mac
)
