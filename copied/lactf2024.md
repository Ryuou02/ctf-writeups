# new housing portal
source - https://medium.com/@dassomnath/new-housing-portal-web-write-up-lactf-2024-110b068fb19f
This is basically an application, where you register with a **username**, **password**, **name** and **deepest darkest secret**.
After registring, we can log in.
On logging in we have option to find other roommates and to view invitations.

1. when we click 'find other roommates', we can search for their name, it sends a get request to `/finder` with get parameter `q` having value of our username
2. when we click 'view invitations', we can see the invitations sent from other users as well as their deepest darkest secret => *our goal is to get this from samy*

so all we need to do is make samy search for our username and send us an invite. 
In the finder page, when a username is found to exist, it will be displayed in the list, after the get request for that particular username is sent. i.e. first you find the user, then you can invite the user.

basically, we make the bot to send a get request, so that it searches for our username. Then it should also click 'invite' on doing so.
so we make our username like - 
`user<img src=s onerror='document.invite.submit()'> ` 
so that the username itself will cause the bot to click submit, as it loads.
we make the bot to send get request like - 
```http://localhost:3000/finder?q=user<img+src%3Ds+onerror%3D'document.invite.submit()'>```
