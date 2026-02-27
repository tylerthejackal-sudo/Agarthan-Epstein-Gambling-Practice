<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Agarthan Epstein Gambling Practice</title>
<style>
  body { background-color: #013220; font-family: Arial, sans-serif; color: white; text-align: center; margin: 0; padding: 0; }
  h1 { font-size: 3em; font-weight: bold; margin-top: 50px; }
  .button { background-color: #444; border: none; color: white; padding: 15px 30px; margin: 10px; font-size: 1em; cursor: pointer; border-radius: 5px; transition: 0.2s; }
  .button:hover { background-color: #666; }
  #allin-mode { color: gold; font-weight: bold; display: none; margin-top: 10px; }
  #message { margin-top: 20px; min-height: 50px; }
  #coins { font-size: 1.2em; margin-top: 10px; }
  #phone-popup { position: fixed; top: 20%; left: 50%; transform: translateX(-50%); background-color: #222; border: 2px solid #555; padding: 20px; display: none; z-index: 10; border-radius: 10px; }
  #phone-popup input { margin-top:10px; padding:5px; border-radius:5px; width:90%; }
  #dealer-hand, #player-hand { margin-top: 10px; }
  #intro-screen { margin-top: 50px; }
  #game-container { display:none; margin-top: 20px; }
  #music-control { margin-top: 20px; }
</style>
</head>
<body>

<!-- Intro screen -->
<div id="intro-screen">
  <h1>Agarthan Epstein Gambling Practice</h1>
  <p>Choose your difficulty:</p>
  <button class="button" id="easy-btn">Easy</button>
  <button class="button" id="normal-btn">Normal</button>
  <button class="button" id="hard-btn">Hard</button>
</div>

<div id="coins">Coins: 1000</div>

<div id="game-container">
  <div id="music-control">
    <button class="button" id="mute-btn">Mute Music</button>
    <input type="range" id="volume-slider" min="0" max="100" value="30" style="margin-left:10px;">
  </div>

  <div id="drinks-container">
    <button class="button" id="drink-button">Drink Beer 🍺</button>
    <p>Drinks consumed: <span id="drink-count">0</span>/5</p>
    <p id="hint-drink">Hint: Drink 5 beers to enter 🔥 All-In-Mode 🔥 before gambling!</p>
  </div>

  <div id="blackjack-container" style="display:none;">
    <div id="allin-mode">🔥 All-In-Mode Activated 🔥</div>
    <p id="bet-info">Bet per round: 1000 coins | Win: 2000 coins | Tie: 1000 coins returned</p>
    <div id="dealer-hand">Dealer: ???</div>
    <div id="player-hand">Player: ???</div>
    <button class="button" id="hit-button">Hit</button>
    <button class="button" id="stand-button">Stand</button>
    <p id="message"></p>
    <button class="button" id="exit-button">Exit Game</button>
  </div>
</div>

<div id="phone-popup">
  <p>📱 Text your wife:</p>
  <input type="text" id="wife-text-input" placeholder="Type your message here...">
  <button class="button" id="send-text">Send</button>
</div>

<!-- Background music -->
<audio id="bg-music" loop>
  <source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_d7f6d7b574.mp3?filename=cartoons-hava-nagila-445173.mp3" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>

<script>
let coins = 1000, drinks = 0, allInMode = false, wifeUsed = false, firstRound = true;
let bet = 1000, winAmount = 2000, dealerEdge = 0, musicMuted = false;

const bgMusic = document.getElementById('bg-music'); 
const muteBtn = document.getElementById('mute-btn'); 
const volumeSlider = document.getElementById('volume-slider');
bgMusic.volume = volumeSlider.value / 100;
bgMusic.play();

muteBtn.addEventListener('click', ()=>{
  musicMuted = !musicMuted;
  bgMusic.muted = musicMuted;
  muteBtn.textContent = musicMuted ? "Unmute Music" : "Mute Music";
});

volumeSlider.addEventListener('input', () => {
  bgMusic.volume = volumeSlider.value / 100;
  if(bgMusic.volume > 0 && musicMuted) {
    musicMuted = false;
    bgMusic.muted = false;
    muteBtn.textContent = "Mute Music";
  }
});

document.getElementById('easy-btn').addEventListener('click',()=>startGame(-2));
document.getElementById('normal-btn').addEventListener('click',()=>startGame(2));
document.getElementById('hard-btn').addEventListener('click',()=>startGame(4));

function startGame(edge){
  dealerEdge=edge;
  document.getElementById('intro-screen').style.display='none';
  document.getElementById('game-container').style.display='block';
  document.getElementById('blackjack-container').style.display='block';
  document.getElementById('allin-mode').style.display='none';
  message.textContent="Hint: Drink 5 beers 🍺 to enable All-In Mode!";
}

const coinsDisplay = document.getElementById('coins');
const drinkButton = document.getElementById('drink-button');
const drinkCount = document.getElementById('drink-count');
const blackjackContainer = document.getElementById('blackjack-container');
const allInModeText = document.getElementById('allin-mode');
const message = document.getElementById('message');
const dealerHandDiv = document.getElementById('dealer-hand');
const playerHandDiv = document.getElementById('player-hand');
const phonePopup = document.getElementById('phone-popup');
const wifeTextInput = document.getElementById('wife-text-input');
const sendTextButton = document.getElementById('send-text');
const exitButton = document.getElementById('exit-button');
const betInfo = document.getElementById('bet-info');

let playerCards=[], dealerCards=[], playerTotal=0, dealerTotal=0;

function drawCard(){ return Math.floor(Math.random()*13)+1; }
function cardValue(card){ if(card>10) return 10; if(card===1) return 11; return card; }
function handTotal(hand){ let total=hand.reduce((sum,c)=>sum+cardValue(c),0); let aces=hand.filter(c=>c===1).length; while(total>21 && aces>0){total-=10; aces--;} return total; }

drinkButton.addEventListener('click', ()=>{
  drinks++; drinkCount.textContent=drinks;
  if(drinks>=5){ allInMode=true; allInModeText.style.display='block'; document.getElementById('drinks-container').style.display='none'; message.textContent="You're ready to gamble! 🔥 All-In Mode 🔥 is active, but be careful!"; startRound(); }
  else { message.textContent="You feel a little tipsy. Keep drinking..."; }
});

function startRound(){
  if(coins<bet){ if(!wifeUsed){ phonePopup.style.display='block'; return;} else { alert("No coins left. Restarting..."); location.reload(); return; } }

  if(firstRound){ allInMode=true; allInModeText.style.display='block'; document.getElementById('drinks-container').style.display='none'; bet=coins; winAmount=bet*2; betInfo.textContent=`Bet per round: ${bet} coins | Win: ${winAmount} coins | Tie: ${bet} coins returned`; playerCards=[2,3]; dealerCards=[10,7]; firstRound=false; message.textContent="🔥 All-In Mode Activated 🔥 First round is rigged to lose!"; }
  else { playerCards=[drawCard(),drawCard()]; dealerCards=[drawCard(),drawCard()]; }

  coins-=bet; updateCoins();
  playerTotal=handTotal(playerCards); dealerTotal=handTotal(dealerCards);
  dealerHandDiv.textContent="Dealer: ?? + "+dealerCards[1];
  playerHandDiv.textContent="Player: "+playerCards.join(",")+" (Total: "+playerTotal+")";

  const nextBtn=document.getElementById('next-round-button');
  if(nextBtn) nextBtn.style.display='none';
}

function updateCoins(){ coinsDisplay.textContent="Coins: "+coins; }

function dealerPlay(){ while(dealerTotal<19){ if(Math.random()*100<dealerEdge) break; dealerCards.push(drawCard()); dealerTotal=handTotal(dealerCards); } }

function endRound(){
  dealerPlay();
  dealerHandDiv.textContent="Dealer: "+dealerCards.join(",")+" (Total: "+dealerTotal+")";
  playerHandDiv.textContent="Player: "+playerCards.join(",")+" (Total: "+playerTotal+")";

  if(playerTotal>21 || (playerTotal<dealerTotal && dealerTotal<=21)){ message.textContent=`You lose! You lost ${bet} coins.`; }
  else if(dealerTotal>21 || playerTotal>dealerTotal){ coins+=winAmount; message.textContent=`You win! You gain ${winAmount} coins!`; }
  else { coins+=bet; message.textContent="It's a tie! Your bet is returned."; }

  updateCoins();

  let nextBtn=document.getElementById('next-round-button');
  if(!nextBtn){
    nextBtn=document.createElement('button');
    nextBtn.id='next-round-button';
    nextBtn.textContent='Next Round';
    nextBtn.className='button';
    nextBtn.style.marginTop='15px';
    nextBtn.addEventListener('click',()=>startRound());
    blackjackContainer.appendChild(nextBtn);
  } else nextBtn.style.display='inline-block';

  if(coins<=0 && !wifeUsed) phonePopup.style.display='block';
}

document.getElementById('hit-button').addEventListener('click',()=>{
  playerCards.push(drawCard());
  playerTotal=handTotal(playerCards);
  playerHandDiv.textContent="Player: "+playerCards.join(",")+" (Total: "+playerTotal+")";
  if(playerTotal>21) endRound();
});

document.getElementById('stand-button').addEventListener('click',()=>{ endRound(); });

sendTextButton.addEventListener('click',()=>{
  const msg=wifeTextInput.value.trim().toLowerCase();
  const expectedMsg="hey babe got any extra cash? i need it to gamble and donate the rewards to israel";
  if(msg===expectedMsg){ coins+=200; updateCoins(); message.textContent="She agrees! You got 200 coins. ⚠️ Bet now is 20 coins for a 40 coin reward if you win."; wifeUsed=true; phonePopup.style.display='none'; bet=20; winAmount=40; betInfo.textContent=`Bet per round: ${bet} coins | Win: ${winAmount} coins | Tie: ${bet} coins returned`; setTimeout(()=>{message.textContent+=" Hint: You *could* drink 5 beers 🍺 to enter All-In Mode, but it's not advised!";},1000);}
  else alert("Your wife ignores the message. Try being more specific...");
  wifeTextInput.value="";
});

exitButton.addEventListener('click',()=>{ message.textContent="Maybe one more game… 😉"; });
</script>
</body>
</html>



Where can I make this game