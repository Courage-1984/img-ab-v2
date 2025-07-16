import GIF from 'gif.js.optimized';

function onExportGifVideo({ frames, exportType, sliderSpeed }) {
  if (exportType === 'gif') {
    // Encode frames as GIF
    const gif = new GIF({
      workers: 2,
      quality: 10,
      workerScript: 'gif.worker.js'
    });
    frames.forEach(frame => {
      const img = new Image();
      img.src = frame;
      gif.addFrame(img, { delay: (sliderSpeed / frames.length) * 1000 });
    });
    gif.on('progress', p => {
      console.log('GIF encoding progress:', Math.round(p * 100) + '%');
    });
    gif.on('finished', blob => {
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'comparison.gif';
      document.body.appendChild(a);
      a.click();
      setTimeout(() => {
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
      }, 100);
    });
    gif.render();
  } else {
    // TODO: Video export
    console.log('Export frames:', frames.length, 'Type:', exportType, 'Speed:', sliderSpeed);
  }
} 