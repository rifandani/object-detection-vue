<template>
  <div>
    <video
      autoplay
      muted
      playsInline
      v-bind="camera"
      width="640"
      height="480"
      id="cam"
      style="position:fixed;top:50px"
    />
    <canvas id="c" width="640" height="480" style="position:fixed;top:50px" />
  </div>
</template>

<script>
import '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';
import { reactive, onMounted } from 'vue';

export default {
  name: 'App',
  setup() {
    // semacam useRef
    const camera = reactive({
      srcObject: null,
    });

    const webcam = async () => {
      try {
        camera.srcObject = await navigator.mediaDevices.getUserMedia({
          audio: false,
          video: true,
        });
        await camera.onloadedmetadata;
      } catch (e) {
        console.log(e);
      }
    };

    const load = async () => {
      try {
        // load webcam
        await webcam();

        // load pre-trained models
        const loadModel = await cocoSsd.load({ base: 'mobilenet_v2' });
        const cam = document.getElementById('cam');

        // panggil fungsi detectFromVideoFrame
        detectFromVideoFrame(loadModel, cam);
      } catch (e) {
        console.log(e);
      }
    };

    const detectFromVideoFrame = (model, video) => {
      // detect object dari video
      model.detect(video).then(
        (predictions) => {
          showDetection(predictions); // show object detection

          requestAnimationFrame(() => {
            detectFromVideoFrame(model, video);
          });
        },
        (error) => {
          console.log(error);
        },
      );
    };

    // draw the detections bounding boxes, as well as the labels, and confidence score over the video
    const showDetection = (prediction) => {
      const c = document.getElementById('c');
      const ctx = c.getContext('2d');
      ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
      const font = '24px helvetica';
      ctx.font = font;
      ctx.textBaseline = 'top';
      prediction.forEach((prediction) => {
        const x = prediction.bbox[0];
        const y = prediction.bbox[1];
        const width = prediction.bbox[2];
        const height = prediction.bbox[3];

        // Draw the bounding box.
        ctx.strokeStyle = '#2fff00';
        ctx.lineWidth = 1;
        ctx.strokeRect(x, y, width, height);

        // Draw the label background.
        ctx.fillStyle = '#2fff00';

        // ganti 'preson' jadi 'toilet'
        let predictionText = prediction.class;
        if (predictionText == 'person') {
          predictionText = 'toilet';
        }

        const textWidth = ctx.measureText(predictionText).width;
        const textHeight = parseInt(font, 10);

        // draw top left rectangle
        ctx.fillRect(x, y, textWidth + 10, textHeight + 10);

        // draw bottom left rectangle
        ctx.fillRect(
          x,
          y + height - textHeight,
          textWidth + 15,
          textHeight + 10,
        );

        // Draw the text last to ensure it's on top.
        ctx.fillStyle = '#000000';
        ctx.fillText(predictionText, x, y);
        ctx.fillText(prediction.score.toFixed(2), x, y + height - textHeight);
      });
    };

    // useEffect
    onMounted(() => {
      if (
        navigator.mediaDevices.getUserMedia ||
        navigator.mediaDevices.webkitGetUserMedia
      ) {
        load();
      } else {
        alert('not supported');
      }
    });

    return {
      camera,
    };
  },
};
</script>
