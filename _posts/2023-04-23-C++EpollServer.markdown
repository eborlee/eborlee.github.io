---
layout:     post
title:      "C++ Multi-Reactor Server Engine"
subtitle:   " \"Linux, Epoll, Threading Pool\""
date:       2024-06-01 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
    - Development
---


Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents


* TOC
{:toc}




# Github Repo Link
[https://github.com/eborlee/ReactorNet](https://github.com/eborlee/ReactorNet)

# ReactorNet
ReactorNet is a high-performance server application designed to handle multiple client connections efficiently. It is built using modern C++11 and leverages various I/O multiplexing mechanisms such as epoll, poll, and select on Linux. The server is optimized for concurrent connections using a thread pool, ensuring robust and scalable performance.


# Introduction
MyServer is a high-performance server application designed to handle multiple client connections efficiently. It is built using modern C++ and leverages epoll for scalable I/O event notification.

# Features
- I/O Multiplexing: Supports epoll, poll, and select to efficiently manage multiple simultaneous client connections.
- Modern C++11: Utilizes features of C++11 for improved performance, maintainability, and readability.
- Thread Pool: Employs a thread pool to handle concurrent connections, reducing the overhead of thread creation and destruction, and improving response time.
- Modular Design: Designed with modularity in mind, making it easy to extend and customize for different use cases.
- High Scalability: Capable of handling a large number of connections simultaneously, making it suitable for high-traffic environments.
# Installation
## Prerequisites
- C++11 or higher
- GNU Make
- g++ compiler
## Building the Project
1. Clone the repository:
```shell
git clone https://github.com/eborlee/ReactorNet.git
cd ReactorNet
```
2. Build the project using Makefile:
```shell
make
```
The executable will be generated in the build directory.
# Usage
1. Run the server:
```shell
./build/ReactorNet
```
2. The server will start and listen for incoming connections.


