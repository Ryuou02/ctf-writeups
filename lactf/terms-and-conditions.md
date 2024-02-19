This is a rather funny challenge in which we're supposed to accept the terms and conditions however, the "I accept" button keeps moving away from the cursor
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/13f7770a-ff12-4c98-aef8-507d63bbd6c6)
on viewing the page source we see
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/166cca3e-9d19-4335-a5e7-6f6d238e6e2a)
we just need to focus on the addeventlistener part, where it changes the position of the button based on the position of the mouse. Now all we need to do to solve the challenge is open the console and change the variable values
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/5db4cfe5-b750-4401-9822-bf3a2f33f701)
however, they seem to not allow console? It is because of this in the page source - 
```
    setInterval(function() {
            if (window.innerHeight !== height || window.innerWidth !== width) {
                document.body.innerHTML = "<div><h1>NO CONSOLE ALLOWED</h1></div>";
                height = window.innerHeight;
                width = window.innerWidth;
            }
        }, 10);
```
And the height and width variables are set when the page loads.
we just need to reload the page while keeping the console debugger open to avoid this.
then we change mx to 0
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/9db06dc1-2ebd-4ce5-98cc-e27a1a20d1c2)
then we get the flag.
