<template>
  <div :style="transform" class="cube">
    <div class="cube__face cube__face--top"></div>
    <div class="cube__face cube__face--right"></div>
    <div class="cube__face cube__face--bottom"></div>
    <div class="cube__face cube__face--left"></div>
    <div class="cube__face cube__face--front"></div>
    <div class="cube__face cube__face--back"></div>
  </div>
</template>

<script>
import Vec3 from '../Vec3';

export default {
  name: 'HelloWorld',
  props: {
    position: {
      type: Object,
      default: () => new Vec3(0, 0, 0), 
    },
  },
  data: () => ({
  }),
  computed: {
    transform() {
      const { x, y, z } = this.position;
      return { transform: `translate3d(${x * 10}vw, ${(y) * 10}vw, ${z * 10}vw)` };
    },
  },
}
</script>

<style scoped lang="scss">

.cube {
  width: 10vw;
  height: 10vw;
  position: absolute;
  transform-style: preserve-3d;
}

.cube__face {
  backface-visibility: hidden;
  height: 10vw;
  position: absolute;
  transition: .2s ease-out;
  width: 10vw;
  border: 5px solid black;
  &:hover {
    background-color: rgba(0, 0, 255, .6);
    transform: rotateY(0deg);
  }
}

.cube__face--front  { 
  background: rgba(255, 0, 0, 1);
  transform: rotateY(  0deg) translateZ(5vw); 
}
.cube__face--right  { 
  background: rgba(255, 255, 0, 1);
  transform: rotateY( 90deg) translateZ(5vw);
}
.cube__face--back   {
  background: rgba(255, 0, 255, 1);
  transform: rotateY(180deg) translateZ(5vw); 
}
.cube__face--left   { 
  background: rgba(0, 255, 255, 1);
  transform: rotateY(-90deg) translateZ(5vw); 
}
.cube__face--top    { 
  background: rgba(0, 0, 255, 1);
  transform: rotateX( 90deg) translateZ(5vw); 
}
.cube__face--bottom { 
  background: rgba(0, 255, 0, 1);
  transform: rotateX(-90deg) translateZ(5vw); 
}

</style>
