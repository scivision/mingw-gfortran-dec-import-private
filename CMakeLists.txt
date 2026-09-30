cmake_minimum_required(VERSION 3.25)

project(dllimport_private LANGUAGES Fortran)

# Is a Fortran module variable allowed to carry a DLLIMPORT attribute when it has
# PRIVATE accessibility?
#
# GNU Fortran 16.2.0 (x86_64-w64-mingw32) rejects it:
#   Error: 'dllimport' implies default visibility, but 'x' has already been
#   declared with a different visibility
# The identical declaration is accepted when the variable is PUBLIC, so the trigger
# is DLLIMPORT combined with private accessibility, nothing else.
#
# Real-world impact: MSYS2 mingw-w64-msmpi 10.1.1-22 added
# "!GCC$ ATTRIBUTES DLLIMPORT" for the MPI sentinel variables to mpif.h. Any module
# that does INCLUDE 'mpif.h' and has a default PRIVATE statement then fails to
# compile. That is a standard-conforming and widespread idiom; it breaks MUMPS.
# This project needs no MPI to show the problem.

set(cases
  gcc_public
  gcc_private
  gcc_private_explicit
  gcc_private_public
  dec_public
  dec_private
)

# these modules contain no main program, so only compile them
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

message(STATUS "\n${CMAKE_Fortran_COMPILER_ID} ${CMAKE_Fortran_COMPILER_VERSION} (${CMAKE_Fortran_COMPILER})")

foreach(c IN LISTS cases)
  message(CHECK_START "${c}")

  try_compile(ok
    SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/${c}.f
    OUTPUT_VARIABLE out
  )

  if(ok)
    message(CHECK_PASS "compiles")
  else()
    message(CHECK_FAIL "REJECTED")
    string(REGEX MATCHALL "[Ee]rror[^\n]*" err "${out}")
    list(REMOVE_DUPLICATES err)
    list(JOIN err "\n      " err)
    if(err)
      message(STATUS "${c}:\n      ${err}")
    endif()
  endif()
endforeach()

file(GENERATE OUTPUT .gitignore CONTENT "*")
