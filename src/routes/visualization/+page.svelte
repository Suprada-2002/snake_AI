<script>
    import { onMount } from "svelte";

    onMount ( async() => {
const cvs = document.getElementById("myCanvas");
const ctx = cvs.getContext("2d");

cvs.width = 400;
cvs.height = 400;

let frames = 0;

let foodEaten = false;

let openSet = [];
let closedSet = [];
let current;
let path = [];

const direction = {
    current : 0,
    idle : 0,
    right : 1,
    down : 2,
    left : 3,
    up : 4
}

document.addEventListener("keydown", function(evt){
    switch(evt.keyCode){
        case 37:
            //move left
            if(direction.current != direction.left && direction.current != direction.right) direction.current = direction.left;
            break;
        case 38:
            //move up
            if(direction.current != direction.up && direction.current != direction.down) direction.current = direction.up;
            break;
        case 39:
            //move right
            if(direction.current != direction.right && direction.current != direction.left) direction.current = direction.right;
            break;
        case 40:
            //move down
            if(direction.current != direction.down && direction.current != direction.up) direction.current = direction.down;
            break;
    }

});

function getDistance(pointX1, pointY1, pointX2, pointY2) {
    let distanceX = pointX2 - pointX1;
    let distanceY = pointY2 - pointY1;

   return Math.sqrt(Math.pow(distanceX,2) + Math.pow(distanceY,2));
}

const grid = {
    position : new Array(20),
    init : function(){
        for(let i = 0; i<20; i++){
            this.position[i] = new Array(20);
        }
        
        for(let i = 0; i<20; i++){
            for(let j = 0; j<20; j++){
                this.position[i][j] = {i : i, j : j, x : j*20 + 10, y : i*20 + 10, g : 0, h : 0, f : 9999999, cameFrom : 0};
            }
        }
    },
    
    draw : function(){
        for(let i = 0; i<20; i++){
            for(let j = 0; j<20; j++){
                ctx.beginPath();
                ctx.strokeStyle = "black";
                ctx.rect(this.position[i][j].x - 10, this.position[i][j].y -10, 20, 20);
                ctx.stroke();
                ctx.closePath();
            }
        }
    },
    getHValue : function(node, goal){
        let diffX = goal.x - node.x;
        let diffY = goal.y - node.y;
        
        return Math.sqrt(Math.pow(diffX,2) + Math.pow(diffY,2));
    },
    addNeighbour : function(neighbour){
        let isNeighbourSnake = false;
        
        // if the neighbour is snake
        
        for(let i=0; i<snake.position.length; i++){
            if(neighbour.x == snake.position[i].x && neighbour.y == snake.position[i].y){
                isNeighbourSnake = true;
                break;
            }
        }
        
        if(!isNeighbourSnake){
            let tempG = current.g + 1;
            let tempH = this.getHValue(neighbour, food);
            let tempF = tempG + tempH;
            if(tempF < neighbour.f){
                neighbour.cameFrom = current;//{i : current.i, j : current.j};
                neighbour.g = tempG;
                neighbour.h = tempH;
                neighbour.f = tempF;
            }
            if(!openSet.includes(neighbour)){
                openSet.push(neighbour);
            }
        }
        
        
    },
    
    aStar : function(){
        let p = Math.floor((snake.position[0].y - 10)/20);
        let q = Math.floor((snake.position[0].x - 10)/20);    
        openSet.push(grid.position[p][q]);
        
        while(openSet.length > 0){
            current = openSet[0];
            openSet.splice(0,1);
            closedSet.push(current);
            
            if(current.x == food.x && current.y == food.y){
                break;
            }
            
            // for all neighbours
            // i, j+1
            if(current.j < 19){
                let neighbour = this.position[current.i][current.j+1];
                if (!closedSet.includes(neighbour)){
                    this.addNeighbour(neighbour);
                }
            }
            if(current.i < 19){
                let neighbour = this.position[current.i + 1][current.j];
                if (!closedSet.includes(neighbour)){
                    this.addNeighbour(neighbour);
                }
            }
            if(current.j > 0){
                let neighbour = this.position[current.i][current.j-1];
                if (!closedSet.includes(neighbour)){
                    this.addNeighbour(neighbour);
                }
            }
            if(current.i > 0){
                let neighbour = this.position[current.i - 1][current.j];
                if (!closedSet.includes(neighbour)){
                    this.addNeighbour(neighbour);
                }
            }
            
            // bring the cell with lowest F value to the index 0 in openSet
            for(let r = openSet.length -1 ; r >0 ; r--){
                if(openSet[r].f < openSet[r-1].f){
                    let temp = openSet[r-1];
                    openSet[r-1] = openSet[r];
                    openSet[r] = temp;
                }
            }
        }
        path.splice(0);
        this.findPath();
    },
    
    findPath : function(){
        let previous = '';
        for(let i = closedSet.length - 1; i>=0; i--){
            
            if(i == closedSet.length - 1 && previous == ''){
                previous = closedSet[i];
                path.push(previous);
            }
            else if(previous.cameFrom.i == closedSet[i].i && previous.cameFrom.j == closedSet[i].j){
                previous = closedSet[i];
                path.push(previous);
            }            
        }
    },
    drawPath : function(){
        ctx.strokeStyle = "blue";
        for(let i=0; i<path.length -1; i++){
            ctx.beginPath();
            ctx.moveTo(path[i].x, path[i].y);
            ctx.lineTo(path[i+1].x, path[i+1].y);
            ctx.stroke();
            ctx.closePath();
        }
    }
}

const food = {
    //x : cvs.width/4 + 10,
    //y : cvs.height/4 + 10,
    x : 10,
    y : 10,
    r : 10,

    draw : function(){
        ctx.beginPath();
        ctx.fillStyle = "red";
        ctx.arc(this.x, this.y, this.r, 0 , 2*Math.PI);
        ctx.fill();
        ctx.closePath();
    },
    
    update : function(){
        if(frames % 6 == 0){
            if(direction.current == direction.right) {

                    this.x += 19;
            }
            if(direction.current == direction.left) {
                this.x -= 19;
            }
            if(direction.current == direction.up) {
                this.y -= 19;
            }
            if(direction.current == direction.down) {
                this.y += 19;
            }
            
            if(this.x < 0 ) this.x = cvs.width - 10;
            if(this.x > cvs.width ) this.x = 10;
            if(this.y < 0 ) this.y = cvs.height - 10;
            if(this.y > cvs.height ) this.y = 10;
        }
    }
}
const snake = {
    radius : 10,
    //position : [{ x : cvs.width/2 + 10, y : cvs.height/2 + 10}],
    
    position : [{x : 110, y : 110}],
    snakeHeadIndex : path.length - 1,//1,
    
    draw : function() {
        ctx.fillStyle = "black";
        for( let i = 0; i< this.position.length; i++){
            let p = this.position[i];
            ctx.beginPath();
            ctx.arc(p.x, p.y, this.radius, 0, 2*Math.PI);
            ctx.fill();
            ctx.closePath();
        }
    },

    update : function() {
        if(frames % 6 == 0){
           /* if(foodEaten == true){

                this.position.push({
                    x : this.position[this.position.length -1].x,
                    y : this.position[this.position.length -1].y                   
                });
                foodEaten = false;
            }*/
            
            
            if(this.position[0].x < 0 ) this.position[0].x = cvs.width - 10;
            if(this.position[0].x > cvs.width ) this.position[0].x = 10;
            if(this.position[0].y < 0 ) this.position[0].y = cvs.height - 10;
            if(this.position[0].y > cvs.height ) this.position[0].y = 10;
            
            
            for( let i = this.position.length -1; i > 0;  i--){
                    if(this.position[0].x == this.position[i].x && this.position[0].y == this.position[i].y && this.position.length > 2) {
                        this.position.splice(1);
                        break;
                    }
                    this.position[i].x = this.position[i-1].x; 
                    this.position[i].y = this.position[i-1].y; 
                }
            
            /*if(direction.current == direction.right) {

                this.position[0].x += 20;
            }
            if(direction.current == direction.left) {
                this.position[0].x -= 20;
            }
            if(direction.current == direction.up) {
                this.position[0].y -= 20;
            }
            if(direction.current == direction.down) {
                this.position[0].y += 20;
            }; */
            
            
            
            // snake ate food
            
            if(getDistance(food.x,food.y,this.position[0].x, this.position[0].y) <= 2*food.r){
                let newFoodIndexI = Math.floor(Math.random() * 20);
                let newFoodIndexJ = Math.floor(Math.random() * 20);
                food.x = grid.position[newFoodIndexI][newFoodIndexJ].x;
                food.y = grid.position[newFoodIndexI][newFoodIndexJ].y;
                //foodEaten = true;
                
                this.position.push({
                    x : this.position[this.position.length -1].x,
                    y : this.position[this.position.length -1].y                   
                });
                
                // create new aStar path
                openSet.splice(0);
                closedSet.splice(0);
                //this.snakeHeadIndex = path.length-1; //1;
                grid.init();
                grid.aStar();
                this.snakeHeadIndex = path.length-1; //1;
            }
            // move snake along the aStar path
            if(this.snakeHeadIndex < 0) this.snakeHeadIndex = path.length - 1;
            this.position[0].x = path[path.length - 2].x; // closedSet[this.snakeHeadIndex].x;
            this.position[0].y = path[path.length - 2].y; //closedSet[this.snakeHeadIndex].y;
            if(this.snakeHeadIndex > 0) this.snakeHeadIndex--; //this.snakeHeadIndex++;
            // create new aStar path
                openSet.splice(0);
                closedSet.splice(0);
                //this.snakeHeadIndex = path.length-1; //1;
                grid.init();
                grid.aStar();
                this.snakeHeadIndex = path.length-1; //1;
        }

    }
}

function main() {

ctx.clearRect(0, 0, cvs.width, cvs.height);
snake.update();
snake.draw();
food.update();
food.draw();
grid.draw();
grid.drawPath();
frames ++;
requestAnimationFrame(main);

}
grid.init();
grid.aStar();
requestAnimationFrame(main);
});
    </script>

<div class="container">
    <h3>Snake Game</h3>
    <canvas id="myCanvas"></canvas>
</div>

<style>
    .container {
        width: 80vw;
        margin: 0 auto;
    }
    canvas{
        display: block;
        border: 1px solid black;
    }
</style>