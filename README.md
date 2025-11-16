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


#include <stdio.h>      
#include <stdlib.h>     
#include <fcntl.h>     
#include <unistd.h>     
#include <sys/stat.h>   



int main(int argc, char *argv[]) {
    int fd_src, fd_dest;
    ssize_t bytes_read, bytes_written;
    char buffer[BUFFER_SIZE];

   
    if (argc != 3) {
        printf("Usage: %s <source_file> <destination_file>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

   
    fd_src = open(argv[1], O_RDONLY);
    if (fd_src == -1) {
        perror("Error opening source file");
        exit(EXIT_FAILURE);
    }

  
    fd_dest = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
    if (fd_dest == -1) {
        perror("Error opening/creating destination file");
        close(fd_src);
        exit(EXIT_FAILURE);
    }

  
    printf("Copying file '%s' to '%s'...\n", argv[1], argv[2]);

  
    while ((bytes_read = read(fd_src, buffer, BUFFER_SIZE)) > 0) {
     
        bytes_written = write(fd_dest, buffer, bytes_read);

      
        if (bytes_written != bytes_read) {
            perror("Error writing to destination file");
            close(fd_src);
            close(fd_dest);
            exit(EXIT_FAILURE);
        }
    }

 
    if (bytes_read == -1) {
        perror("Error reading from source file");
        close(fd_src);
        close(fd_dest);
        exit(EXIT_FAILURE);
    }

   
    if (close(fd_src) == -1) {
        perror("Error closing source file");
        
    }
    if (close(fd_dest) == -1) {
        perror("Error closing destination file");
    }

   
    printf("File copy successful!\n");
    return EXIT_SUCCESS;
}
```
## 2.To Write a C program that illustrates files locking
```


#include <stdio.h>      
#include <stdlib.h>    
#include <fcntl.h>      
#include <unistd.h>    

int main(int argc, char *argv[]) {
    int fd;
    struct flock lock;

    if (argc != 2) {
        printf("Usage: %s <file_to_lock>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

   
    fd = open(argv[1], O_RDWR | O_CREAT, 0666); 
    if (fd == -1) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }


    puts("Attempting to acquire exclusive lock...");

   
    lock.l_type = F_WRLCK;      
    lock.l_whence = SEEK_SET;   
    lock.l_start = 0;           
    lock.l_len = 0;            
    lock.l_pid = getpid();     
  

    if (fcntl(fd, F_SETLKW, &lock) == -1) {
        perror("fcntl (F_SETLKW)");
        close(fd);
        exit(EXIT_FAILURE);
    }

    printf("Lock acquired! (PID: %d) Holding for 10 seconds...\n", getpid());
    printf("You can now run this program from another terminal to see it wait.\n");

   
    sleep(10);

    

    puts("Releasing lock...");

    
    lock.l_type = F_UNLCK;

    if (fcntl(fd, F_SETLKW, &lock) == -1) {
        perror("fcntl (F_UNLCK)");
        close(fd);
        exit(EXIT_FAILURE);
    }

    puts("Lock released.");

  
    close(fd);

    return EXIT_SUCCESS;
}
```


## OUTPUT

<img width="812" height="412" alt="Screenshot from 2025-11-16 15-13-54" src="https://github.com/user-attachments/assets/968defe8-8cbc-454d-8f7e-f75065f960cd" />

<img width="994" height="727" alt="Screenshot from 2025-11-16 15-20-07" src="https://github.com/user-attachments/assets/bf6151ad-1dea-4ee6-98ce-a6861a4cb2cf" />


# RESULT:
The programs are executed successfully.
