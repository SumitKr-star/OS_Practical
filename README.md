# OS Practical
## Same program , Different Code
```
#include <stdio.h>
#include <unistd.h>

int main()
{
    int pid = fork();

    if (pid == 0)
    {
        printf("I am Child Process\n");
    }
    else if (pid > 0)
    {
        printf("I am Parent Process\n");
    }
    else
    {
        printf("Fork failed\n");
    }

    return 0;
}
```

## Parent wait for child process
```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
    int pid = fork();

    if (pid == 0)
    {
        printf("Child process started\n");
        sleep(3);
        printf("Child process finished\n");
    }
    else if (pid > 0)
    {
        wait(NULL);
        printf("Parent process finished\n");
    }

    return 0;
}
```

## 4. Linux Kernel and CPU Information
```

#include <stdio.h>
#include <sys/utsname.h>
#include <string.h>

int main()
{
    struct utsname info;

    if (uname(&info) == -1)
    {
        perror("uname");
        return 1;
    }

    printf("Operating System: %s\n", info.sysname);
    printf("Node Name: %s\n", info.nodename);
    printf("Kernel Release: %s\n", info.release);
    printf("Kernel Version: %s\n", info.version);
    printf("Machine: %s\n", info.machine);

    FILE *fp = fopen("/proc/cpuinfo", "r");

    if (fp == NULL)
    {
        perror("File opening failed");
        return 1;
    }

    char line[256];

    while (fgets(line, sizeof(line), fp))
    {
        if (strncmp(line, "model name", 10) == 0 ||
            strncmp(line, "Hardware", 8) == 0)
        {
            printf("CPU Model: %s", line);
            break;
        }
    }

    fclose(fp);

    return 0;
}
```
```
cat /proc/cpuinfo
```

## 5. Linux Memory Information
```

#include <stdio.h>

int main()
{
    FILE *fp;
    char line[256];

    long total = 0;
    long free_mem = 0;
    long available = 0;

    fp = fopen("/proc/meminfo", "r");

    if (fp == NULL)
    {
        perror("File opening failed");
        return 1;
    }

    while (fgets(line, sizeof(line), fp))
    {
        sscanf(line, "MemTotal: %ld", &total);
        sscanf(line, "MemFree: %ld", &free_mem);
        sscanf(line, "MemAvailable: %ld", &available);
    }

    fclose(fp);

    printf("Total Memory: %ld kB\n", total);
    printf("Free Memory: %ld kB\n", free_mem);
    printf("Available Memory: %ld kB\n", available);
    printf("Used Memory: %ld kB\n", total - free_mem);

    return 0;
}
```

## 6. Copy Files Using System Calls 
```
#include <fstream>
#include <iostream>

int main() {
    std::ifstream source("source.txt", std::ios::binary);
    if (!source.is_open()) {
        std::perror("Source file error");
        return 1;
    }

    std::ofstream destination("destination.txt", std::ios::binary | std::ios::trunc);
    if (!destination.is_open()) {
        std::perror("Destination file error");
        return 1;
    }

    char buffer[100];

    while (source.read(buffer, sizeof(buffer))) {
        destination.write(buffer, source.gcount());
        if (!destination) {
            std::perror("Write error");
            return 1;
        }
    }

    // Write any remaining partial chunk read before EOF
    if (source.gcount() > 0) {
        destination.write(buffer, source.gcount());
        if (!destination) {
            std::perror("Write error");
            return 1;
        }
    }

    if (source.bad()) {
        std::perror("Read error");
        return 1;
    }

    std::cout << "File copied successfully!\n";
    return 0;
}
```
```
g++ copy.cpp -o copy
./copy
```




