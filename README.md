Build mongrel2-cpp (m2pp) in CMake as an external project
=========================================================

To use this repository to build mongrel2-cpp into a CMake project, add this as a
submodule:

    git submodule add https://github.com/cpick/mongrel2-cpp-cmake.git

Add it as a subdirctory in the project's CMakeLists.txt and then link against
${M2PP_LIBRARIES}:

    add_submodule(mongrel2-cpp-cmake)
    ...
    target_link_libraries(executable ${M2PP_LIBRARIES})
