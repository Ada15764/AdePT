# SPDX-FileCopyrightText: 2025 CERN
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.26)
project(ATLAS_calorimeters)

# - Subprojects
add_subdirectory(HECTB)
add_subdirectory(TileCalTB)
add_subdirectory(LArBarrel)

