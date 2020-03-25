// Scroll down to "About" for instructions on this project ↓

var initialCount;
var currentScene = 1;



var firstScene = function(buts) {
    currentScene = 1;
    background(255, 255, 255);
    fill(0, 0, 0);
    text("Welcome to Memory++!\nClick the button below to start", 200, 200);
    buts.draw();
    buts.mousePressed(buts);
};
var Button = function(x, y) {
    this.x = x;
    this.y = y;
    this.width = 200;
    this.height = 120;
};
var restartButton = new Button(100, 255);
var playButton = new Button(100, 255);

Button.prototype.draw = function() {
    rect(this.x, this.y, this.width, this.height, 10);
};

Button.prototype.mousePressed = function(buttonType) {
        if (buttonType === restartButton) {
            if (mouseX >= this.x && mouseX <= this.x + this.width && mouseY >= this.y && mouseY <= this.y + this.height) {
                Program.restart();    
            }
        }
        if (buttonType === playButton) {
            if (mouseX >= this.x && mouseX <= this.x + this.width && mouseY >= this.y && mouseY <= this.y + this.height) {
                println("Hello");    
            }
        }
};




var Tile = function(x, y, face) {
    this.x = x;
    this.y = y;
    this.width = 70;
    this.face = face;
    this.isFaceUp = false;
    this.isMatch = false;
    this.color = color(214, 247, 202);
};

Tile.prototype.draw = function() {
    fill(this.color);
    strokeWeight(2);
    rect(this.x, this.y, this.width, this.width, 10);
    if (this.isFaceUp) {
        image(this.face, this.x, this.y, this.width, this.width);
    } else {
        image(getImage("avatars/leaf-green"), this.x, this.y, this.width, this.width);
    }
};

Tile.prototype.isUnderMouse = function(x, y) {
    return mouseX >= this.x && mouseX <= this.x + this.width  &&
        mouseY >= this.y && mouseY <= this.y + this.width;
};

var secondScene = function(tiled) {
    for (var i = 0; i < tiled.length; i++) {
        tiled[i].draw();
        
    }    
};

// Global config
var NUM_COLS = 5;
var NUM_ROWS = 4;


// Declare an array of all possible faces
var faces = [
    getImage("avatars/leafers-seed"),
    getImage("avatars/leafers-seedling"),
    getImage("avatars/leafers-sapling"),
    getImage("avatars/leafers-tree"),
    getImage("avatars/leafers-ultimate"),
    getImage("avatars/marcimus"),
    getImage("avatars/mr-pants"),
    getImage("avatars/mr-pink"),
    getImage("avatars/old-spice-man"),
    getImage("avatars/robot_female_1")
];

// Make an array which has 2 of each, then randomize it
var possibleFaces = faces.slice(0);
var selected = [];
for (var i = 0; i < (NUM_COLS * NUM_ROWS) / 2; i++) {
    // Randomly pick one from the array of remaining faces
    var randomInd = floor(random(possibleFaces.length));
    var face = possibleFaces[randomInd];
    // Push twice onto array
    selected.push(face);
    selected.push(face);
    // Remove from array
    possibleFaces.splice(randomInd, 1);
}

// Now shuffle the elements of that array
var shuffleArray = function(array) {
    var counter = array.length;

    // While there are elements in the array
    while (counter > 0) {
        // Pick a random index
        var ind = Math.floor(Math.random() * counter);
        // Decrease counter by 1
        counter--;
        // And swap the last element with it
        var temp = array[counter];
        array[counter] = array[ind];
        array[ind] = temp;
    }
};
shuffleArray(selected);

// Create the tiles
var tiles = [];
for (var i = 0; i < NUM_COLS; i++) {
    for (var j = 0; j < NUM_ROWS; j++) {
        var tileX = i * 78 + 10;
        var tileY = j * 78 + 40;
        var tileFace = selected.pop();
        tiles.push(new Tile(tileX, tileY, tileFace));
    }
}

background(255, 255, 255);

var numTries = 0;
var numMatches = 0;
var flippedTiles = [];
var delayStartFC = null;

mouseClicked = function() {
    for (var i = 0; i < tiles.length; i++) {
        var tile = tiles[i];
        if (tile.isUnderMouse(mouseX, mouseY)) {
            if (flippedTiles.length < 2 && !tile.isFaceUp) {
                tile.isFaceUp = true;
                flippedTiles.push(tile);
                if (flippedTiles.length === 2) {
                    numTries++;
                    if (flippedTiles[0].face === flippedTiles[1].face) {
                        flippedTiles[0].isMatch = true;
                        flippedTiles[1].isMatch = true;
                        flippedTiles.length = 0;
                        numMatches++;
                    }
                    delayStartFC = frameCount;
                }
            } 
            loop();
        }
    }
};

draw = function() {
    if (currentScene === 2) {
        initialCount = 1;
    }
    var score = frameCount;
    var finalScore = score - initialCount;
    background(255, 255, 255);
    if (delayStartFC && (frameCount - delayStartFC) > 30) {
        for (var i = 0; i < tiles.length; i++) {
            var tile = tiles[i];
            if (!tile.isMatch) {
                tile.isFaceUp = false;
            }
        }
        flippedTiles = [];
        delayStartFC = null;
        noLoop();
    }
    
    secondScene(tiles);
    
    if (numMatches === tiles.length/2) {
        background(255, 255, 255);
        fill(0, 0, 0);
        textSize(20);
        text("You found them all in " + numTries + " tries!", 20, 100);
        noStroke();
        fill(255, 0, 0);
        restartButton.draw();
        mouseClicked = function() {
            restartButton.mousePressed(restartButton);
        };
        fill(255, 255, 255);
        text("Click to \n restart!", restartButton.x + 50, restartButton.y + 50);
    }
    
    
};
firstScene(playButton);
noLoop();
