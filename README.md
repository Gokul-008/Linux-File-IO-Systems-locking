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
```
/*
 * file_lock.c
 *
 * This program demonstrates how to acquire an exclusive (write) lock
 * on a file using fcntl(). The program locks the file, "holds" the
 * lock for 10 seconds, and then releases it.
 *
 * Usage:
 * ./file_lock <file_to_lock>
 *
 * How to Test:
 * 1. Compile the program:
 * gcc file_lock.c -o file_lock
 *
 * 2. Open two separate terminal windows (Terminal 1 and Terminal 2).
 *
 * 3. In Terminal 1, run the program:
 * ./file_lock test.lockfile
 *
 * You will see: "Attempting to acquire exclusive lock..."
 * "Lock acquired! Holding for 10 seconds..."
 *
 * 4. IMMEDIATELY (within 10 seconds), in Terminal 2, run the *same* command:
 * ./file_lock test.lockfile
 *
 * You will see: "Attempting to acquire exclusive lock..."
 * This terminal will now *wait* and appear to be frozen.
 *
 * 5. Wait for Terminal 1 to finish. After 10 seconds, it will print:
 * "Releasing lock..."
 * "Lock released."
 *
 * 6. As soon as Terminal 1 releases the lock, Terminal 2 will
 * immediately acquire it and print:
 * "Lock acquired! Holding for 10 seconds..."
 * ...and then it will release it after its own 10 seconds.
 */

#include <stdio.h>      // For perror, printf, puts
#include <stdlib.h>     // For exit, EXIT_FAILURE, EXIT_SUCCESS
#include <fcntl.h>      // For open() and fcntl()
#include <unistd.h>     // For close(), sleep(), getpid()

int main(int argc, char *argv[]) {
    int fd;
    struct flock lock;

    if (argc != 2) {
        printf("Usage: %s <file_to_lock>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    // Open the file for reading and writing.
    // O_CREAT will create it if it doesn't exist.
    fd = open(argv[1], O_RDWR | O_CREAT, 0666); // 0666 for easy permissions
    if (fd == -1) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }

    // --- Acquire the Lock ---

    puts("Attempting to acquire exclusive lock...");

    // Initialize the flock structure for locking
    lock.l_type = F_WRLCK;      // Exclusive Write Lock
    lock.l_whence = SEEK_SET;   // Lock from the start of the file
    lock.l_start = 0;           // Starting offset (0)
    lock.l_len = 0;             // Length (0 means lock the entire file)
    lock.l_pid = getpid();      // (Not used by F_SETLKW, but good practice)

    // F_SETLKW: Set Lock and Wait
    // This call will BLOCK (wait) if the lock is already held by
    // another process. It will only return once it acquires the lock
    // or if an error occurs.
    if (fcntl(fd, F_SETLKW, &lock) == -1) {
        perror("fcntl (F_SETLKW)");
        close(fd);
        exit(EXIT_FAILURE);
    }

    printf("Lock acquired! (PID: %d) Holding for 10 seconds...\n", getpid());
    printf("You can now run this program from another terminal to see it wait.\n");

    // Simulate doing "work" while holding the lock
    sleep(10);

    // --- Release the Lock ---

    puts("Releasing lock...");

    // To release the lock, set the type to F_UNLCK
    // and call fcntl again with the same lock region.
    lock.l_type = F_UNLCK;

    if (fcntl(fd, F_SETLKW, &lock) == -1) {
        perror("fcntl (F_UNLCK)");
        close(fd);
        exit(EXIT_FAILURE);
    }

    puts("Lock released.");

    // Close the file
    close(fd);

    return EXIT_SUCCESS;
}
```


## OUTPUT

1. -rwxr-xr-x    1 root     root         18348 Apr 17 14:14 file.o
2. -rwxr-xr-x    1 root     root         18376 Apr 17 14:20 text.o



# RESULT:
The programs are executed successfully.
