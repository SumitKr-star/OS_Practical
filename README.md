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

## 8.FIFO
```
#include <iomanip>
#include <iostream>

int main() {
    int n;
    int bt[20], wt[20], tat[20];
    double avg_wt = 0.0, avg_tat = 0.0;

    std::cout << "Enter number of processes: ";
    std::cin >> n;

    for (int i = 0; i < n; i++) {
        std::cout << "Enter burst time P" << (i + 1) << ": ";
        std::cin >> bt[i];
    }

    wt[0] = 0;

    for (int i = 1; i < n; i++) {
        wt[i] = wt[i - 1] + bt[i - 1];
    }

    std::cout << "\nProcess\tBT\tWT\tTAT\n";

    for (int i = 0; i < n; i++) {
        tat[i] = wt[i] + bt[i];

        avg_wt += wt[i];
        avg_tat += tat[i];

        std::cout << "P" << (i + 1) << "\t"
                  << bt[i] << "\t"
                  << wt[i] << "\t"
                  << tat[i] << "\n";
    }

    std::cout << std::fixed << std::setprecision(2);
    std::cout << "\nAverage Waiting Time: " << (avg_wt / n);
    std::cout << "\nAverage Turnaround Time: " << (avg_tat / n) << "\n";

    return 0;
}
```
### SJF
```
#include <iomanip>
#include <iostream>

int main() {
    int n;
    int pid[20], bt[20], wt[20], tat[20];
    double avg_wt = 0.0, avg_tat = 0.0;

    std::cout << "Enter number of processes: ";
    std::cin >> n;

    for (int i = 0; i < n; i++) {
        pid[i] = i + 1;
        std::cout << "Enter burst time P" << pid[i] << ": ";
        std::cin >> bt[i];
    }

    // Sort processes by burst time (shortest job first)
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (bt[j] > bt[j + 1]) {
                int temp_bt = bt[j];
                bt[j] = bt[j + 1];
                bt[j + 1] = temp_bt;

                int temp_pid = pid[j];
                pid[j] = pid[j + 1];
                pid[j + 1] = temp_pid;
            }
        }
    }

    wt[0] = 0;
    for (int i = 1; i < n; i++) {
        wt[i] = wt[i - 1] + bt[i - 1];
    }

    std::cout << "\nProcess\tBT\tWT\tTAT\n";

    for (int i = 0; i < n; i++) {
        tat[i] = wt[i] + bt[i];
        avg_wt += wt[i];
        avg_tat += tat[i];

        std::cout << "P" << pid[i] << "\t"
                  << bt[i] << "\t"
                  << wt[i] << "\t"
                  << tat[i] << "\n";
    }

    std::cout << std::fixed << std::setprecision(2);
    std::cout << "\nAverage Waiting Time: " << (avg_wt / n);
    std::cout << "\nAverage Turnaround Time: " << (avg_tat / n) << "\n";

    return 0;
}
```

### SRTF
```
#include <climits>
#include <iomanip>
#include <iostream>

int main() {
    int n;
    int at[20], bt[20], rt[20], wt[20], tat[20];
    double avg_wt = 0.0, avg_tat = 0.0;

    std::cout << "Enter number of processes: ";
    std::cin >> n;

    for (int i = 0; i < n; i++) {
        std::cout << "Enter arrival time P" << (i + 1) << ": ";
        std::cin >> at[i];
        std::cout << "Enter burst time P" << (i + 1) << ": ";
        std::cin >> bt[i];
        rt[i] = bt[i];
    }

    int completed = 0;
    int time = 0;

    while (completed < n) {
        int shortest = -1;
        int min_rt = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (at[i] <= time && rt[i] > 0 && rt[i] < min_rt) {
                min_rt = rt[i];
                shortest = i;
            }
        }

        if (shortest == -1) {
            time++;
            continue;
        }

        rt[shortest]--;
        time++;

        if (rt[shortest] == 0) {
            completed++;
            int finish_time = time;
            tat[shortest] = finish_time - at[shortest];
            wt[shortest] = tat[shortest] - bt[shortest];
            if (wt[shortest] < 0) {
                wt[shortest] = 0;
            }

            avg_wt += wt[shortest];
            avg_tat += tat[shortest];
        }
    }

    std::cout << "\nProcess\tAT\tBT\tWT\tTAT\n";

    for (int i = 0; i < n; i++) {
        std::cout << "P" << (i + 1) << "\t"
                  << at[i] << "\t"
                  << bt[i] << "\t"
                  << wt[i] << "\t"
                  << tat[i] << "\n";
    }

    std::cout << std::fixed << std::setprecision(2);
    std::cout << "\nAverage Waiting Time: " << (avg_wt / n);
    std::cout << "\nAverage Turnaround Time: " << (avg_tat / n) << "\n";

    return 0;
}
```


