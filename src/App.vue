<script setup lang="ts">
import { onMounted, ref } from 'vue';


// `sin(  i * 120) *
// sin(  i  * 4)

// Math.sin(  (i+1000) / sr * 2 * Math.PI * 120) *
// Math.sin( (i+2000) / sr * 2 * Math.PI * (i/2) )`


// square(  i * 120) *
// sin(i* 4) 

const input = ref(`
(
r(i)>0.5?

square(  i * 120) *
sin(i* 4) 


:

sawtooth(  i * 240) *
sawtooth(i* 24) 

)*
square(i*32)
`);

const loop = ref(false);

const visc = ref<HTMLCanvasElement>();

let defs = new Map<string, number|((a:any)=>number)>([
  //convert sin and cos to revolutions/ Hz
  ["r",(i)=>i/sampleRate.value],
  ["sin", (i)=>Math.sin(i*Math.PI*2 /sampleRate.value)],
  ["cos", (i)=>Math.cos(i*Math.PI*2 /sampleRate.value)],
  ["square", (i)=>(i/sampleRate.value)%2 < 1 ? -1 : 1],
  ["sawtooth", (i)=>((i/sampleRate.value)%1 *2 -1) ],
  ["PI", Math.PI],
  ["pi", Math.PI],
  ["TAU", Math.PI*2],
  ["tau", Math.PI*2],
  ["rand", Math.random]
]);


const sampleRate = ref(NaN)
const period = ref(1)

const audioCtx = new AudioContext();

sampleRate.value = audioCtx.sampleRate

let buffer = audioCtx.createBuffer(2, audioCtx.sampleRate*period.value, audioCtx.sampleRate);
function resetBuffer() {
  buffer = audioCtx.createBuffer(2, audioCtx.sampleRate*period.value, audioCtx.sampleRate);
}

// Math.sin(  i / sr * 2 * Math.PI * 120)

let source= audioCtx.createBufferSource();

function createFunc(){
  return new Function("i","sr",...Array.from(defs.keys()),`return (${input.value});`)
}

function play() {

  try{ source.stop()}
  catch(e) {}
  drawWave()

  const f = createFunc()

  for(let c = 0; c < buffer.numberOfChannels; c++) {
    let channel = buffer.getChannelData(c);
    for(let i = 0; i < buffer.length; i++) {
      // console.log(i/sampleRate.value)
      channel[i] = f(i,sampleRate.value,...Array.from(defs.values()));
    }
  }
  
   source = audioCtx.createBufferSource();
  source.buffer = buffer;
  source.loop = loop.value;
  source.onended = ()=> source.loop = loop.value
  source.connect(audioCtx.destination);

  source.start();

}

let ctx:CanvasRenderingContext2D;
let ch:number,cw:number;

function drawWave() {
  let f=createFunc()
  console.log("clearing", ctx, cw, ch)
  ctx.clearRect(0, 0, cw, ch);
  ctx.fillStyle = "black";
  ctx.fillRect(0, 0, cw, ch);

  ctx.strokeStyle = "white";
  ctx.lineWidth = 0.5;
  ctx.beginPath();
  ctx.moveTo(0, ch/2);
  for (let i = 0; i < cw; i++) {
    ctx.lineTo(i, ch/2 + f(i*sampleRate.value/cw *period.value, sampleRate.value,...Array.from(defs.values())) * ch/2.3);
  
  }
  ctx.stroke();
  
}

onMounted(() => {
  console.log("mounted")
  let c = visc.value?.getContext("2d")
  if (c && visc.value?.height && visc.value?.width) {
    ctx = c
    
    ch = visc.value?.height;
    cw = visc.value?.width;
  }else{
    window.alert("no canvas")
  }
  
})

</script>

<template>
  <div class="container">
    <div class="inputs">
      <label for="formula">formula</label>
      <textarea class="formula" v-model="input" name="formula" type="textarea" ></textarea>
      <button @click="play">play</button>

      <label for="loop">loop</label>
      <input  type="checkbox" @change="source.loop = loop" v-model="loop" name="loop" id="loop">

      <label for="period">period (s)</label>
      <input type="number" @change="resetBuffer" v-model="period" name="period">
    </div>

    <div class="vis">
      <canvas id="visc" ref="visc">

      </canvas>
    </div>
  </div>

  <br>

  <p>sample rate: {{ sampleRate }}</p>
</template>

<style scoped lang="scss">
.container {
  display:flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  .inputs {
    display: flex;
    flex-direction: column;
    gap: 10px;
    align-items: center;

    width: 50%;
    min-width: 30rem
  }

  .vis {
    width: 50%;
    min-width: 30rem;
    padding: 2rem;

    #visc{
      border: 2px solid white;
    }
  }
}


.formula{
  width: 20rem;
  height: 20rem;
}
</style>
