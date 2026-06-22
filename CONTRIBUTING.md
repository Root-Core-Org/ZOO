# Contributing to Zoo

We are excited that you want to contribute your applications to the Zoo binary ecosystem for Root Core OS! To maintain the integrity and cleanliness of the repository, we follow a strict submission workflow.

## Submission Process

We **only** accept contributions through GitHub Issues under the following rules:

1. **Create an Issue:** Open a new issue in this repository.
2. **Apply the Correct Label:** You **must** select and apply the `New App` label to your issue.
   * *Critical Note:* Issues submitted without the `New App` label will be automatically closed, deleted, or rejected.
3. **Describe Your Application:** In the issue description, write a short summary detailing:
   * What the application does.
   * Why it is needed or how it benefits Root Core OS.
   * A brief overview of its features.
4. **Provide a Permanent Link:** At the bottom of your description, you must provide a permanent, open-source link (e.g., your personal GitHub repository or release page) where you will host and publish future updates for your application.

---

## How to Test and Compile Your Applications

### Full Instruction:

#### 1. ZOO Applications
If you are developing a standard standalone ZOO application:
* Analyze the kernel calls, the **CROCI command standard**, and other existing code containing various system interfaces. Integrate these into your program to handle system operations. Your application will interact directly via syscalls.
* Once your C code is ready, trigger the compilation using the provided build tool:
```bash
  ./mkzoo

```

* To understand which system calls are available and required to compile your application properly, inspect the contents of the `mkzoo` script inside the `tools` directory. There are many calls available, and explaining them individually here is highly complex.

#### 2. Libraries for ZOO Applications

If you are maintaining multiple applications and want to avoid duplicating shared routines across your codebases, you can offload those calls into a separate binary and distribute it as a `.lib` file along with your ZOO binaries:

* Write your C code containing the functions that will be invoked by the main ZOO binaries.
* Compile your binaries as dynamic libraries, or static libraries if the codebase needs to be baked directly into the application during compilation.
* The compilation process is identical to a standard ZOO application, but utilizes the `mklib` tool:
```bash
./mklib

```


* Detailed configuration and implementation guidelines can be found inside `tools/mklib.c`.

#### 3. Drivers

If you want to add hardware or subsystem support (for example, implementing `i2c` bus support inside Root Core), editing the core kernel files directly is highly discouraged. Instead, you should create a **Shared Driver File (SDF)**:

* Write C code defining the structural instructions for the kernel and application layers (i.e., your driver's exported API routines, such as `i2c_init()`, etc.).
* Compile your driver using the `mksdf` tool:
```bash
./mksdf

```


* Detailed compilation guidelines are located inside `tools/mksdf.c`.

---

### What is the difference between a Library and a Driver?

While libraries and drivers in Root Core OS are almost identical in terms of internal functional implementation, they differ fundamentally in how they interact with the system and binaries:

* **Library (`.lib`):** A library acts as a helper that injects missing code into executable files *only* if those files explicitly request it, and it does so strictly on a per-file isolation level.
* **Driver (`.sdf`):** A driver immediately registers its instructions globally within the operating system. It informs every running binary that its specific system calls are universally available. For example, if a program calls `i2c_init()`, the kernel dynamically intercepts this call, informs the binary that the routine exists, and routes the execution directly to the corresponding `.sdf` driver file.
