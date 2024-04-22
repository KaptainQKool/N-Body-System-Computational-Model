<script lang="ts">
  import World from "./lib/World.svelte";
  import {World as GameWorld, Ball, GameObject} from "./lib/Existance"
  import type {WorldInfo} from "./lib/Existance"
import { onMount } from "svelte";
  
  let world_settings = {
      time_scale: 1,
      time_scalemin: 0,
      time_scalemax: 1,
      elasticity: 0.5,
      elasticitymin: -1,
      elasticitymax: 2,
      object_elasticity: 0.75,
      object_elasticitymin: -1,
      object_elasticitymax: 2,
      width: 1510,
      height: 670,
      scale: 5, // this means that every 1 pixel is 10000 meters or something idk
      scalemin: 1,
      scalemax: 10,
      // display_scale: 2,
      // display_scalemin: -1,
      // display_scalemax: 5,
      zoom_x: 755,
      zoom_y: 335,
      zoom_percentage: 1,
      alignment: "middle",
      max_steps: 1,
      max_stepsmin: 1,
      max_stepsmax: 20,
      show_path: true,
      gravity: 0,
      gravitymin: -20,
      gravitymax: 10,
      useGravity: false,
      gravConstant: 0.000000000066743 // G = 6.6743 * 10^-11 = 0.000000000066743
      // F = G*M*m / (r^2)
      // F = m*a
      // a = G*M / (r^2)
  }
  let shouldReset = false
  function resetData() {
      shouldReset = true
      location.reload()
  }
  window.onbeforeunload = function(){
      if (!shouldReset) {
          this.localStorage['settings'] = JSON.stringify(world_settings)
      } else {
          this.localStorage['settings'] = null
      }
  };
  onMount(() => {
      let maybe_settings = JSON.parse(window.localStorage['settings'])
      if (maybe_settings) {
          world_settings = maybe_settings
      }
      zoomX = world_settings.zoom_x
      zoomY = world_settings.zoom_y
      showPaths = world_settings.show_path
      zoomPercentage = world_settings.zoom_percentage
 
      window.addEventListener("keydown", (event) => {
          if (event.key === "p") {
              if (GameWorld.paused) {
                  unpauseSim()
              } else {
                  pauseSim()
              }
          }
          // do something
      });
      
  })
  let world_settings_visible = {
      'elasticity': true,
      'elasticitymin': false,
      'elasticitymax': false,
      'object_elasticity': true,
      'object_elasticitymin': false,
      'object_elasticitymax': false,
      'gravity': false, 
      'gravitymin': false,
      'gravitymax': false,  
      'width': true,
      'height': true,
      'scale': true,
      'scalemin': false,
      'scalemax': false,
      'display_scale': true,
      'display_scalemin': false,
      'display_scalemax': false,
      'time_scale':true,
      'time_scalemin':false,
      'time_scalemax':false,
      'max_steps': true,
      'max_stepsmin': false,
      'max_stepsmax': false


  }
  //@ts-ignore
  let visible_world_settings = Object.keys(world_settings).filter((val) => world_settings_visible[val]) //goofy ah typescript!?
  let derivative: number[] = []
  let ball_launch_settings: number[] = [-1, world_settings.gravity]
  let ball_launch_angle: number[] = [-1, 90]
  let ball_launch_mass: number[] = [-1, 10]
  let angle = Math.PI/4
  let xpos = 0
  let is_x_random = false
  let ypos = 670
  let is_y_random = false
  let inverted_ypos = 0
  let launch_mass = 10
  let is_mass_random = false
  let rand_mass_min = 1
  let rand_mass_max = 100
  let balls_from = 0
  let showPaths = world_settings.show_path
  let balls_to = 90
  let balls_amount = 10
  let table: number[][] = []
  // let alignment = "middle"
  let zoomX = world_settings.zoom_x
  let zoomY = world_settings.zoom_y
  let selectedObject: GameObject|null = null
  let zoomPercentage = world_settings.zoom_percentage
  let clearAllObjects = () => {
      GameWorld.clearAll()
      
  }
  // derivative has all numbers 0 through 100
  for (let i=0;i<1;i++) {
      derivative[i] = i
      if (ball_launch_settings[i] == null || ball_launch_settings[i] == -1) {
          ball_launch_settings[i] = 0
      }
      if (ball_launch_angle[i] == null || ball_launch_angle[i] == -1) {
          ball_launch_angle[i] = 0
      }
      if (ball_launch_mass[i] == null || ball_launch_mass[i] == -1) {
          ball_launch_mass[i] = 0
      }
     
  }
 
  function launchBall() {
      
      ball_launch_settings[1] = world_settings.gravity
      let [ball_launch_settings_x, ball_launch_settings_y] = GameWorld.getComponentsOfKinematics(ball_launch_settings, ball_launch_angle)
      let x_init = xpos
      let y_init = ypos
      if(is_x_random) {
        x_init = Math.random() * world_settings.width
      }
      if(is_y_random) {
        y_init = Math.random() * world_settings.height
      }
      ball_launch_settings_x.unshift(x_init)
      ball_launch_settings_y.unshift(y_init)
      
      let timescale = world_settings.time_scale
      world_settings.time_scale = 0
      let ball = new Ball(ball_launch_settings_x, ball_launch_settings_y, "black", world_settings, 10)
      if(is_mass_random) {
        ball.setMass((Math.random()*(rand_mass_max-rand_mass_min) + rand_mass_min) * 100000000000)
      } else {
        ball.setMass(launch_mass*100000000000)
      }
      GameWorld.addToWorld(ball)
      world_settings.time_scale = timescale
  }
  function fanwave() {
      let total_angle = (balls_to - balls_from) * (Math.PI/180)
      let [ball_launch_settings_x, ball_launch_settings_y] = GameWorld.getComponentsOfKinematics(ball_launch_settings, ball_launch_angle)
      
      ball_launch_settings_x.shift()  
      ball_launch_settings_y.shift()
      //@ts-ignore
      let velocity = parseFloat(prompt("velocity? "))
      for (let i = 0; i < balls_amount; i++) {
          let obj = Ball.createWithVector(xpos, ypos, velocity, balls_from + (total_angle/(balls_amount))*i, world_settings, ball_launch_settings_x, ball_launch_settings_y)
          GameWorld.addToWorld(obj)
      }
  }

  let kinematics_lang = ["Velocity", "Acceleration", "Jerk", "Snap", "Crackle", "Pop", "Lock", "Drop"]
  function getNameForDerivative(num: number) {
    if (num >= kinematics_lang_position.length-1) {
          return num + "th"
      }
      return kinematics_lang[num]
  }

  let kinematics_lang_position = ["Position", "Velocity", "Acceleration", "Jerk", "Snap", "Crackle", "Pop", "Lock", "Drop"]
  function getNameForDerivativeP(num: number) {
      if (num > kinematics_lang_position.length-1) {
          return num + "th"
      }
      return kinematics_lang_position[num]
  }

  function onDerivativeChange(e:any) {
      //@ts-ignore
      
      let value = 0
      if (e.target.value != '' && !isNaN(e.target.value)) {
        
          value = parseFloat(e.target.value)

          let id = parseInt(e.target.id)
         
          ball_launch_settings[id] = value
      }
  }
  function onCanvasClick(x:number, y:number) {
      [xpos, ypos] = GameWorld.zoomCoordsToRealCoords(x, y) //makes it so u can click the canvas and the point u click becomes the ball's launch point
      
  }
  function updateProperties(obj: GameObject){
      selectedObject = obj
  }
  function updateTable(table_data: number[][]) {
      table = table_data
  }
  function controlsChanged() {
      //@ts-ignore
      zoomX = parseFloat(zoomX)
      //@ts-ignore why is typescript stupid
      zoomY = parseFloat(zoomY)
      //@ts-ignore
      zoomPercentage = parseFloat(zoomPercentage)
      console.log(showPaths)
      
      world_settings = {...world_settings, zoom_percentage: zoomPercentage, zoom_x: zoomX, zoom_y:zoomY, show_path: showPaths}
      console.log(world_settings)
  }
  function setVelocity(distance: number, angle: number) {
      ball_launch_angle[0] = angle
      ball_launch_settings[0] = distance
      ball_launch_angle = [...ball_launch_angle]
      ball_launch_settings = [...ball_launch_settings]

  }
  function roundTo(number:number, decimal_places:number) {
      var factorOfTen = Math.pow(10,decimal_places);
      return Math.round(number * factorOfTen)/factorOfTen;
      
  }

  function onDerivativeAngleChange(e:any) {
    
      let value = 0
      if (e.target.value != '' && !isNaN(e.target.value)) {
          value = parseFloat(e.target.value)
    
          let id = parseInt(e.target.id)
          ball_launch_angle[id] = value
      }
  }
  function onWorldSettingsChanged(e: any) {
      //@ts-ignore
    
      let value = 0
      if (e.target.value != '' && !isNaN(e.target.value)) {
          world_settings = {...world_settings, [e.target.id]: parseFloat(e.target.value)}

      }
      
  }
  function pauseSim() {
      GameWorld.pause()
  }
  function unpauseSim() {
      GameWorld.unpause()
  }
  
  let getWorldSettings = () => {
  
      return world_settings
  }
  function getTime(selectedObject: GameObject) {
      //@ts-ignore
      return selectedObject.world.time
  }
