# GoodFour - Dynamic Hints System

This repository showcases the dynamic, point-based hints system designed for **GoodFour**, a word-grouping puzzle game. The system is engineered to provide strategic assistance to players without undermining the core challenge of the puzzle.

## System Overview & Design Philosophy

The Hints System operates on a simple economy: players start with 3 "hint points" and can spend them on hints of increasing value. This tiered structure encourages strategic thinking rather than simple guessing.

My design philosophy for this feature was centered on three principles:

1.  **Strategic Depth:** The system should feel like a tool, not a cheat. By assigning different costs, it prompts the player to evaluate how much help they truly need.
2.  **Modularity:** The hint logic is self-contained and can be easily disabled (as it is for "Hard Mode"), ensuring the core game engine remains independent.
3.  **Data-Driven Balancing:** The `hintUsage` object tracks which hints players use most often, providing valuable data for future puzzle design and difficulty tuning.

## Hint Types & Costs

| Hint Type               | Point Cost | Functionality                                                               |
| ----------------------- | :--------: | --------------------------------------------------------------------------- |
| **Get Category Clue**   |     1      | Reveals a broad, thematic clue for one of the unsolved categories.          |
| **Find a Pair**         |     2      | Highlights two words on the board that belong in the same hidden group.     |
| **Remove One Wrong Tile**|     3      | When a player has four tiles selected, this removes one incorrect tile.     |

## Code Highlight: `removeOneWrongTileHint()`

To demonstrate the implementation, the function below is the most complex of the three hints. It intelligently identifies which tile to remove from a player's selection.

The logic first checks for a "one-away" combination (three correct tiles). If that's not the case, it checks for a "two-away" combination. This ensures the hint is as helpful as possible. Only if no logical connection is found does it resort to removing a random tile.

```javascript
/**
 * Spends 3 points to remove one incorrect tile from a selection of four.
 */
function removeOneWrongTileHint() {
    if (hintPoints < 3) {
        showToast("Not enough points!");
        return;
    }
    if (selectedTiles.length !== 4) {
        showToast("Select 4 tiles first!");
        return;
    }
    hintPoints -= 3;
    hintUsage.removeWrongTile += 1;
    hintUsage.pointsSpent += 3;

    const selectedWords = selectedTiles.map(tile => tile.dataset.word);
    let tileToRemove = null;

    // First, check for a "one away" scenario (3 correct tiles)
    for (const group of gameData) {
        const correctWordsInSelection = selectedWords.filter(word => group.words.includes(word));
        if (correctWordsInSelection.length === 3) {
            const wrongWord = selectedWords.find(word => !group.words.includes(word));
            tileToRemove = selectedTiles.find(tile => tile.dataset.word === wrongWord);
            break;
        }
    }

    // If not "one away", check for a "two away" scenario
    if (!tileToRemove) {
        for (const group of gameData) {
            const correctWordsInSelection = selectedWords.filter(word => group.words.includes(word));
            if (correctWordsInSelection.length === 2) {
                const wrongWords = selectedWords.filter(word => !group.words.includes(word));
                const randomWrongWord = wrongWords[Math.floor(Math.random() * wrongWords.length)];
                tileToRemove = selectedTiles.find(tile => tile.dataset.word === randomWrongWord);
                break;
            }
        }
    }

    // Fallback: if no logical connection is found, remove a random tile
    if (!tileToRemove) {
        tileToRemove = selectedTiles[Math.floor(Math.random() * selectedTiles.length)];
    }

    if (tileToRemove) {
        tileToRemove.classList.remove('selected');
        selectedTiles = selectedTiles.filter(t => t !== tileToRemove);
        updateButtonStates();
        hintModalOverlay.style.display = 'none';
    }
}
