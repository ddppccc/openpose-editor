<template>
  <div>
    <h1>人物骨骼图调整</h1>
    <div class="controls">
      <input type="file" @change="handleImageUpload" accept="image/*">
      <button @click="savePose">保存姿势图</button>
    </div>
    <canvas ref="canvas" width="800" height="800"></canvas>
  </div>
</template>

<script>
export default {
  data() {
    return {
      canvas: null,
      ctx: null,
      image: null,
      jointRadius: 10,
      lineWidth: 12,
      joints: [
        { x: 400, y: 100, color: '#0100ff' }, // 头部中心
        { x: 380, y: 90, color: '#aa00ff' },  // 左眼
        { x: 420, y: 90, color: '#fe00fd' },  // 右眼
        { x: 360, y: 120, color: '#ff00aa' }, // 左耳
        { x: 440, y: 120, color: '#ff0055' }, // 右耳
        { x: 400, y: 200, color: '#ff0000' }, // 颈部
        { x: 350, y: 200, color: '#ffaa00' }, // 左肩
        { x: 450, y: 200, color: '#aaff00' }, // 右肩
        { x: 350, y: 350, color: '#ffff00' }, // 左肘
        { x: 450, y: 350, color: '#55ff00' }, // 右肘
        { x: 350, y: 450, color: '#ff5500' }, // 左手腕
        { x: 450, y: 450, color: '#00ff00' }, // 右手腕
        { x: 370, y: 400, color: '#00ff55' }, // 躯干左部
        { x: 430, y: 400, color: '#00aaff' }, // 躯干右部
        { x: 370, y: 500, color: '#00ffaa' }, // 左髋
        { x: 430, y: 500, color: '#0055ff' }, // 右髋
        { x: 370, y: 600, color: '#00ffff' }, // 左膝
        { x: 430, y: 600, color: '#003cb3' }  // 右膝
      ],
      connections: [
        { start: 0, end: 1, color: '#3c00b3' },  // 头部到左眼
        { start: 0, end: 2, color: '#b300b3' },  // 头部到右眼
        { start: 1, end: 3, color: '#7700b3' },  // 左眼到左耳
        { start: 2, end: 4, color: '#b30077' },  // 右眼到右耳
        { start: 0, end: 5, color: '#0000b3' },  // 头部到颈部
        { start: 5, end: 6, color: '#b30000' },  // 颈部到左肩
        { start: 5, end: 7, color: '#b33c00' },  // 颈部到右肩
        { start: 6, end: 8, color: '#b37700' },  // 左肩到左肘
        { start: 7, end: 9, color: '#77b300' },  // 右肩到右肘
        { start: 8, end: 10, color: '#b3b300' }, // 左肘到左手腕
        { start: 9, end: 11, color: '#3cb300' }, // 右肘到右手腕
        { start: 5, end: 12, color: '#00b300' }, // 颈部到躯干左部
        { start: 5, end: 13, color: '#00b3b3' }, // 颈部到躯干右部
        { start: 12, end: 14, color: '#00b33c' }, // 躯干左部到左髋
        { start: 13, end: 15, color: '#0077b3' }, // 躯干右部到右髋
        { start: 14, end: 16, color: '#00b377' }, // 左髋到左膝
        { start: 15, end: 17, color: '#003cb3' }  // 右髋到右膝
      ],
      selectedJoint: null
    };
  },
  methods: {
    setupCanvas() {
      this.canvas = this.$refs.canvas;
      this.ctx = this.canvas.getContext('2d');
      this.drawSkeleton();
    },
    drawSkeleton(includeImage = true) {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

      if (includeImage && this.image) {
        this.ctx.drawImage(this.image, 0, 0, this.canvas.width, this.canvas.height);
      }

      this.connections.forEach(conn => {
        const startJoint = this.joints[conn.start];
        const endJoint = this.joints[conn.end];
        this.ctx.beginPath();
        this.ctx.moveTo(startJoint.x, startJoint.y);
        this.ctx.lineTo(endJoint.x, endJoint.y);
        this.ctx.strokeStyle = conn.color;
        this.ctx.lineWidth = this.lineWidth;
        this.ctx.stroke();
      });

      this.joints.forEach(joint => {
        this.ctx.beginPath();
        this.ctx.arc(joint.x, joint.y, this.jointRadius, 0, Math.PI * 2);
        this.ctx.fillStyle = joint.color;
        this.ctx.fill();
        this.ctx.closePath();
      });
    },
    handleImageUpload(event) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.image = new Image();
        this.image.onload = () => {
          this.drawSkeleton();
        };
        this.image.src = e.target.result;
      };
      reader.readAsDataURL(event.target.files[0]);
    },
    savePose() {
      this.drawSkeleton(false);
      const dataURL = this.canvas.toDataURL('image/png');
      const link = document.createElement('a');
      link.href = dataURL;
      link.download = 'skeleton_pose.png';
      link.click();
      this.drawSkeleton(true);
    },
    onMouseDown(event) {
      const { offsetX, offsetY } = event;
      this.joints.forEach(joint => {
        const dist = Math.hypot(joint.x - offsetX, joint.y - offsetY);
        if (dist < this.jointRadius) {
          this.selectedJoint = joint;
        }
      });
    },
    onMouseMove(event) {
      if (this.selectedJoint) {
        this.selectedJoint.x = event.offsetX;
        this.selectedJoint.y = event.offsetY;
        this.drawSkeleton();
      }
    },
    onMouseUp() {
      this.selectedJoint = null;
    }
  },
  mounted() {
    this.setupCanvas();
    this.$refs.canvas.addEventListener('mousedown', this.onMouseDown);
    this.$refs.canvas.addEventListener('mousemove', this.onMouseMove);
    this.$refs.canvas.addEventListener('mouseup', this.onMouseUp);
  },
  beforeUnmount() {
    this.$refs.canvas.removeEventListener('mousedown', this.onMouseDown);
    this.$refs.canvas.removeEventListener('mousemove', this.onMouseMove);
    this.$refs.canvas.removeEventListener('mouseup', this.onMouseUp);
  }
};
</script>

<style scoped>
body {
  text-align: center;
  background-color: #333;
  color: white;
}
canvas {
  background-color: black;
}
.controls {
  margin-bottom: 10px;
}
</style>
