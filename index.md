# Rizzistence Nights

Rizzistence Nights is a web party game I built as a prototype for Rizz Wireless. For context, I work for a mobile network startup called Rizz Wriless and they wanted to know if i had any sugetions for new ways they could market using creative tech. I suggested a web based party game like Quiplash or Cards against humanity that they could sponser. The goal was to show how a browser game could be used as a marketing tool at social events.

This version is an early prototype meant to show the full pipeline which i built to pitch to them: gameplay, React frontend, Firebase backend, and real-time multiplayer features.



## Concept

I took inspiration from games like Quiplash and Jackbox. The idea was to make something that followd these requierments:

- Joinable easaly from any phone  
- No downloads or host required  
- It should be ast, social, and creative 
- Built around simple prompts and constraints that create funny answers

Players answer a prompt, vote on the answers, and compete across a few rounds. The entire game runs automatically.



## How the System Works

### Frontend (React)
The React app handles:
- Joining the game (name + icon)
- Answering prompts  
- Voting  
- Showing the scoreboard and results  
- Switching screens based on game state  

Each screen is its own component, and the main `App.jsx` decides which one to show based on the current phase.

### Firebase (Backend)
Firebase handles all real-time data and syncing:

- **games/**  
  Stores the current round, phase, prompt, constraint, and timestamps  
- **players/**  
  Stores player name, icon, score, and which game they’re in  
- **answers/**  
  Stores each submitted answer and its vote count  

Listeners update every player's screen instantly when anything changes.

### Data Flow
User > React Component > useGame() Hook > Firestore > All Players Update


## What I Learned and Feedback

**What worked well**
- The real time syncing was smooth using Firestore listeners  
- React + Firebase kept the code small and easy to maintain  
- the gameplay was an easy drop-in/drop-out system since players aren’t “hosts”  

**What I would improve**
- the timer and time constraint was too stressful
- not eveyone wants to be creative when having fun so forcing people to come up with creative responces on the spot might not be fun as a game.
- Add better constraints and more interesting prompts  
- Improve timer accuracy on mobile (there was a slight timer delay on some phones)

## WALKTHROUGH for useGame.js

This hook controls the whole game: joining, syncing, timers, answers, voting, and moving between rounds. 

### 1. Local State
When it starts It keeps track of:
- `gameState` – the current round, phase, prompt, etc.
- `players` – everyone in the game
- `answers` – answers for the current round
- `timeLeft` – how many seconds are left in the phase

### 2. Player "Heartbeat"
When a player joins, we start updating their `lastPing` in Firestore every 8 seconds.  
If they close the tab or go inactive, the pings stop. This is how I detect and remove inactive players.

### 3. Auto-Cleanup for Inactive Players
Every 5 seconds, it looks at everyone in the current game
 If a player hasn’t pinged in around 15 seconds then it deletes them  

### 4. Joining or Creating a Game
When the hook first runs
- checks if there’s any game already running (`waiting` or `playing`)
- If yes then join it  
- If no then make a new one with a random prompt + constraint

### 5. Listening for Answers
Whenever the `gameState` changes, we listen for `answers` from the current round only.  
This way players only ever see answers for the active round.

### 6. Timers
Each phase has a fixed length:
- Answer phase  `ANSWER_TIME`
- Voting phase  `VOTING_TIME`

1. Looks at when the phase started (`phaseStartTime`)
2. Calculate how much time is left
3. Count down every second
4. When it hits 0, it auto-switchs the phase

This keeps all players roughly synced without needing server logic..... this part i need to work on more

### 7. Moving From Answering to Voting
When the answer timer hits 0, we call `advancePhase()`:
- Sets phase to `voting`
- Resets the timer with a new `phaseStartTime`

### 8. Moving to the Next Round
When voting ends:
- shortlists answrs witht he most votes
- Whoever has the highest votes gets +1 point  
- If it was the last round then show results  
- Otherwise start a new round with a new prompt + constraint

### Summary
The hook basically acts as the game’s control center:
- Handles joining  
- Syncs everything in real-time  
- Keeps timers consistent  
- Moves the game forward automatically  
- Stores answers and votes  
- Cleans up inactive players  

The frontend stays simple because all the logic lives inside this hook.