</script>


  <div id="interface">
      <div id="world">
          <World framerate="60" height="670" width="1510" updateTable={updateTable} {updateProperties} getWorldSettings={getWorldSettings} canvasclick={onCanvasClick} setvelo={setVelocity}/>
      </div>
      <div id="columns">
          <div id="world_settings">
              {#each visible_world_settings as world}
                  <p>{world.toUpperCase()}</p>
                  <input type="number" step="0.01" id={world} bind:value={world_settings[world]} on:change={onWorldSettingsChanged}>
                  <input type="range" min={world_settings[world+"min"]} max={world_settings[world+"max"]} step="0.01" id={world} bind:value={world_settings[world]} on:change={onWorldSettingsChanged}>
                  <br/>
              {/each}
              <!-- <input type="checkbox" bind:checked={useGravity} on:change={controlsChanged} /> -->
          </div>

          <div id="topright">
              <h1>Michael and David's Amazing Spectacular N-Body Problem Simulator</h1><h3>(it's better than ani's)</h3>

              <div id="datap">
                      <table>
                          <tr>
                              <th>Time</th>
                              <th>X Position</th>
                              <th>Y Position</th>
                              <th>X Velocity</th>
                              <th>Y Velocity</th>
                              <th>X Acceleration</th>
                              <th>Y Acceleration</th>
                          </tr>
                          {#each table as point, i}
                              <tr>
                                  {#each point as p, i2}
                                  <th>{roundTo(p, 5)}</th>
                                  {/each}
                              </tr>
                          {/each}
                      </table>
              </div>
          </div>
          <div id="ball">
            <button id="launch" on:click={launchBall}>Launch Ball</button>
            <div id="mass_input">
              
            
                    <label for="launchMass">Mass:</label>
                    <input type="number" placeholder="10" name="launchMass" id="launchMass" bind:value={launch_mass}>
                    <label for="launchMass">Random?</label>
                    <input type="checkbox" bind:checked={is_mass_random} on:change={controlsChanged} />
             
            </div>
            <div id="mass_randomness">
              
            
                <label for="randMin">Random mass from</label>
                <input type="number" placeholder="1" name="randMin" id="randMin" bind:value={rand_mass_min}>
                <label for="randMax">to</label>
                <input type="number" placeholder="100" name="randMax" id="randMax" bind:value={rand_mass_max}>
             
            </div>
            <div id="important_settings">
              
            
                    <label for="posX">X Position:</label>
                    <input type="number" placeholder="0" name="posX" id="posX" bind:value={xpos}>
                    <label for="launchMass">Random?</label>
                    <input type="checkbox" bind:checked={is_x_random} on:change={controlsChanged} />
                    <label for="posY">Y Position:</label>
                    <input type="number" placeholder="670" name="posY" id="posY" bind:value={inverted_ypos} on:change={() => {ypos = 670 - inverted_ypos}}>
                    <label for="launchMass">Random?</label>
                    <input type="checkbox" bind:checked={is_y_random} on:change={controlsChanged} />
             
            </div>
            {#each derivative as d} 
                <div id="single">
                    <label for="something" id="dumblabel">{getNameForDerivative(d)}:</label>
                    <input type="number" placeholder="0" name={getNameForDerivative(d)} id={d+""} bind:value={ball_launch_settings[d]} on:change={onDerivativeChange}>
                    <label for="something" id="dumblabel">Angle:</label>
                    <input type="angle" placeholder="45" name={getNameForDerivative(d)} id={d+""} bind:value={ball_launch_angle[d]} on:change={onDerivativeAngleChange}>
                </div>
            {/each}
        </div>  
        <div id="ball_controls">
            <button on:click={clearAllObjects}>Clear all Objects</button>
            <button on:click={resetData}>Reset All Data</button>
            <button on:click={pauseSim}>Pause</button>
            <button on:click={unpauseSim}>Unpause</button>
        </div>
      </div>
      <div id="controls">
          
          <div id="rightie">
              <div id="actualcontrols">
                  <div id="zoombox">

                      <h3>Controls</h3>
                      <p>Canvas is 1510 by 670 pixels</p>
                      <label>Zoom X</label>
                      <input bind:value={zoomX} on:change={controlsChanged} />
                      <label>Zoom Y</label>
                      <input bind:value={zoomY} on:change={controlsChanged}/>
                      <label>Zoom Amount</label>
                      <input bind:value={zoomPercentage} on:change={controlsChanged}/>
                      <button on:click={() => {world_settings.alignment = "left"}}>Left-Aligned Zoom</button>
                      <button on:click={() => {world_settings.alignment = "middle"}}>Middle Zoom</button>
                      <hr/>
                      <label>Show Paths?</label>
                      <input type="checkbox" bind:checked={showPaths} on:change={controlsChanged} />
                  </div>
                  <div id="properties">
                      {#if selectedObject != undefined && selectedObject.world != undefined}
                      <!--TS ur crazy, time definitely exists on that type idk what to say-->
                          <p>{roundTo(getTime(selectedObject), 2)} seconds</p>
                      {/if}
                  </div>
              </div>
             
          </div>
      </div>
      <div id="toys">
        <div id="fanwave_container">
            <h3>Toys</h3>
            <button on:click={fanwave}>Fan Wave</button>
            <br/>
            <label>From Angle</label>
            <input bind:value={balls_from} placeholder="90"/>
            <label>To Angle</label>
            <input bind:value={balls_to} placeholder="90"/>
            <label>Ball Amount</label>
            <input bind:value={balls_amount} placeholder="10"/>

        </div>
    </div>
  </div>


<style scoped>
  
  #interface{
      overflow-x:hidden;
      display: flex;
      flex-direction: column;
      height: 98vh;
      width: 100%;
  }
  #topright {
      min-height: 100%;
      max-height: 100%;
      overflow-y: scroll;
      width: 25%;

      
  }
  #datap {
      overflow-y: scroll;
      max-height:50vh;
  }
  
  #datap table{
      height: 100%;
      overflow-y: scroll;
  }
 #properties {
  border-left: 1px solid black;
  padding: 10px;
 }
  #prop_kine {
      display: flex;
      flex-direction: row;
      width: 600px;
      border: 0px solid black;
      
  }
  #x_stuff, #y_stuff {
      display: flex;
      width: 30%;
      flex-direction: column;
      border: 0px solid black;
      border-right: 1px solid black;
     
  }

  #dumblabel {
      width: 10px;
  }
  #toys {
      /* padding: 10px; */
      width: 10%;
  }
  #world_settings {
      border: 1px solid black;
      /* min-width: 10px; */
      /* padding: 2ch; */
      width: 15%;
  }
  #columns {
      display: flex;
      flex-direction: row;
      
      
  }
  #ball_controls {
      display: flex;
      flex-direction: column;
      
      
  }
  #controls {
      display: flex;
      flex-direction: row;
      /* min-height: 10px; */
      max-height: fit-content;
      border: 1px solid black;
  }
  #ball {
      min-width: 670px;
      border: 1px solid black;
      display: flex;
      flex-direction: column;
  }
  #important_settings {
  
      padding: 20px;
      margin-bottom: 20px;
  }
  #zoombox {
      display: flex;
      flex-direction: column;
  }
  #single {
      display: flex;
      flex-direction: row;
      justify-content: space-around;
  }
  #actualcontrols {
      display: flex;
      flex-direction: row;
      padding: 10px;
      width: 40%;
  }
  #toys {
      display: flex;
      border: 1px solid black;
      /* max-width: 150px; */
      /* justify-content: center; */
      flex-direction: column;
  }
  #toys input {
      width: 90%;
      
  }
 
</style>