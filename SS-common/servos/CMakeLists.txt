cmake_minimum_required(VERSION 3.11.1)

project(servos C)

add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(
        ${PROJECT_NAME} INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)

target_sources(${PROJECT_NAME} INTERFACE
        ${CMAKE_CURRENT_SOURCE_DIR}/SS_servos.c
        ${CMAKE_CURRENT_SOURCE_DIR}/SS_servos_test.c)
