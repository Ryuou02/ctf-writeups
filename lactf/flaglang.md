the first thing I do when I open the source code is to open the Docker file of the challenge for possible hints like if the flag is in a text file and where it is, within the server in case it is a text file. However, there seems to be nothing there.
however in the Docker file we see
```
CMD ["node", "/app/src/app.js"]
```
this indicates that it is a node application with app.js a startpoint. This is quite obvious by looking at the files but still, this is another way to tell the backend of the server.
on opening the countries.yaml file, in the first few lines, we see

```
  %YAML 1.1
  ---
  Flagistan:
  iso: FL
  msg: "<REDACTED>"
  password: "<REDACTED>"
  deny: 
    ["AF","AX","AL","DZ","AS","AD","AO","AI","AQ","AG","AR","AM","AW","AU","AT","AZ","BS","BH","BD","BB","BY","BE","BZ","BJ","BM","BT","BO","BA","BW","BV","BR","IO","BN","BG","BF","BI","KH","CM","CA","CV","KY","CF","TD","CL","CN","CX","CC","CO","KM","CG","CD","CK","CR","CI","HR","CU","CY","CZ","DK","DJ","DM","DO","EC","EG","SV","GQ","ER","EE","ET","FK","FO","FJ","FI","FR","GF","PF","TF","GA","GM","GE","DE","GH","GI","GR","GL","GD","GP","GU","GT","GG","GN","GW","GY","HT","HM","VA","HN","HK","HU","IS","IN","ID","IR","IQ","IE","IM","IL","IT","JM","JP","JE","JO","KZ","KE","KI","KR","KP","KW","KG","LA","LV","LB","LS","LR","LY","LI","LT","LU","MO","MK","MG","MW","MY","MV","ML","MT","MH","MQ","MR","MU","YT","MX","FM","MD","MC","MN","ME","MS","MA","MZ","MM","NA","NR","NP","NL","AN","NC","NZ","NI","NE","NG","NU","NF","MP","NO","OM","PK","PW","PS","PA","PG","PY","PE","PH","PN","PL","PT","PR","QA","RE","RO","RU","RW","BL","SH","KN","LC","MF","PM","VC","WS","SM","ST","SA","SN","RS","SC","SL","SG","SK","SI","SB","SO","ZA","GS","ES","LK","SD","SR","SJ","SZ","SE","CH","SY","TW","TJ","TZ","TH","TL","TG","TK","TO","TT","TN","TR","TM","TC","TV","UG","UA","AE","GB","US","UM","UY","UZ","VU","VE","VN","VG","VI","WF","EH","YE","ZM","ZW"]

```
now we know exactly which country we need, in order to get the flag.
Now I finally open the challenge. (earlier opening the challenge was the first thing i did but now I just view the source code first, in case it is provided)
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/81e31972-dea8-4285-a25d-5d1e83aeb933)
This seems like a challenge where we simply change the flag we want to view and then we can view a flag and the message behind it.
now, I open burpsuit, since I want to modify the flag I want to view
I clicked to view Timore-Leste's flag and this is what the intercepted request looks like
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/75aec169-ff4a-427c-904c-f38a5799cd59)
so the request being sent has 2 parts - 
1 is the get request parameter which is totally obvious
the second is the ``iso`` cookie which has some text indicating the initial 2 characters of the country followed by random text. We need to change the initials to 'FL' and the country name to 'Flagistan' as we can see that this is what we need, from the yaml file.
hence I modified the request to look like -
<img width="1081" alt="burpsuit" src="https://github.com/Ryuou02/ctf-writeups/assets/133224167/cd00e650-92be-461c-af1a-6c3b0b6116d3">
hence we get the flag - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/eec7bd7d-27f3-4902-9fdf-81c4d06cb2f7)


