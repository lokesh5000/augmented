---
id: home 
title: AR PIC
sidebar_label: Home 
slug: /
---

import {useEffect, useRef, useState, useCallback, useMemo} from 'react';
import styles from './compile.module.scss';
import clsx from 'clsx';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import BrowserOnly from '@docusaurus/BrowserOnly';

export const UploadTab = ({onDataReady}) => {
  const [error, setError] = useState('');
  const [dropzone, setDropzone] = useState(null);
  const [percentage, setPercentage] = useState(null);
  const refDropzone = useRef(null);
  const compiler = new window.MINDAR.IMAGE.Compiler();
  const loadImage = useCallback(async (file) => {
    return new Promise((resolve, reject) => {
      let img = new Image()
      img.onload = () => resolve(img);
      img.onerror = reject;
      img.src = URL.createObjectURL(file);
    })
  }, []);
  const compileFiles = useCallback(async (files) => {
    setPercentage(0);
    const images = [];
    for (let i = 0; i < files.length; i++) {
      images.push(await loadImage(files[i]));
    }
    console.log("files", files, images);
    const dataList = await compiler.compileImageTargets(images, (progress) => {
      setPercentage(progress.toFixed(2));
    });
    const exportedBuffer = await compiler.exportData();
    onDataReady(dataList, exportedBuffer);
  }, []);
  const loadMindFile = useCallback(async (file) => {
    var reader = new FileReader();
    reader.onload = async function() {
      const dataList = compiler.importData(this.result);
      const exportedBuffer = await compiler.exportData();
      onDataReady(dataList, exportedBuffer);
    }
    reader.readAsArrayBuffer(file);
  }, []);
  useEffect(() => {
    console.log("use effect", refDropzone.current);
    const myDropzone = new Dropzone(refDropzone.current, { url: "#", autoProcessQueue: false, addRemoveLinks: true });
    setDropzone(myDropzone);
  }, []);
  const startHandler = useCallback(async () => {
    const files = dropzone.files;
    if (files.length === 0) {
      setError("please select images.");
      return;
    }
    const ext = files[0].name.split('.').pop();
    if (ext === 'mind') {
      loadMindFile(files[0]); 
    } else {
      compileFiles(files);
    }
  }, [dropzone]); 
  return (
    <div>
      <p>Select target images and start</p>
      <div ref={refDropzone} className="dropzone"></div>
      {error && <p className="text--danger">{error}</p>}
      <div className="padding-vert--md">
	{percentage === null && <button className={clsx("button", "button--primary", styles.startButton)} onClick={startHandler}>Start</button>}
	{percentage !== null && (
	  <div>
	    Progress: {percentage} %
	  </div>	
	)}
      </div>
    </div>
  )
}

export const VisualizeTab = ({dataList, exportedBuffer}) => {
  const canvasRef = useRef(null);
  const [targetIndex, setTargetIndex] = useState(0);
  const [keyframeIndex, setKeyframeIndex] = useState(0);
  const drawPoint = useCallback((ctx, color, centerX, centerY, radius=1) => {
    ctx.beginPath();
    ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI, false);
    ctx.fillStyle = color;
    ctx.lineWidth = 1;
    ctx.strokeStyle = color;
    ctx.stroke();
  }, []);
  const drawRect = useCallback((ctx, color, centerX, centerY, width) => {
    ctx.beginPath();
    ctx.strokeStyle = color;
    ctx.rect(centerX - width/2, centerY - width /2 , width, width);
    ctx.stroke();
  }, []);
  useEffect(() => {
    const targetImage = dataList[targetIndex].imageList[keyframeIndex];
    const matchingPoints = [...dataList[targetIndex].matchingData[keyframeIndex].maximaPoints, dataList[targetIndex].matchingData[keyframeIndex].minimaPoints];
    const canvas = canvasRef.current;
    canvas.width = targetImage.width;
    canvas.height = targetImage.height;
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const tData = new Uint8ClampedArray(targetImage.width * targetImage.height * 4);
    for (let i = 0; i < targetImage.data.length; i++) {
      tData[i*4+0] = targetImage.data[i];
      tData[i*4+1] = targetImage.data[i];
      tData[i*4+2] = targetImage.data[i];
      tData[i*4+3] = 255;
    }
    const imageData = new ImageData(tData, targetImage.width, targetImage.height);
    ctx.putImageData(imageData, 0, 0);
    for (let i = 0; i < matchingPoints.length; i++) {
      const point = matchingPoints[i];
      drawPoint(ctx, '#ff0000', Math.round(point.x), Math.round(point.y), point.scale);
    }
  }, [dataList, targetIndex, keyframeIndex]);
  const numTargetRange = useMemo(() => {
    return dataList.map((data, index) => index);
  }, [dataList])
  const numScaleRange = useMemo(() => {
    return dataList[targetIndex].imageList.map((data, index) => index);
  }, [dataList, targetIndex]);
  const canvasStyle = useMemo(() => {
    const width = (100 * dataList[targetIndex].imageList[keyframeIndex].scale);
    return {
      width: width + "%",
      maxHeight: "100%",
      top: ((100-width)/2) + "%"
    }
  }, [dataList, targetIndex, keyframeIndex]);
  const downloadHandler = useCallback(() => {
    var blob = new Blob([exportedBuffer]);
    var aLink = window.document.createElement('a');
    aLink.download = 'targets.mind';
    aLink.href = window.URL.createObjectURL(blob);
    aLink.click();
    window.URL.revokeObjectURL(aLink.href);
  }, [exportedBuffer]);
  return (
    <div>
      <div className="tabs-container">
	<ul role="tablist" aria-orientation="horizontal" className={clsx('tabs')}>
	  {numTargetRange.map((index) => (
	    <li
	      role="tab"
	      className={clsx('tabs__item', {
		'tabs__item--active': targetIndex === index,
	      }, styles.tabItem)}
	      key={index}
	      onClick={() => {setTargetIndex(index); setKeyframeIndex(0)}}>
	      Image {index+1}
	    </li>
	  ))}
	</ul>
      </div>
      <div className="tabs-container">
	<ul role="tablist" aria-orientation="horizontal" className={clsx('tabs')}>
	  {numScaleRange.map((index) => (
	    <li
	      role="tab"
	      className={clsx('tabs__item', {
		'tabs__item--active': keyframeIndex === index,
	      }, styles.tabItem)}
	      key={index}
	      onClick={() => {setKeyframeIndex(index)}}>
	      Scale {index+1}
	    </li>
	  ))}
	</ul>
      </div>
      <div className={styles.visualizeCanvasWrapper}>
	<canvas style={canvasStyle} className={styles.visualizeCanvas} ref={canvasRef}></canvas>
      </div>
      <div className="padding-vert--md">
	<button className={clsx("button", "button--primary", styles.startButton)} onClick={downloadHandler}>Download compiled</button>
      </div>
    </div>
  )
}

export const Compile = () => {
  const [step, setStep] = useState('upload');
  const [dataList, setDataList] = useState(null);
  const [exportedBuffer, setExportedBuffer] = useState(null);
  const onDataReady = useCallback((dataList, exportedBuffer) => {
    setDataList(dataList);
    setExportedBuffer(exportedBuffer);
    setStep('visualize');
  }, []);
  return (
    <BrowserOnly>
     {() => (
      <div>
	{step === 'upload' && <UploadTab onDataReady={onDataReady}/>}
	{step === 'visualize' && <VisualizeTab dataList={dataList} exportedBuffer={exportedBuffer}/>}
      </div>
     )}
    </BrowserOnly>
  )
}

<Compile/>

