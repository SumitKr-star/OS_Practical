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

#include <iostream>
#include <fstream>
#include <string>
#include <cstdio>
#include <sys/utsname.h>

using namespace std;

int main()
{
    struct utsname info;

    // Get operating system information
    if (uname(&info) == -1)
    {
        perror("uname");
        return 1;
    }

    cout << "Operating System: " << info.sysname << "\n";
    cout << "Node Name: " << info.nodename << "\n";
    cout << "Kernel Release: " << info.release << "\n";
    cout << "Kernel Version: " << info.version << "\n";
    cout << "Machine: " << info.machine << "\n";

    // Read CPU information
    ifstream fp("/proc/cpuinfo");

    if (!fp.is_open())
    {
        cout << "File opening failed\n";
        return 1;
    }

    string line;

    while (getline(fp, line))
    {
        if (line.rfind("model name", 0) == 0 ||
            line.rfind("Hardware", 0) == 0)
        {
            cout << "CPU Model: " << line << "\n";
            break;
        }
    }

    fp.close();

    return 0;
}

```
```
cat /proc/cpuinfo
```

## 5. Linux Memory Information
```

#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <cstdio>

using namespace std;

int main()
{
    ifstream fp("/proc/meminfo");

    if (!fp.is_open())
    {
        cout << "File opening failed\n";
        return 1;
    }

    long total = 0;
    long free_mem = 0;
    long available = 0;

    string line;

    // Read memory information
    while (getline(fp, line))
    {
        istringstream iss(line);

        string key;
        long value = 0;

        if (iss >> key >> value)
        {
            if (key == "MemTotal:")
            {
                total = value;
            }
            else if (key == "MemFree:")
            {
                free_mem = value;
            }
            else if (key == "MemAvailable:")
            {
                available = value;
            }
        }
    }

    fp.close();

    // Display memory information
    cout << "\nTotal Memory: " << total << " kB\n";
    cout << "Free Memory: " << free_mem << " kB\n";
    cout << "Available Memory: " << available << " kB\n";
    cout << "Used Memory: " << (total - free_mem) << " kB\n";

    return 0;
}

```

## 6. Copy Files Using System Calls 
```

#include <iostream>
#include <fstream>
#include <cstdio>

using namespace std;

int main()
{
    // Open source file
    ifstream source("source.txt", ios::binary);

    if (!source.is_open())
    {
        cout << "Source file error\n";
        return 1;
    }

    // Open destination file
    ofstream destination("destination.txt",
                         ios::binary | ios::trunc);

    if (!destination.is_open())
    {
        cout << "Destination file error\n";
        return 1;
    }

    char buffer[100];

    // Copy file contents
    while (source.read(buffer, sizeof(buffer)))
    {
        destination.write(buffer, source.gcount());

        if (!destination)
        {
            cout << "Write error\n";
            return 1;
        }
    }

    // Copy remaining data
    if (source.gcount() > 0)
    {
        destination.write(buffer, source.gcount());

        if (!destination)
        {
            cout << "Write error\n";
            return 1;
        }
    }

    // Check read error
    if (source.bad())
    {
        cout << "Read error\n";
        return 1;
    }

    cout << "File copied successfully!\n";

    source.close();
    destination.close();

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
using namespace std;

int a[100];
int sum1 = 0, sum2 = 0;
int n;

void first_thread()
{
    for (int i = 0; i < n / 2; i++)
    {
        sum1 += a[i];
    }
}

void second_thread()
{
    for (int i = n / 2; i < n; i++)
    {
        sum2 += a[i];
    }
}

int main()
{
    cout << "Enter even number of elements: ";
    cin >> n;

    if (n <= 0 || n > 100 || n % 2 != 0)
    {
        cout << "Enter an even number between 2 and 100\n";
        return 1;
    }

    cout << "Enter numbers:\n";

    for (int i = 0; i < n; i++)
    {
        cin >> a[i];
    }

    // Create two threads
    thread t1(first_thread);
    thread t2(second_thread);

    // Wait for both threads
    t1.join();
    t2.join();

    // Display results
    cout << "\nFirst Thread Sum: " << sum1;
    cout << "\nSecond Thread Sum: " << sum2;
    cout << "\nTotal Sum: " << sum1 + sum2;

    return 0;
}
```

## 10. Memory Allocation Strategies
```

#include <iostream>
using namespace std;

int main()
{
    int b[20], p[20];
    int nb, np;

    cout << "Enter number of memory blocks: ";
    cin >> nb;

    cout << "Enter block sizes:\n";
    for (int i = 0; i < nb; i++)
    {
        cin >> b[i];
    }

    cout << "Enter number of processes: ";
    cin >> np;

    cout << "Enter process sizes:\n";
    for (int i = 0; i < np; i++)
    {
        cin >> p[i];
    }

    int choice;

    cout << "\n1. First Fit";
    cout << "\n2. Best Fit";
    cout << "\n3. Worst Fit";

    cout << "\nEnter choice: ";
    cin >> choice;

    if (choice < 1 || choice > 3)
    {
        cout << "Invalid choice!";
        return 0;
    }

    int temp[20], allocated[20];

    // Copy block sizes
    for (int i = 0; i < nb; i++)
    {
        temp[i] = b[i];
    }

    // Initialize allocation
    for (int i = 0; i < np; i++)
    {
        allocated[i] = -1;
    }

    // Memory allocation
    for (int i = 0; i < np; i++)
    {
        int index = -1;

        for (int j = 0; j < nb; j++)
        {
            if (temp[j] >= p[i])
            {
                if (choice == 1)
                {
                    index = j;
                    break;
                }

                if (choice == 2)
                {
                    if (index == -1 || temp[j] < temp[index])
                    {
                        index = j;
                    }
                }

                if (choice == 3)
                {
                    if (index == -1 || temp[j] > temp[index])
                    {
                        index = j;
                    }
                }
            }
        }

        if (index != -1)
        {
            allocated[i] = index;
            temp[index] -= p[i];
        }
    }

    // Display result
    cout << "\nProcess\tSize\tBlock\n";

    for (int i = 0; i < np; i++)
    {
        cout << "P" << i + 1 << "\t"
             << p[i] << "\t";

        if (allocated[i] == -1)
        {
            cout << "Not Allocated\n";
        }
        else
        {
            cout << "B" << allocated[i] + 1 << "\n";
        }
    }

    return 0;
}

```



