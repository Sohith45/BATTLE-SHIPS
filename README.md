#include <iostream>
#include <cstdlib>  // For rand() and srand()
#include <ctime>    // For time()
using namespace std;

const int gridSize = 10;
const char emptyCell = '-';
const char shipCell = 'S';
const char hitCell = 'H';
const char missCell = 'M';

void initializeGrid(char grid[gridSize][gridSize]) {
    for (int i = 0; i < gridSize; i++) {
        for (int j = 0; j < gridSize; j++) {
            grid[i][j] = emptyCell;
        }
    }
}

void printGrid(char grid[gridSize][gridSize], bool hideShips = false) {
    cout << "  A B C D E F G H I J" << endl;
    for (int i = 0; i < gridSize; i++) {
        cout << i + 1 << " ";
        for (int j = 0; j < gridSize; j++) {
            if (hideShips && grid[i][j] == shipCell)
                cout << emptyCell << " ";
            else
                cout << grid[i][j] << " ";
        }
        cout << endl;
    }
}

bool isValidPlacement(char grid[gridSize][gridSize], int row, int col, int length, char direction) {
    if (direction == 'H') {
        if (col + length > gridSize) return false;
        for (int i = 0; i < length; i++) {
            if (grid[row][col + i] != emptyCell) return false;
        }
    } else if (direction == 'V') {
        if (row + length > gridSize) return false;
        for (int i = 0; i < length; i++) {
            if (grid[row + i][col] != emptyCell) return false;
        }
    }
    return true;
}

void placeShip(char grid[gridSize][gridSize], int length) {
    int row, col;
    char direction;
    do {
        row = rand() % gridSize;
        col = rand() % gridSize;
        direction = rand() % 2 == 0 ? 'H' : 'V'; // Horizontal or Vertical
    } while (!isValidPlacement(grid, row, col, length, direction));

    if (direction == 'H') {
        for (int i = 0; i < length; i++) {
            grid[row][col + i] = shipCell;
        }
    } else {
        for (int i = 0; i < length; i++) {
            grid[row + i][col] = shipCell;
        }
    }
}

void placeAllShips(char grid[gridSize][gridSize]) {
    placeShip(grid, 5); // Carrier
    placeShip(grid, 4); // Battleship
    placeShip(grid, 3); // Cruiser
    placeShip(grid, 3); // Submarine
    placeShip(grid, 2); // Destroyer
}

bool takeShot(char grid[gridSize][gridSize], int row, int col) {
    if (grid[row][col] == shipCell) {
        grid[row][col] = hitCell;
        cout << "It's a hit!" << endl;
        return true;
    } else if (grid[row][col] == emptyCell) {
        grid[row][col] = missCell;
        cout << "It's a miss!" << endl;
        return false;
    } else {
        cout << "You already fired there!" << endl;
        return false;
    }
}

bool allShipsSunk(char grid[gridSize][gridSize]) {
    for (int i = 0; i < gridSize; i++) {
        for (int j = 0; j < gridSize; j++) {
            if (grid[i][j] == shipCell) return false;
        }
    }
    return true;
}

void playerTurn(char enemyGrid[gridSize][gridSize]) {
    string shot;
    int row, col;
    cout << "Enter your shot (e.g., A5): ";
    cin >> shot;

    col = shot[0] - 'A'; // Convert column letter to index
    row = stoi(shot.substr(1)) - 1; // Convert row number to index

    if (row < 0 || row >= gridSize || col < 0 || col >= gridSize) {
        cout << "Invalid coordinates! Try again." << endl;
        playerTurn(enemyGrid); // Recursive call if input is invalid
    } else {
        takeShot(enemyGrid, row, col);
    }
}

void computerTurn(char playerGrid[gridSize][gridSize]) {
    int row, col;
    do {
        row = rand() % gridSize;
        col = rand() % gridSize;
    } while (playerGrid[row][col] == hitCell || playerGrid[row][col] == missCell);
    
    cout << "Computer fires at: " << char('A' + col) << row + 1 << endl;
    takeShot(playerGrid, row, col);
}

int main() {
    srand(time(0));  // Seed random number generator
    
    char playerGrid[gridSize][gridSize], computerGrid[gridSize][gridSize];
    initializeGrid(playerGrid);
    initializeGrid(computerGrid);

    // Place computer's ships
    placeAllShips(computerGrid);
    
    // For demo purposes, you can print the computer's grid to see ship locations (optional)
    // cout << "Computer's grid (hidden):" << endl;
    // printGrid(computerGrid);

    // Place player's ships manually (could be randomized or manually entered)
    cout << "Placing player's ships..." << endl;
    placeAllShips(playerGrid);

    // Game loop
    while (true) {
        cout << "\nPlayer's Grid:" << endl;
        printGrid(playerGrid);
        cout << "\nComputer's Grid (your shots):" << endl;
        printGrid(computerGrid, true);

        // Player's turn
        playerTurn(computerGrid);
        if (allShipsSunk(computerGrid)) {
            cout << "You sank all the computer's ships! You win!" << endl;
            break;
        }

        // Computer's turn
        computerTurn(playerGrid);
        if (allShipsSunk(playerGrid)) {
            cout << "The computer sank all your ships! You lose!" << endl;
            break;
        }
    }

    return 0;
}
