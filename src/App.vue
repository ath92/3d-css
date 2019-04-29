<template>
  <div :style="appPerspective" 
       id="app" 
       tabindex="0"
       @pointerlockchange="onPointerLockChange"
       @mousemove="updateDirection" 
       @click="requestPointerLock">
    <div :style="worldTransform" class="scene">
      <template v-for="x in 3">
        <cube v-for="y in 3" :position="{ x: x, y: y, z: 0 }" />
      </template>
      <cube :position="{ x: 1, y: 3, z: 2 }" /> 
      <cube v-for="y in 3" :position="{ x: 8, y: y, z: 0 }" />
      <cube :position="{ x: 1, y: 3, z: 6 }" />
      <player :position="{ x: 3, y: 0, z: 3}" />
      <div class="floor">
        <template v-for="(nothing, i) in Array(10)">
          <template v-if="(i) % 3 === 0">
            🧙🌎🌍🧙
          </template>
          <template v-if="(i + 1) % 3 === 0">
            🧙🌏🌎🧙
          </template>
          <template v-if="(i + 2) % 3 === 0">
            🧙🌍🌏🧙
          </template>
        </template>
      </div>
      <h1 class="header" :style="headerStyle">Welcome to CSSLand</h1>
    </div>
    <span class="description">
      Click to look around. WASD to move around.
    </span>
      <!-- <div class="position">
        {{ direction.x }} - {{ direction.y }} - {{ direction.z }}
        <div>{{ position.x }} - {{ position.y }} - {{ position.z }}</div>
        <div> {{ yAngle }}</div>
      </div> -->
  </div>
</template>
<script>
import Cube from './components/Cube.vue';
import Player from './components/Player';
import Vec3 from './Vec3';

const speed = 30;
const jumpSpeed = 30;
const up = new Vec3(0, -1, 0);
const left = forward => forward.rotateAroundAxis(up, 0.5 * Math.PI);
const perspective = 750;

export default {
  components: {
    Cube,
    Player,
  },
  data: () => ({
    direction: new Vec3(0, 0, -1),
    position: new Vec3(0, 0, 0),
    verticalSpeed: 0,
    ticker: 0,
    directionKeys: {
      w: false,
      s: false,
      a: false,
      d: false,
    },
    hasPointerLock: false,
    shooting: false,
  }),
  computed: {
    yAngle() {
      return this.direction.zAngle() * 180 / Math.PI - 180;
    },
    worldTransform() {
      const sign = this.direction.x > 0 ? -1 : 1;
      const rotate = `rotateY(${sign * this.yAngle}deg)`
      const playerHeight = 120;

      return { 
        transform: `translate3d(${-this.position.x}px, ${-this.position.y + playerHeight}px, ${-this.position.z}px) ${rotate}`,
        transformOrigin: `calc(50% + ${this.position.x}px)
                          calc(50% + ${this.position.y}px) 
                          ${this.position.z + perspective}px`,
      };
    },
    appPerspective() {
      return { perspective: `${perspective}px` };
    },
    carpetStep() {
      return Math.round(this.ticker / 30);
    },
    headerStyle() {
      return { transform: `rotateY(${this.ticker/2%360}deg)` };
    }
  },
  methods: {
    tick() {
      if (this.directionKeys.w) {
        this.position = this.position.add(this.direction.multiplyScalar(speed));
      } 
      if(this.directionKeys.s) {
        this.position = this.position.add(this.direction.multiplyScalar(-speed));
      }
      if(this.directionKeys.a) {
        this.position = this.position.add(left(this.direction).multiplyScalar(speed));
      }
      if(this.directionKeys.d) {
        this.position = this.position.add(left(this.direction).multiplyScalar(-speed));
      }
      if (this.verticalSpeed > 0 || this.position.y <= 0) {
        this.position = this.position.add(up.multiplyScalar(this.verticalSpeed));
        this.verticalSpeed -= jumpSpeed / 15;
      }
      this.ticker++;
      requestAnimationFrame(this.tick);
    },
    onKeydown({ key }) {
      if(key === ' ') {
        this.verticalSpeed = jumpSpeed;
      } else if(Object.keys(this.directionKeys).includes(key)){
        this.directionKeys[key] = true;
      }
    },
    onKeyup({ key }) {
      if(Object.keys(this.directionKeys).includes(key)){
        this.directionKeys[key] = false;
      }
    },
    requestPointerLock() {
      this.$el.requestPointerLock();
    },
    onPointerLockChange() {
      this.hasPointerLock = document.pointerLockElement === this.$el;
      if (this.hasPointerLock) {
        window.addEventListener('keyup', this.onKeyup);
        window.addEventListener('keydown', this.onKeydown);
      } else {
        window.removeEventListener('keyup', this.onKeyup);
        window.removeEventListener('keydown', this.onKeydown);
      }
    },
    updateDirection(e) {
      if (!this.hasPointerLock) return;
      this.direction = this.direction.rotateAroundAxis(up, -e.movementX / 100);
    }
  },
  mounted() {
    this.tick();
    document.addEventListener('pointerlockchange', this.onPointerLockChange);
  },
  destroyed() {
    document.removeEventListener('pointerlockchange', this.onPointerLockChange);
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

.scene {
  transform-style: preserve-3d;
}

.position {
  position: relative;
  z-index: 1000;
}

.floor {
  background-color: #ccc;
  border: 10px solid black;
  width: 50vw;
  height: 50vw;
  transform:  rotateX(90deg) translate3d(0, 0, -15vw) scale(4.0);
  font-size: 160px;
  overflow: hidden;
}

.header, .header-shadow {
  position: absolute;
  top: -180px;
  font-size: 100px;
  font-family: 'Comic sans';
  text-shadow: 20px 20px 20px rgba(0,0,0,.4);
}

.description {
  position: absolute;
  top: 20px;
  width: 100%;
  text-align: center;
  left: 0;
}

</style>
