I don't feel like going to the source code directly for this one since I'm kind of lazy.
Here's what the challenge looks like -
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/08b04c8a-cdce-40ea-b246-67bc74574e99)
on losing the game, an alert box displays like - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/c37d5831-a39a-41a4-ac75-e32f34231e93)
on viewing the page source, I see - 
```

const wsurl = new URL(location);
wsurl.protocol = location.protocol == 'http:' ? 'ws' : 'wss';
wsurl.pathname = '/ws';

const ws = new WebSocket(wsurl);

ws.addEventListener('open', () => {
  ws.addEventListener('message', (e) => {
    const msg = JSON.parse(e.data);
    switch (msg[0]) {
      case Msg.GAME_UPDATE:
        ballPos = serverToViewport(msg[1][0]);
        serverPos = serverToViewport(msg[1][1]);
        updateFromRemote();
        break;
      case Msg.GAME_END:
        alert(msg[1]);
        break;
    }
  })
```
from which we can understand it is using websocket to communicate to the server, and the server sends a message on the game ending. I had a look at the server code and it looks like this
```
      // wall bouncing
      if (ball[1] < -yMax || ball[1] > yMax) {
        ball[1] = clamp(ball[1], -yMax, yMax);
        ballV[1] *= -1;
      }

      // check if there has been a winner
      // server wins
      if (ball[0] < 0) {
        ws.send(JSON.stringify([
          Msg.GAME_END,
          'oh no you have lost, have you considered getting better'
        ]));
        clearInterval(interval);

      // game still happening
      } else if (ball[0] < 100) {
        ws.send(JSON.stringify([
          Msg.GAME_UPDATE,
          [ball, me]
        ]));

      // user wins
      } else {
        ws.send(JSON.stringify([
          Msg.GAME_END,
          'omg u won, i guess you considered getting better ' +
          'here is a flag: ' + flag,
          [ball, me]
        ]));
        clearInterval(interval);
      }
```
you can leave all the difficult stuff and just leave the game as it is, in the default position of user ball. then eventually the server stops responding on time and you get the flag, GG
