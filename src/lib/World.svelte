<script lang="ts">
    // import {World} from "$lib/World"
	import { onMount } from "svelte";
    import {World, GameObject, Ball} from "./Existance"
    import type {WorldInfo} from "./Existance"

 export let framerate: string 
 export let width: string 
 export let height: string 

 export let canvasclick: (x:number, y:number) => void
 export let setvelo: (distance:number, angle:number) => void
 export let getWorldSettings: () => WorldInfo
 export let updateProperties: (obj: GameObject) => void
 export let updateTable: (table_data: number[][]) => void
console.log(`SETTINGS:
Framerate: ${framerate}
Width: ${width}
Height: ${height}
`)

let pastMousePosX: number
let pastMousePosY: number
function putMouseDown(e: any) {
    pastMousePosX = e.offsetX
    pastMousePosY = e.offsetY
}
function putMouseUp(e: any) {
    const dx = (pastMousePosX - e.offsetX) * -1
    const dy = (pastMousePosY - e.offsetY)
    const distance = Math.sqrt((Math.pow(dx, 2)) + (Math.pow(dy, 2)))
    // console.log(distance)
    if (distance > 15) {
        let angle = Math.atan2(dy, dx)
        setvelo((1/getWorldSettings().scale) * distance, (angle * (180/Math.PI)))
    }
    canvasclick(pastMousePosX, pastMousePosY)
}

function onCanvasLoad() {
    console.log("Canvas loaded!")
    //@ts-ignore
    let canvas: HTMLCanvasElement = document.getElementById("world-canvas")
    let c = canvas.getContext("2d")
    if (c == null) {
        return
    }
    canvas.width = parseInt(width)
    canvas.height = parseInt(height)
    console.log(width)
   
   
    // let obj = Ball.createWithVector(0, canvas.height, 100, Math.PI/4, world_info)
    // World.addToWorld(obj)
    
    

    createLoop()
}
function createLoop() {
     //@ts-ignore
    let canvas: HTMLCanvasElement = document.getElementById("world-canvas")
    //@ts-ignore
    let ctx: CanvasRenderingContext2D = canvas.getContext("2d")
    
   
    if (ctx == null) {
        return
    }
    let lastTime = performance.now()
    let refresh = () => {
        // console.log("Displaying")
        let world_info = getWorldSettings()
        let nowTime = performance.now()
        let delta = (nowTime-lastTime) / 1000
        delta = delta * world_info.time_scale
        // console.log(delta)
        lastTime = nowTime
        World.update(delta, world_info)
        updateProperties(GameObject.lastCreated)
        updateTable(World.table)
        ctx.fillStyle = "#e8edea"
        ctx.fillRect(0, 0, canvas.width, canvas.height)
     
        let [x_pos, y_pos] = World.realCoordsToZoomCoords(0, 0)
        let [x2_pos, y2_pos] = World.realCoordsToZoomCoords(world_info.width, world_info.height)

        ctx.fillStyle = "#ffffff"
        ctx.fillRect(x_pos, y_pos, x2_pos-x_pos, y2_pos-y_pos)
      
        // ctx.fillStyle = "yellow"
        // ctx.fillRect(0, canvas.height-10, 45, 1)
        //@ts-ignore
     
        World.display(ctx)
        if (World.paused) {
            
            ctx.font="35px Arial";
            ctx.textAlign="center";
            ctx.textBaseline="middle";
            ctx.fillText("Paused, Press P to toggle", canvas.width/2, canvas.height/2);
            
        }
        setTimeout(() => {
            refresh()
        }, Math.round((1/parseInt(framerate)) * 1000))
        
    }
    refresh()
   
    
}
onMount(()=> {
    onCanvasLoad()
})
</script>

<div id="world-container">
    <canvas id="world-canvas" width="0" height="0" on:mousedown={putMouseDown} on:mouseup={putMouseUp}></canvas>
</div>

<style>
    #world-container {
        border: 1px solid black;
        display: flex;
        width: min-content;
        height: min-content;
    }
</style>