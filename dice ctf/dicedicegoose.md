description - <br>
  Follow the leader.
<br>
##what the challenge is about
this is a simple game, where you have a player and a goose, on a 11x20 grid and there is a wall placed. your player can move up, down, left or right and the goose also moves up, down, left or right by 1 cell in the grid, randomly.
the main goal of the challenge is to win the game in least number of moves.

#how to solve
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
and the function - 
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


I called the above function using the console debugger (which is part of the developer tools). It allows you to win, and returns a flag with the history of your player and the movement of the goose encoded in it. I tried teleporting the player right in front of the goose and many things that'll give me victory in 9 moves, but I missed a key point in the code that is - 
<code>

    document.onkeypress = (e) => {
    if (won) return;

    let nxt = [player[0], player[1]];

    switch (e.key) {
      case "w":
        nxt[0]--;
        break;
      case "a":
        nxt[1]--;
        break;
      case "s":
        nxt[0]++;
        break;
      case "d":
        nxt[1]++;
        break;
    }

    if (!isValid(nxt)) return;

    player = nxt;

    if (player[0] === goose[0] && player[1] === goose[1]) {
      win(history);
      won = true;
      return;
    }

    do {
      nxt = [goose[0], goose[1]];
      switch (Math.floor(4 * Math.random())) {
        case 0:
          nxt[0]--;
          break;
        case 1:
          nxt[1]--;
          break;
        case 2:
          nxt[0]++;
          break;
        case 3:
          nxt[1]++;
          break;
      }
    } while (!isValid(nxt));

    goose = nxt;

    history.push([player, goose]);

    redraw();
    };
  
</code>

here, you can see towards the end, there is <code>history.push([player, goose]);</code> this means that the history that gets encoded depends on the position of both the player and goose, not just 1 of them. I had overlooked this and assumed that it was just the history of the player's movements. So what I did was, I called the function <code>walls.push(x,y)</code> to create a wall such that the goose only has 1 direction in which it could move. Hence creating a situation which would be really rare(the goose moving in a straight line for 9 blocks) to occur.The goose needs to be made to move in a straight line for 9 blocks till it reaches our player, which we move down.
in order to get this to happen, you place horizontal walls above the goose and below the goose so that it can only move forward or backward, and then place a wall right behind the goose so that it doesn't move backward. and every step you move the player down, the goose will move forward, that time you need to place a wall right behind the goose and keep doing this for 9 moves till the player and goose meet.
