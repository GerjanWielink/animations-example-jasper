# timeout-wrong.html
**This one will crash your web page**
```js
let x = 0;

function moveTile() {
    x = x + 1;
    const tile = document.querySelector("#tile-1");
    tile.style.top = x;
}

// container is 300px
// tile is 30px
// 300 - 30 = 270
if (x < 270) {
    setTimeout(moveTile, 1000)
}
```

`setTimeout` shedules a function to be executed in the future. This does not pause the rest of the execution. So in this case at `t=0` (I'll refer to time passing as `t=x`) we enter the `while(...)` loop. In the first iteration you schedule `moveTile` to execute 1000ms in the future. Then we're still at `t=0` and `x=0` as `moveTile` has not run yet. So we schedule `moveTile` again, still at `t=0` and `x=0`. This will repeat infinitely many times (as many as it can fit in the 1s until `moveTile` will finally run). This 1s is enough to crash the webpage with al these scheduled function calls.

# timeout-less-wrong.html
```js
let x = 0;

function moveTile() {
    const tile = document.querySelector("#tile-1");
    tile.style.top = x;
}

// container is 300px
// tile is 30px
// 300 - 30 = 270
if (x < 270) {
    x = x + 1;
    setTimeout(moveTile, 5000)
}
```

So the problem in the previous one was us scheduling (almost) infinite function calls at `t=0`. To get rid of this problem we could increment `x` in the `while` loop instead of the `moveTile` function. This way at `t=0` we increment `x` by `1` and we schedule `moveTile` to run in 100 seconds. Then still at `t=0` we come into the loop again, we increment `x` again and we schedule `moveTile` again. This will run `300` times at `t=0` so we've scheduled `moveTile` to run `300` times at `t=5000`. This means that all these move calls will still run at the same time. So the tile will be at the bottom but we stil don't have our animation as we want it. 

# timeout-correct.html
```js
let x = 0;

function moveTile() {
    x = x + 1;
    const tile = document.querySelector("#tile-1");
    tile.style.top = x;

    // container is 300px
    // tile is 30px
    // 300 - 30 = 270
    if (x < 270) {
        setTimeout(moveTile, 10)
    }
}

moveTile()
```

So our main problem was that we were scheduling all our calls to run at the same time (`t=0`). Now there would be two solutions to this. Either schedule all our calls at `t=0` but schedule them to run at different times (`10, 20, 30, ...` ms in the future). Or we could schedule it to run once, and only after it has ran schedule the next one. In the example I've gone for the second option as this one is a lot easier. In the first case you could run into some other annoying traps javascript has to offer which I don't want to get in to now. 

So in the example we solve this using recursion (a function calling itself). At `t=0` we make a single call to `moveTile`. `moveTile` move the tile and if the tile is not at the bottom yet it schedules another call to itself `10ms` in the future. When this one runs at `t=10` it will again schedule the next call to run at `t=20` etc. Giving us our desired animation! :D

# timeout-sleep.html
```js
(async () => {
    let x = 0;

    // https://stackoverflow.com/questions/951021/what-is-the-javascript-version-of-sleep
    function sleep(ms) { 
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    function moveTile() {
        x = x + 1;
        const tile = document.querySelector("#tile-1");
        tile.style.top = x;
    }

    while(x < 270) {
        moveTile()
        await sleep(5)
    }
})()
```

Another way to write this would be using a self created `sleep` function. I've found this one one stackoverflow so I guess this is probably the one you also tried. This example works but to properly understand why it actually works in Javascript requires some more in depth knowledge of async behaviour and Promises in Javascript. I would not recommend using this for now until you've checked out some of the basics on `Promises`. I'll write up another document when that time comes :p.

# timeout-css.html
```css
.tile {
    width: 30px;
    height: 30px;
    background-color: papayawhip;
    position: absolute;
    top: 0;
    transition: 1s;
    transition-timing-function: ease-in;
}
```

```js
function moveTile() {
    const tile = document.querySelector("#tile-1");
    tile.style.top = 270;
}

// we still use a small timeout as without it the tile would
// actually never have the original top = 0 and we would not get a
// animation. This would not be a problem if it actually triggers on a click
// or something. 
setTimeout(moveTile, 5)
```

The last example uses css to achieve the animation instead of js. We still use js to tell the tile to go to the bottom but we only have to to give it the final value and use css to make it animate there. 

We first explicitely set the div to `top: 0;` to have the proper initial css state. After `5ms` (arbitrary value) we change that to `top: 270` using Javascript. Now the `transition: 1s` tells css that whenever the css of this element changes we want it to transition to that next state in `1s`. 

The `transition-timing-function: ease-in;` defines what the anmiation should actually look like. There are stome standard options and you could make it really complex to adjust the falling animation. You could use this to make it have some more realistic gravity for example. 
