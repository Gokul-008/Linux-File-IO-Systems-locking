# Linux-File-IO-Systems-locking
Ex07-Linux File-IO Systems-locking
# AIM:
To Write a C program that illustrates files copying and locking

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux IO Systems locking

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## 1.To Write a C program that illustrates files copying 

```
/*
 * file_copy.c
 *
 * This program demonstrates copying a file using low-level Linux/POSIX
 * system calls (open, read, write, close).
 *
 * Usage:
 * ./file_copy <source_file> <destination_file>
 *
 * Example:
 * 1. Create a source file: echo "Hello world" > source.txt
 * 2. Compile the program:  gcc file_copy.c -o file_copy
 * 3. Run the program:       ./file_copy source.txt destination.txt
 * 4. Check the result:     cat destination.txt
 */

#include <stdio.h>      // For perror, printf
#include <stdlib.h>     // For exit, EXIT_FAILURE, EXIT_SUCCESS
#include <fcntl.h>      // For open() and file flags (O_RDONLY, O_WRONLY, etc.)
#include <unistd.h>     // For read(), write(), close()
#include <sys/stat.h>   // For mode bits (S_IRUSR, S_IWUSR)

#define BUFFER_SIZE 4096 // 4KB buffer

int main(int argc, char *argv[]) {
    int fd_src, fd_dest;
    ssize_t bytes_read, bytes_written;
    char buffer[BUFFER_SIZE];

    // 1. Check for correct number of command-line arguments
    if (argc != 3) {
        printf("Usage: %s <source_file> <destination_file>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    // 2. Open the source file for reading
    fd_src = open(argv[1], O_RDONLY);
    if (fd_src == -1) {
        perror("Error opening source file");
        exit(EXIT_FAILURE);
    }

    // 3. Open the destination file for writing.
    //    - O_CREAT: Create the file if it doesn't exist.
    //    - O_WRONLY: Open for writing only.
    //    - O_TRUNC: Truncate (empty) the file if it already exists.
    //    - Permissions (0644):
    //      - S_IRUSR | S_IWUSR: Owner can read and write
    //      - S_IRGRP: Group can read
    //      - S_IROTH: Others can read
    fd_dest = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
    if (fd_dest == -1) {
        perror("Error opening/creating destination file");
        close(fd_src); // Clean up the already-opened source file
        exit(EXIT_FAILURE);
    }

    // 4. Read from source and write to destination
    printf("Copying file '%s' to '%s'...\n", argv[1], argv[2]);

    // Loop as long as read() returns more than 0 bytes
    while ((bytes_read = read(fd_src, buffer, BUFFER_SIZE)) > 0) {
        // Write the exact number of bytes that were just read
        bytes_written = write(fd_dest, buffer, bytes_read);

        // Check for write errors or partial writes
        if (bytes_written != bytes_read) {
            perror("Error writing to destination file");
            close(fd_src);
            close(fd_dest);
            exit(EXIT_FAILURE);
        }
    }

    // 5. Check for a read error (read returns -1)
    if (bytes_read == -1) {
        perror("Error reading from source file");
        close(fd_src);
        close(fd_dest);
        exit(EXIT_FAILURE);
    }

    // 6. Close both file descriptors
    if (close(fd_src) == -1) {
        perror("Error closing source file");
        // Continue to close dest, but will exit with failure
    }
    if (close(fd_dest) == -1) {
        perror("Error closing destination file");
    }

    // If we get here, the loop finished (bytes_read == 0) and no errors occurred
    printf("File copy successful!\n");
    return EXIT_SUCCESS;
}
```


## OUTPUT

1. -rwxr-xr-x    1 root     root         18348 Apr 17 14:14 file.o
2. -rwxr-xr-x    1 root     root         18376 Apr 17 14:20 text.o



# RESULT:
The programs are executed successfully.
