cmake
cmakeminimumrequired(VERSION 3.23)
project(CosmicDefenderSDL3 LANGUAGES CXX)

set(CMAKECXXSTANDARD 26)
set(CMAKECXXSTANDARD_REQUIRED ON)
set(CMAKECXXEXTENSIONS OFF)

Find SDL3 and SDL_ttf (install via package manager or vcpkg)
find_package(SDL3 REQUIRED COMPONENTS video render)
findpackage(SDL3ttf REQUIRED)

addexecutable(cosmicdefender src/main.cpp)

targetlinklibraries(cosmic_defender
  PRIVATE SDL3::SDL3 SDL3::SDL3_ttf
)
