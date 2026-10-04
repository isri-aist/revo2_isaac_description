cmake_minimum_required(VERSION 3.10)

# Isaac Sim descriptions of the BrainCo Revo2 hands for mc_isaac (data only)
project(revo2_isaac_description VERSION 0.1.0 LANGUAGES NONE)

option(SRC_MODE "Point the descriptions to the USD files of the source tree instead of the installed ones" OFF)

if(SRC_MODE)
  set(REVO2_ISAAC_USD_DIR "${CMAKE_CURRENT_SOURCE_DIR}/usd")
else()
  set(REVO2_ISAAC_USD_DIR "${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME}/usd")
  install(DIRECTORY usd/ DESTINATION "share/${PROJECT_NAME}/usd")
endif()

# <mc_rtc robot module name> <side> <USD file>
set(REVO2_MODELS
    "revo2_left_hand left revo2_left.usd"
    "revo2_right_hand right revo2_right.usd"
)

# one <module name>.yaml per robot, looked up by mc_isaac in <prefix>/share/mc_isaac
foreach(MODEL IN LISTS REVO2_MODELS)
  separate_arguments(MODEL)
  list(GET MODEL 0 REVO2_MODULE)
  list(GET MODEL 1 SIDE)
  list(GET MODEL 2 USD_FILE)
  set(REVO2_USD "${REVO2_ISAAC_USD_DIR}/${USD_FILE}")
  configure_file(mc_isaac/revo2_hand.in.yaml "${CMAKE_CURRENT_BINARY_DIR}/mc_isaac/${REVO2_MODULE}.yaml" @ONLY)
  install(FILES "${CMAKE_CURRENT_BINARY_DIR}/mc_isaac/${REVO2_MODULE}.yaml" DESTINATION share/mc_isaac)
endforeach()
