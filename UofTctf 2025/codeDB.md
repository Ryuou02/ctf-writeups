This challenge allows us to search for words or expressions from a list of files containing programs written in different programming langauages.

![image](https://github.com/user-attachments/assets/9721e3de-a5a3-489b-911e-0cbf942ec3c6)

On viewing the code_samples folder within the source code, we can see that there is a file `flag.txt`.


On viewing the source code, we can see that the visibility of flag.txt file is set to false.
```
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
```
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




