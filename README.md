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


``
