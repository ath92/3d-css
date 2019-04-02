<template>
  <div :style="appPerspective" 
       id="app" 
       tabindex="0"
       @click="requestPointerLock" 
       @mousemove="updateDirection" 
       @keydown="onKeydown"
       @keyup="onKeyup">
    <div :style="translate" class="scene">
      <div :style="rotate" class="content">
        <template v-for="x in 3">
          <cube v-for="y in 3" :position="{ x: x, y: y, z: 0 }" />
          <cube :position="{ x: 1, y: 3, z: 1 }" />
          <cube :position="{ x: 1, y: 3, z: 2 }" />
        </template>
      </div>
    </div>
    <div class="position">
      {{ direction.x }} - {{ direction.y }} - {{ direction.z }}
      <div>{{ position.x }} - {{ position.y }} - {{ position.z }}</div>
      <div> {{ yAngle }}</div>
    </div>
  </div>
</template>

<script>
import Cube from './components/Cube.vue';
import Vec3 from './Vec3';

const speed = 30;
const up = new Vec3(0, -1, 0);
const left = forward => forward.rotateAroundAxis(up, 0.5 * Math.PI);
const perspective = 500;

export default {
  components: {
    Cube
  },
  data: () => ({
    direction: new Vec3(0, 0, -1),
    position: new Vec3(0, 0, 0),
    directionKeys: {
      front: false,
      back: false,
      left: false,
      right: false,
      jump: false,
    }
  }),
  computed: {
    yAngle() {
      return this.direction.zAngle() * 180 / Math.PI - 180;
    },
    rotate() {
      const sign = this.direction.x > 0 ? -1 : 1;
      const rotate = `rotateY(${sign * this.yAngle}deg)`

      return { 
        transform: rotate,
        transformOrigin: `calc(50% + ${this.position.x}px)
                          calc(50% + ${this.position.y}px) 
                          calc(${this.position.z}px + ${perspective}px)`,
      };
    },
    translate() {
      return { transform: `translate3d(${-this.position.x}px, ${-this.position.y}px, ${-this.position.z}px)` };
    },
    appPerspective() {
      return { perspective: `${perspective}px` };
    }
  },
  methods: {
    tick() {
      requestAnimationFrame(this.tick);
      if (this.directionKeys.jump) {
        this.position = this.position.add(new Vec3(0, -30, 0));
        this.directionKeys.jump = false;
      }
      if (this.directionKeys.front) {
        this.position = this.position.add(this.direction.multiplyScalar(speed));
      } 
      if(this.directionKeys.back) {
        this.position = this.position.add(this.direction.multiplyScalar(-speed));
      }
      if(this.directionKeys.left) {
        this.position = this.position.add(left(this.direction).multiplyScalar(speed));
      }
      if(this.directionKeys.right) {
        this.position = this.position.add(left(this.direction).multiplyScalar(-speed));
      }
      if (this.position.y < 0) {
        this.position = this.position.add(new Vec3(0, 0.1, 0));
      }
    },
    onKeydown({ key }) {
      if (key === 'w') {
        this.directionKeys.front = true;
      } else if(key === 's') {
        this.directionKeys.back = true;
      } else if(key === 'a') {
        this.directionKeys.left = true;
      } else if(key === 'd') {
        this.directionKeys.right = true;
      } else if(key === ' ') {
        console.log('jhbasdhbasjhdba');
        this.directionKeys.jump = true;
      }
    },
    onKeyup({ key }) {
      if (key === 'w') {
        this.directionKeys.front = false;
      } else if(key === 's') {
        this.directionKeys.back = false;
      } else if(key === 'a') {
        this.directionKeys.left = false;
      } else if(key === 'd') {
        this.directionKeys.right = false;
      }
    },
    requestPointerLock() {
      this.$el.requestPointerLock();
    },
    updateDirection(e) {
      this.direction = this.direction.rotateAroundAxis(up, -e.movementX / 100);
    }
  },
  mounted() {
    this.tick();
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  overflow: hidden;
  width: 100vw;
  height: 100vh;
}

.scene, .content {
  transform-style: preserve-3d;
}

.position {
  position: relative;
  z-index: 1000;
}
</style>
