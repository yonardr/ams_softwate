<template>
  <div id="app">
    <canvas id="canvas" width="800" height="600"></canvas>
  </div>
</template>

<script>
import {onMounted} from 'vue';
import {Canvas, Line, Triangle, Circle} from 'fabric';

export default {
  name: 'App',
  setup() {
    onMounted(() => {
      const canvas = new Canvas('canvas');

      // Create the scale
      const scaleCenter = new Circle({
        left: 385,
        top: 90,
        radius: 15,
        fill: 'gray',
        selectable: false,
      });

      const scaleLine = new Line([400, 100, 400, 400], {
        stroke: 'gray',
        selectable: false,
        strokeWidth: 5,
      });

      const scaleArm = new Line([250, 200, 550, 200], {
        stroke: 'gray',
        selectable: false,
        strokeWidth: 5,
      });

      canvas.add(scaleCenter, scaleLine, scaleArm);

      // Create the scale pans
      const leftPan = new Triangle({
        left: 204,
        top: 200,
        width: 100,
        height: 120,
        fill: 'transparent',
        stroke: 'gray',
        selectable: false,
      });
      const lineLeftPan = new Line([306, 320, 203, 320], {
        stroke: 'gray',
        strokeWidth: 5,
        selectable: false,
      });

      const rightPan = new Triangle({
        left: 494,
        top: 200,
        width: 100,
        height: 120,
        fill: 'transparent',
        stroke: 'gray',
        selectable: false,
      });
      const lineRightPan = new Line([596, 320, 493, 320], {
        stroke: 'gray',
        strokeWidth: 5,
        selectable: false,
      });

      canvas.add(leftPan, rightPan, lineLeftPan, lineRightPan);

      // Create circles
      const leftCircle = new Circle({
        left: 230,
        top: 280,
        radius: 20,
        fill: 'red',
        lockMovementX: true,
        lockMovementY: true,
        lockScalingX: false,
        lockScalingY: false
      });

      const rightCircle = new Circle({
        left: 530,
        top: 280,
        radius: 20,
        fill: 'cyan',
        lockMovementX: true,
        lockMovementY: true,
        lockScalingX: false,
        lockScalingY: false
      });

      canvas.add(leftCircle, rightCircle);

      // Recalculate the balance when circles are modified
      const recalculateBalance = () => {
        const leftArea = Math.PI * Math.pow(leftCircle.radius * leftCircle.scaleX, 2);
        const rightArea = Math.PI * Math.pow(rightCircle.radius * rightCircle.scaleX, 2);


        if (leftArea > rightArea) {
          scaleArm.set({angle: -10, top: 220});
        } else if (rightArea > leftArea) {
          scaleArm.set({angle: 10, top: 170});
        } else {
          scaleArm.set({angle: 0, top: 195});
        }


        let leftPanTop = 200;
        let rightPanTop = 200;
        if (leftArea > rightArea) {
          leftPanTop += 23;
          rightPanTop -= 28;
        } else if (rightArea > leftArea) {
          leftPanTop -= 25;
          rightPanTop += 25;
        }
        leftPan.set({top: leftPanTop});
        rightPan.set({top: rightPanTop});
        lineLeftPan.set({top:leftPanTop +117});
        lineRightPan.set({top: rightPanTop+117})


        const minScale = 0.5; // Минимальный масштаб 50%
        const maxScale = 1.5;   // Максимальный масштаб 200%

        leftCircle.scaleX = Math.max(minScale, Math.min(maxScale, leftCircle.scaleX));
        leftCircle.scaleY = Math.max(minScale, Math.min(maxScale, leftCircle.scaleY));

        rightCircle.scaleX = Math.max(minScale, Math.min(maxScale, rightCircle.scaleX));
        rightCircle.scaleY = Math.max(minScale, Math.min(maxScale, rightCircle.scaleY));


        const leftPanCenter = {
          left: leftPan.left + leftPan.width / 2,
          top: leftPan.top + leftPan.height / 2,
        };

        const rightPanCenter = {
          left: rightPan.left + rightPan.width / 2,
          top: rightPan.top + rightPan.height / 2,
        };

        // Обновление позиции кругов
        leftCircle.set({
          left: leftPanCenter.left - leftCircle.radius * leftCircle.scaleX,
          top: lineLeftPan.top - leftCircle.radius * 2 * leftCircle.scaleY,
        });

        rightCircle.set({
          left: rightPanCenter.left - rightCircle.radius * rightCircle.scaleX,
          top: lineRightPan.top - rightCircle.radius * 2 * rightCircle.scaleY,
        });

        canvas.renderAll();
      };

      leftCircle.on('scaling', recalculateBalance);
      rightCircle.on('scaling', recalculateBalance);
      leftCircle.on('modified', recalculateBalance);
      rightCircle.on('modified', recalculateBalance);
    });
  },
};
</script>

<style>
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f0f0;
}
</style>
