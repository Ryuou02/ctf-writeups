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
        " AND ".join(["{} = '{}'".format(k, v) for k, v in prefs.items()])
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
so that I can see the query being executed in the backend. unlike what I expected, 5 columns are getting returned and not just 4. If some common sense is applied it totally makes sense but I have no idea why I didn't think of it
all we need to do is basic sqli with a union statement.

...it seems that I forgot how I solved this challenge hence I need to open docker and see where I'm making the error, again :')
Anyway, I'll use this opportunity to show how I run the challenge in docker and make place print statements -
1. I unzip the file and run the command ``` docker build . -t ctf_chall``` which makes the docker image and tags it as 'ctf_chall'. then I open docker and run the image by keeping the host port as 0.
2. ![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/4cc9bd24-1f4a-40ee-bd2a-a2a5df342f45)

