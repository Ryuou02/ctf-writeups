description - <br>
  Follow the leader.
<br>
first I viewed the linked scripts 
<script src="/mojojs/mojo_bindings.js"></script> 
and 
<script src="/prog.js"></script>

the first one just turned out to be file that is part of the framework used to make the application.
the second file, I have no idea what it was, I spent some time looking at it, it looked like shellcode.

it is very huge but it looks something like

let code = 'y\x11\x00\x00\x00\x00\x00\x00\x85\x10\x00\x00h\x0c\x00\x00\x95\x00\x00\x00\x00\x00\x00\x00{\x ...
let entrypoint = 1170

maybe it is part of the framework, I'm not sure. Anyways, the main part of the challenge is quite simple. On checking the page source, I found the code
<code>let won = false;</code>
<code>

    function win(history) {
    const code = encode(history) + ";" + prompt("Name?");
    
    const saveURL = location.origin + "?code=" + code;
    displaywrapper.classList.remove("hidden");

    const score = history.length;

    display.children[1].innerHTML = "Your score was: <b>" + score + "</b>";
    display.children[2].href =
      "https://twitter.com/intent/tweet?text=" +
      encodeURIComponent(
        "Can you beat my score of " + score + " in Dice Dice Goose?",
      ) +
      "&url=" +
      encodeURIComponent(saveURL);

    if (score === 9) log("flag: dice{pr0_duck_gam3r_" + encode(history) + "}");
    }
  
    const replayCode = new URL(location.href).searchParams.get("code");
  
    if (replayCode !== null) replay();
  
    let validated = false;
    async function replay() {
      if (!(await validate())) {
        log("Failed to validate");
        return;
    }
    
</code>
