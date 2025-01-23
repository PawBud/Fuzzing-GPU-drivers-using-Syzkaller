# Fuzzing GPU drivers using Syzkaller
Fuzzing provides a target computer program with a bunch of random or unexpected inputs to 
see if it crashes or behaves abnormally. [Syzkaller](https://github.com/google/syzkaller) 
is a popular kernel fuzzer that uses  input programs written in a declarative language 
and dynamically tests target kernel modules to find bugs.

## Aim of this project
This repository aims to create a blueprint for users who want to fuzz graphics 
drivers on Linux.

## Fuzzing
To illustrate fuzzing, let’s start with an example. 
The following program illustrates a C program where one specific input to the 
function introduces a potential memory leak vulnerability, depending on the input size.

```C
void process_input(const char *input) {
    if (strcmp(input, "safe1") == 0) {
        // Path 1: Safe path
        char buffer[20]; // Sufficiently large buffer
        strcpy(buffer, "This is safe");
        printf("Safe1 buffer content: %s", buffer);

    } else if (strcmp(input, "safe2") == 0) {
        // Path 2: Safe path
        char buffer[20]; // Sufficiently large buffer
        snprintf(buffer, sizeof(buffer), "Safe2 input");
        printf("Safe2 buffer content: %s", buffer);
    } 
    else {
        // Path 3: Potentially unsafe path
        char *buffer = (char *)malloc(30);
        if (buffer == NULL) {
            printf("Memory allocation failed");
            return;
        }

        strcpy(buffer, "Memory might be leaked");
        printf("Potentially leaked buffer content: %s", buffer);

        // Potential memory leak: Memory might not be freed
        if (strlen(input) > 10) {
            // Free memory conditionally
            free(buffer);
        }
    }
    
    printf("it worked :)");
}
```

The program evaluates input to determine the execution path: if input is "safe1" or "safe2", 
it performs safe operations with stack-allocated buffers. 
For any other input, the program allocates and utilizes memory using malloc. 
However, memory is only conditionally freed if the length of the input exceeds 10 characters,
which may result in a memory leak for inputs shorter than this threshold. 
Each unique input results in following a unique execution path which contributes to _coverage_.


In the context of kernel fuzzing, applications in userland provide the input to the target. 
Userland applications interact with the kernel through system calls and interfaces provided 
by the kernel. For example, when a user application opens a file, sends a network packet, 
or allocates memory, these operations go through system calls that invoke kernel code to handle 
the requested action. Identifying such issues through static analysis can be challenging due to 
the complexity and the sheer size of kernel code. However, fuzzing dynamically tests various 
inputs by generating and executing them, which increases the likelihood of uncovering such vulnerabilities. 
A fuzzer would repeatedly run the program with different inputs (7), improving the chances of
detecting the memory leak. The set of inputs is referred to as corpus. Fuzzing is usually done in a 
VM as it provides reproducibility, environment management, and isolation.

## Understanding the target
![DRI Stack](media/DRI%20Overview%20Master's%20Thesis.png)

The figure shows [DRI](https://dri.freedesktop.org/wiki/) (Direct Rendering Infrastructure) on Linux, which is a software architecture 
designed to coordinate interactions between the Linux kernel, the X Window System, 3D graphics hardware, 
and an OpenGL-based rendering engine, enabling efficient 3D rendering on Linux platforms.

It is essential to understand the Graphics stack in order to succesfully fuzz the GPU drivers. This helps
an analyst to understand the kind of data that should be passed to the Graphics Driver.

We aimed to fuzz the **proprietary Nvidia graphics driver** \& the **Nouveau Drivers** on Linux.

## Our Experimental Setup
![Environment Setup](media/System%20Specs.png)

The host system ran Syzkaller alongside QEMU in userland, with KVM operating in kernel land. 
The QEMU virtual machine (VM) was launched dynamically based on Syzkaller’s configuration file.
The fuzzing process inside the VM required direct access 
to the Nvidia GPU, which was facilitated through IOMMU and the VFIO driver. 
KVM enabled the guest operating system, running inside the VM, to execute instructions directly 
on the physical CPU. Additionally, it allocated memory to the guest OS via standard malloc() and 
mmap() calls.


### About this repository
This repository contains the supplementary information related to my Master’s Thesis, 
which I completed under the supervision of [**Andrés Goens**](https://goens.org/) with 
the [Parallel Computing Systems](https://pcs-research.nl/) Group at the
[University of Amsterdam](https://www.uva.nl/).

