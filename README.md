# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.

### shmry1:
```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;
    int shmid;

    // Create shared memory segment
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    printf("Shared memory ID = %d\n", shmid);
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    // Attach shared memory
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    // Print the address where memory is attached
    printf("Memory attached at %p\n", (void*)shared_memory);

    // Initialize the shared structure
    shared_stuff = (struct shared_use_st *)shared_memory;

    // Start the main loop to interact with shared memory
    while (running) {
        // Wait for the client to write data
        while (shared_stuff->written_by_you == 0) {
            // If no data is written, keep waiting
            printf("Waiting for client...\n");
            sleep(1);  // Wait for a while to avoid tight looping
        }

        // Print the entered text from shared memory
        printf("You Wrote: %s\n", shared_stuff->some_text);
        shared_stuff->written_by_you = 0;  // Reset flag after reading

        // Prompt user to input some text
        printf("Enter some text: ");
        char buffer[TEXT_SZ];
        fgets(buffer, TEXT_SZ, stdin);

        // Remove newline character if it's present
        buffer[strcspn(buffer, "\n")] = 0;

        // Copy the user input into shared memory and mark it as written
        strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
        shared_stuff->some_text[TEXT_SZ - 1] = '\0';  // Ensure null-termination
        shared_stuff->written_by_you = 1;

        // Check if the user entered "end" to terminate
        if (strncmp(buffer, "end", 3) == 0) {
            running = 0;
        }
    }

    // Detach from shared memory
    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    // Cleanup and delete the shared memory
    if (shmctl(shmid, IPC_RMID, 0) == -1) {
        fprintf(stderr, "failed to delete shared memory\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}
```
### shmry2:
```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;
    int shmid;

    srand((unsigned int)getpid());

    // Create the shared memory segment
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    printf("Shared memory id is %d \n", shmid);
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    // Attach to the shared memory segment
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    // Print the address of the shared memory (pointer)
    printf("Memory Attached at %p\n", (void*)shared_memory); // Use %p to print pointer address

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written_by_you = 0;

    while (running) {
        // Wait until the reader has read the previous message
        if (shared_stuff->written_by_you == 0) {
            // Write data to shared memory
            printf("You Wrote: ");
            fgets(shared_stuff->some_text, TEXT_SZ, stdin);

            // Mark the memory as written by you
            shared_stuff->written_by_you = 1;

            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                running = 0;

            }
        }
    }

    // Detach from the shared memory
    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    // Remove the shared memory segment
    if (shmctl(shmid, IPC_RMID, 0) == -1) {
        fprintf(stderr, "failed to delete\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}
```
## OUTPUT
<img width="735" height="230" alt="Screenshot from 2025-10-04 08-21-15" src="https://github.com/user-attachments/assets/ad8da884-c804-4a8b-9aa7-953dd263e487" />

<img width="735" height="94" alt="Screenshot from 2025-10-04 08-47-47" src="https://github.com/user-attachments/assets/7d069439-1d91-48cb-ab57-85c555c8f4ef" />

<img width="735" height="609" alt="Screenshot from 2025-10-04 08-47-26" src="https://github.com/user-attachments/assets/a96fec2e-a793-47b3-9dc5-763f855617bb" />

# RESULT:
The program is executed successfully.
