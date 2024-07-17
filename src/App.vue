<script setup>
import { ref, onMounted } from 'vue'
import RawSvgResult from "./assets/result.svg?raw"
import panzoom from "panzoom"

//actual svg reference
const svg = ref('')

//reference for svg wrapper, we add pan and zoom to this element
const svgWrapperInner = ref(null)

//reference for range input
const rangeInput = ref(null)

const panzoomInstance = ref(null)

function handleColorChange (event){
  const color = event.target.value
    svgWrapperInner.value.querySelectorAll('path[data-selected]').forEach(path => {
            path.style.fill = color;
        });
}

function selectPaths() {
    let isDragging = false;
    let dragStartCoordinates, dragEndCoordinates;

    svgWrapperInner.value.addEventListener('click', function(event) {
      if(event.ctrlKey || event.metaKey) return;
        const path = event.target.closest('path');
        if (!path) return; // If the clicked element is not a path, do nothing

        if (event.shiftKey) {
            // Toggle 'data-selected' attribute on the clicked path for multiple selection
            if (path.hasAttribute('data-selected')) {
                path.removeAttribute('data-selected');
            } else {
                path.setAttribute('data-selected', 'true');
            }
        } else {
            // For single selection, first clear all previous selections
            document.querySelectorAll('#svg-inner path[data-selected="true"]').forEach(selectedPath => {
                selectedPath.removeAttribute('data-selected');
            });

            // Then select the clicked path
            path.setAttribute('data-selected', 'true');
        }
    });

    svgWrapperInner.value.addEventListener('mousedown', function(event) {
      if(event.ctrlKey || event.metaKey) return;
        isDragging = true;
        dragStartCoordinates = { x: event.clientX, y: event.clientY };
    });

    svgWrapperInner.value.addEventListener('mousemove', function(event) {
        if (!isDragging) return;
        
        dragEndCoordinates = { x: event.clientX, y: event.clientY };

        svgWrapperInner.value.querySelectorAll('path').forEach(path => {
          const pathBoundingBox = path.getBoundingClientRect();
          const intersects = !(pathBoundingBox.right < dragStartCoordinates.x
                            || pathBoundingBox.bottom < dragStartCoordinates.y
                            || pathBoundingBox.left > dragEndCoordinates.x
                            || pathBoundingBox.top > dragEndCoordinates.y);

          if (intersects) {
            path.setAttribute('data-selected', 'true');
          } else if (!event.shiftKey) {
            path.removeAttribute('data-selected');
          }
        });
    });

    svgWrapperInner.value.addEventListener('mouseup', function() {
        isDragging = false;
    });
}

//called when component is mounted
onMounted(() => {
    //this will load the svg content to html
    svg.value = RawSvgResult
    //set panzoom instance to svg wrapper
    panzoomInstance.value = panzoom(svgWrapperInner.value, {
      beforeMouseDown: (e) =>{
        // allow pan only if ctrl key is down. Otherwise - ignore
        var shouldIgnore = !e.metaKey && !e.ctrlKey;
        return shouldIgnore
      },
      //dont allow zoom with mouse
      beforeWheel: function(e) {
      return true;
      },
      // disable all keyboard events, + , - etc
      filterKey: function() {
        return true;
      },
      maxZoom: 2,
      minZoom: 0.2
    });
    selectPaths()
})

const handleZoom = (event) => {
    const zoomValue = parseFloat(event.target.value)
      panzoomInstance.value.zoomAbs(0, 0,zoomValue );
    }
</script>

<template>
  <h1>SVG Viewer</h1>
  <p>Ctrl + mouse to move around</p>
   <div id="svg-viewer-container">
      <div id="svg-inner" ref="svgWrapperInner" v-html="svg">
      </div>
    </div>
    <input type="range" min="0.2" max="2" step="0.2" value="1" @input="handleZoom" id="zoom-slider">
    <select name="colors" id="select-colors" @input="handleColorChange">
      <option value="">Select Color</option>
            <option value="red">Red</option>
            <option value="green">Green</option>
            <option value="blue">Blue</option>
            <option value="yellow">Yellow</option>
          </select>
</template>

<style>
  #svg-viewer-container {
       display: flex;
       justify-content: center;
       align-items: center;
       border: 1px solid gray;
       width: 900px;
       height: 900px;
       overflow: hidden;
  }

  #svg-inner{
    width: 900px;
    height: 900px;
    transform-origin: center !important;
  }

  #svg-inner svg path:hover {
    stroke: red; /* Change the color as needed */
    stroke-width: 2; /* Adjust the border width as needed */
}

path[data-selected="true"] {
    fill: rgb(240, 240, 240); /* Change the color as needed */
    stroke-width: 4;
}
</style>