<template>
  <div>
    <Divider plain orientation="left">{{ $t('common_elements') }}</Divider>
    <div class="tool-box">
      <span @click="() => addText()" :draggable="true" @dragend="onDragend('text')">
        <svg
          t="1650875455324"
          class="icon"
          viewBox="0 0 1024 1024"
          version="1.1"
          xmlns="http://www.w3.org/2000/svg"
          p-id="5401"
          width="26"
          height="26"
        >
          <path
            d="M213.333333 209.92v128h85.333334v-42.666667h170.666666v433.493334H384.853333v85.333333h256v-85.333333H554.666667V295.253333h170.666666v42.666667h85.333334v-128H213.333333z"
            p-id="5402"
          ></path>
        </svg>
      </span>
    </div>
  </div>
</template>

<script setup name="Tools">
import { v4 as uuid } from 'uuid';
// import initializeLineDrawing from '@/core/remove/initializeLineDrawing';
import { getPolygonVertices } from '@/utils/math';
import useSelect from '@/hooks/select';
import { useI18n } from 'vue-i18n';

// 默认属性
const defaultPosition = { shadow: '', fontFamily: 'arial' };
// 拖拽属性
const dragOption = {
  left: 0,
  top: 0,
};
const { t } = useI18n();
const { fabric, canvasEditor } = useSelect();
const state = reactive({
  isDrawingLineMode: false,
  isArrow: false,
});
// let drawHandler = null;

const addText = (option) => {
  const text = new fabric.IText(t('everything_is_fine'), {
    ...defaultPosition,
    ...option,
    fontSize: 80,
    id: uuid(),
  });
  canvasEditor.canvas.add(text);
  // if (!option) {
  //   text.center();
  // }
  canvasEditor.canvas.setActiveObject(text);
};

// const addImg = (e) => {
//   const imgEl = e.target.cloneNode(true);
//   const imgInstance = new fabric.Image(imgEl, {
//     ...defaultPosition,
//     id: uuid(),
//     name: '图片default',
//   });
//   canvasEditor.canvas.add(imgInstance);
//   canvasEditor.canvas.renderAll();
// };

const addTextBox = (option) => {
  const text = new fabric.Textbox(t('everything_goes_well'), {
    ...defaultPosition,
    ...option,
    splitByGrapheme: true,
    width: 400,
    fontSize: 80,
    id: uuid(),
  });
  canvasEditor.canvas.add(text);
  if (!option) {
    text.center();
  }
  canvasEditor.canvas.setActiveObject(text);
};

const addTriangle = (option) => {
  const triangle = new fabric.Triangle({
    ...defaultPosition,
    ...option,
    width: 400,
    height: 400,
    fill: '#92706B',
    id: uuid(),
    name: '三角形',
  });
  canvasEditor.canvas.add(triangle);
  if (!option) {
    triangle.center();
  }
  canvasEditor.canvas.setActiveObject(triangle);
};

const addPolygon = (option) => {
  const polygon = new fabric.Polygon(getPolygonVertices(5, 200), {
    ...defaultPosition,
    ...option,
    fill: '#ccc',
    id: uuid(),
    name: '多边形',
  });
  polygon.set({
    // 创建完设置宽高，不然宽高会变成自动的值
    width: 400,
    height: 400,
    // 关闭偏移
    pathOffset: {
      x: 0,
      y: 0,
    },
  });
  canvasEditor.canvas.add(polygon);
  if (!option) {
    polygon.center();
  }
  canvasEditor.canvas.setActiveObject(polygon);
};

const addCircle = (option) => {
  const circle = new fabric.Circle({
    ...defaultPosition,
    ...option,
    radius: 150,
    fill: '#57606B',
    id: uuid(),
    name: '圆形',
  });
  canvasEditor.canvas.add(circle);
  if (!option) {
    circle.center();
  }
  canvasEditor.canvas.setActiveObject(circle);
};

const addRect = (option) => {
  const rect = new fabric.Rect({
    ...defaultPosition,
    ...option,
    fill: '#F57274',
    width: 400,
    height: 400,
    id: uuid(),
    name: '矩形',
  });
  canvasEditor.canvas.add(rect);
  if (!option) {
    rect.center();
  }
  canvasEditor.canvas.setActiveObject(rect);
};
const drawingLineModeSwitch = (isArrow) => {
  state.isArrow = isArrow;
  state.isDrawingLineMode = !state.isDrawingLineMode;
  canvasEditor.setMode(state.isDrawingLineMode);
  canvasEditor.setArrow(isArrow);

  // this.canvasEditor.setMode(this.isDrawingLineMode);
  // this.canvasEditor.setArrow(isArrow);
  canvasEditor.canvas.forEachObject((obj) => {
    if (obj.id !== 'workspace') {
      obj.selectable = !state.isDrawingLineMode;
      obj.evented = !state.isDrawingLineMode;
    }
  });
};

// 拖拽开始时就记录当前打算创建的元素类型
const onDragend = (type) => {
  // todo 拖拽优化 this.canvas.editor.dragAddItem(event, item);
  switch (type) {
    case 'text':
      addText(dragOption);
      break;
    case 'textBox':
      addTextBox(dragOption);
      break;
    case 'rect':
      addRect(dragOption);
      break;
    case 'circle':
      addCircle(dragOption);
      break;
    case 'triangle':
      addTriangle(dragOption);
      break;
    case 'polygon':
      addPolygon(dragOption);
      break;
    default:
  }
};

onMounted(() => {
  nextTick(() => {
    // 线条绘制
    // drawHandler = initializeLineDrawing(canvasEditor.canvas, defaultPosition);

    canvasEditor.canvas.on('drop', (opt) => {
      // 画布元素距离浏览器左侧和顶部的距离
      const offset = {
        left: canvasEditor.canvas.getSelectionElement().getBoundingClientRect().left,
        top: canvasEditor.canvas.getSelectionElement().getBoundingClientRect().top,
      };

      // 鼠标坐标转换成画布的坐标（未经过缩放和平移的坐标）
      const point = {
        x: opt.e.x - offset.left,
        y: opt.e.y - offset.top,
      };

      // 转换后的坐标，restorePointerVpt 不受视窗变换的影响
      const pointerVpt = canvasEditor.canvas.restorePointerVpt(point);
      dragOption.left = pointerVpt.x;
      dragOption.top = pointerVpt.y;
    });
  });
});
</script>

<style scoped lang="less">
.tool-box {
  display: flex;
  justify-content: space-around;
  span {
    flex: 1;
    text-align: center;
    padding: 5px 0;
    background: #f6f6f6;
    margin-left: 2px;
    cursor: pointer;
    &:hover {
      background: #edf9ff;
      svg {
        fill: #2d8cf0;
      }
    }
  }
  .bg {
    background: #d8d8d8;

    &:hover {
      svg {
        fill: #2d8cf0;
      }
    }
  }
}
.img {
  width: 20px;
}
</style>
