# Brain-game-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Memory Brain Game</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
        }
        
        .game-container {
            width: 400px;
            perspective: 1000px;
        }
        
        .game-title {
            text-align: center;
            color: #3a4a6d;
            font-size: 2.5rem;
            margin-bottom: 20px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
        
        .score-container {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
            font-size: 1.2rem;
            color: #3a4a6d;
        }
        
        .card-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            grid-gap: 10px;
        }
        
        .card {
            position: relative;
            height: 80px;
            cursor: pointer;
            transform-style: preserve-3d;
            transition: transform 0.5s;
        }
        
        .card.flipped {
            transform: rotateY(180deg);
        }
        
        .card-face {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 2rem;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        
        .card-front {
            background: linear-gradient(45deg, #3a4a6d, #5b6b8f);
            color: white;
            transform: rotateY(180deg);
        }
        
        .card-back {
            background: white;
            color: #3a4a6d;
            border: 2px solid #3a4a6d;
        }
        
        .card.matched {
            animation: bounce 0.5s;
            opacity: 0.7;
            cursor: default;
        }
        
        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-10px); }
        }
        
        .controls {
            margin-top: 30px;
        }
        
        button {
            background: #3a4a6d;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 1rem;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        button:hover {
            background: #5b6b8f;
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        
        .win-message {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0,0,0,0.8);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 2rem;
            z-index: 100;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.5s;
        }
        
        .win-message.show {
            opacity: 1;
            pointer-events: all;
        }
        
        .win-message button {
            margin-top: 20px;
            font-size: 1.2rem;
        }
        
        .confetti {
            position: absolute;
            width: 10px;
            height: 10px;
            background-color: #f00;
            opacity: 0;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1 class="game-title">Memory Game</h1>
        <div class="score-container">
            <div>Moves: <span id="moves">0</span></div>
            <div>Matches: <span id="matches">0</span>/8</div>
        </div>
        <div class="card-grid" id="cardGrid"></div>
        <div class="controls">
            <button id="resetBtn">Reset Game</button>
        </div>
    </div>
    
    <div class="win-message" id="winMessage">
        <h2>Congratulations!</h2>
        <p>You won in <span id="finalMoves">0</span> moves!</p>
        <button id="playAgainBtn">Play Again</button>
    </div>
    
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const cardGrid = document.getElementById('cardGrid');
            const movesDisplay = document.getElementById('moves');
            const matchesDisplay = document.getElementById('matches');
            const resetBtn = document.getElementById('resetBtn');
            const winMessage = document.getElementById('winMessage');
            const finalMoves = document.getElementById('finalMoves');
            const playAgainBtn = document.getElementById('playAgainBtn');
            
            let cards = [];
            let flippedCards = [];
            let moves = 0;
            let matches = 0;
            let canFlip = true;
            
            // Emojis for the cards
            const emojis = ['🐶', '🐱', '🐭', '🐹', '🐰', '🦊', '🐻', '🐼'];
            
            // Initialize the game
            function initGame() {
                // Reset game state
                cards = [];
                flippedCards = [];
                moves = 0;
                matches = 0;
                canFlip = true;
                
                // Clear the card grid
                cardGrid.innerHTML = '';
                
                // Update displays
                movesDisplay.textContent = moves;
                matchesDisplay.textContent = `${matches}/8`;
                
                // Hide win message
                winMessage.classList.remove('show');
                
                // Create pairs of cards
                const cardValues = [...emojis, ...emojis];
                
                // Shuffle the cards
                shuffleArray(cardValues);
                
                // Create card elements
                cardValues.forEach((value, index) => {
                    const card = document.createElement('div');
                    card.classList.add('card');
                    card.dataset.index = index;
                    card.dataset.value = value;
                    
                    const cardBack = document.createElement('div');
                    cardBack.classList.add('card-face', 'card-back');
                    cardBack.textContent = '?';
                    
                    const cardFront = document.createElement('div');
                    cardFront.classList.add('card-face', 'card-front');
                    cardFront.textContent = value;
                    
                    card.appendChild(cardBack);
                    card.appendChild(cardFront);
                    
                    card.addEventListener('click', flipCard);
                    
                    cardGrid.appendChild(card);
                    cards.push(card);
                });
            }
            
            // Flip a card
            function flipCard() {
                if (!canFlip || this.classList.contains('flipped') || this.classList.contains('matched')) {
                    return;
                }
                
                this.classList.add('flipped');
                flippedCards.push(this);
                
                if (flippedCards.length === 2) {
                    canFlip = false;
                    moves++;
                    movesDisplay.textContent = moves;
                    
                    checkForMatch();
                }
            }
            
            // Check if the flipped cards match
            function checkForMatch() {
                const [card1, card2] = flippedCards;
                
                if (card1.dataset.value === card2.dataset.value) {
                    // Match found
                    card1.classList.add('matched');
                    card2.classList.add('matched');
                    matches++;
                    matchesDisplay.textContent = `${matches}/8`;
                    
                    flippedCards = [];
                    canFlip = true;
                    
                    // Check for win
                    if (matches === 8) {
                        setTimeout(() => {
                            finalMoves.textContent = moves;
                            winMessage.classList.add('show');
                            createConfetti();
                        }, 500);
                    }
                } else {
                    // No match
                    setTimeout(() => {
                        card1.classList.remove('flipped');
                        card2.classList.remove('flipped');
                        flippedCards = [];
                        canFlip = true;
                    }, 1000);
                }
            }
            
            // Shuffle array function
            function shuffleArray(array) {
                for (let i = array.length - 1; i > 0; i--) {
                    const j = Math.floor(Math.random() * (i + 1));
                    [array[i], array[j]] = [array[j], array[i]];
                }
                return array;
            }
            
            // Create confetti animation
            function createConfetti() {
                const colors = ['#f00', '#0f0', '#00f', '#ff0', '#f0f', '#0ff'];
                
                for (let i = 0; i < 100; i++) {
                    const confetti = document.createElement('div');
                    confetti.classList.add('confetti');
                    confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
                    confetti.style.left = `${Math.random() * 100}vw`;
                    confetti.style.top = '-10px';
                    confetti.style.transform = `rotate(${Math.random() * 360}deg)`;
                    
                    document.body.appendChild(confetti);
                    
                    // Animate confetti
                    const animationDuration = Math.random() * 3 + 2;
                    const animationDelay = Math.random() * 2;
                    
                    confetti.style.transition = `top ${animationDuration}s linear ${animationDelay}s, opacity ${animationDuration}s linear ${animationDelay}s`;
                    
                    setTimeout(() => {
                        confetti.style.top = `${Math.random() * 100 + 100}vh`;
                        confetti.style.opacity = '0';
                    }, 10);
                    
                    // Remove confetti after animation
                    setTimeout(() => {
                        confetti.remove();
                    }, (animationDuration + animationDelay) * 1000 + 100);
                }
            }
            
            // Event listeners
            resetBtn.addEventListener('click', initGame);
            playAgainBtn.addEventListener('click', initGame);
            
            // Start the game
            initGame();
        });
    </script>
</body>
</html>

