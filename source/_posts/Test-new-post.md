---
title: Test-new-post
date: 2016-06-22 13:54:02
comments: true
tags:
---
# Dependency Analyser

## Overview
This project is for the first three project of SU_CIS687 Object Oriented Design course. It is written in C++ and can be used to analysis c++ source code, generate the abstract syntax tree of the code and get the dependency between each files.

In this project, it includes codes about:
* Design patterns(Factory, Strategy, etc..) and
* Some C++ 11 features
* File manager function
* Thread pool implementation(based on lambda)
* Tokenizer and Semi-expression to analysis c++ source code
* Parser to build abstract syntax tree
* Dependency analyser based on abstract syntax tree


## Usage:

**To compile it:**
Run complie.bat(for windows).

**To run(three executable parts):**
1. Search file: FileSearch_Test.exe <root_path> <search_patterns>. Example: FileSearch_Test.exe .. *.cpp *.h
2. Get abstract syntax tree for some files under a specified root folder: MetricsExecutive.exe <root_path> <search_patterns>
3. Analysis dependency between files:
    ParallelDependencyExecutive.exe <root_path> <search_patterns>. Example: ParallelDependencyExecutive.exe .. *.cpp *.h
    