## 9. Sum of Numbers Using Pthreads
```
#include <iostream>
#include <thread>

int a[100];
int sum1 = 0, sum2 = 0;
int n = 0;

void first_thread() {
    for (int i = 0; i < n / 2; i++) {
        sum1 += a[i];
    }
}

void second_thread() {
    for (int i = n / 2; i < n; i++) {
        sum2 += a[i];
    }
}

int main() {
    std::cout << "Enter even number of elements: ";
    std::cin >> n;

    if (n <= 0 || n > 100 || n % 2 != 0) {
        std::cout << "Enter an even number between 2 and 100\n";
        return 1;
    }

    std::cout << "Enter numbers:\n";
    for (int i = 0; i < n; i++) {
        std::cin >> a[i];
    }

    std::thread t1(first_thread);
    std::thread t2(second_thread);

    t1.join();
    t2.join();

    std::cout << "First Thread Sum: " << sum1 << "\n";
    std::cout << "Second Thread Sum: " << sum2 << "\n";
    std::cout << "Total Sum: " << (sum1 + sum2) << "\n";

    return 0;
}
```

## 10. Memory Allocation Strategies
```
#include <iostream>

int main() {
    int b[20], p[20];
    int nb, np;

    std::cout << "Enter number of memory blocks: ";
    std::cin >> nb;

    std::cout << "Enter block sizes:\n";
    for (int i = 0; i < nb; i++) {
        std::cin >> b[i];
    }

    std::cout << "Enter number of processes: ";
    std::cin >> np;

    std::cout << "Enter process sizes:\n";
    for (int i = 0; i < np; i++) {
        std::cin >> p[i];
    }

    int choice;
    std::cout << "\n1. First Fit";
    std::cout << "\n2. Best Fit";
    std::cout << "\n3. Worst Fit";
    std::cout << "\nEnter choice: ";
    std::cin >> choice;

    int temp[20], allocated[20];

    for (int i = 0; i < nb; i++) {
        temp[i] = b[i];
    }

    for (int i = 0; i < np; i++) {
        allocated[i] = -1;
    }

    for (int i = 0; i < np; i++) {
        int index = -1;

        for (int j = 0; j < nb; j++) {
            if (temp[j] >= p[i]) {
                if (choice == 1) {
                    index = j;
                    break;
                }
                if (choice == 2) {
                    if (index == -1 || temp[j] < temp[index]) {
                        index = j;
                    }
                }
                if (choice == 3) {
                    if (index == -1 || temp[j] > temp[index]) {
                        index = j;
                    }
                }
            }
        }

        if (index != -1) {
            allocated[i] = index;
            temp[index] -= p[i];
        }
    }

    std::cout << "\nProcess\tSize\tBlock\n";
    for (int i = 0; i < np; i++) {
        std::cout << "P" << (i + 1) << "\t" << p[i] << "\t";
        if (allocated[i] == -1) {
            std::cout << "Not Allocated\n";
        } else {
            std::cout << "B" << (allocated[i] + 1) << "\n";
        }
    }

    return 0;
}
```



