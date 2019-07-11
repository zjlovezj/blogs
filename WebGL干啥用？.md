# 背景
纯粹是因为使用TensorFlow.js的时候出现了WebGL的相关报错，所以想学学WebGL的基本知识，希望能方便识别错误。

## WebGL的历史
WebGL大概是2009年发布，其后慢慢在各个浏览器中实现的，目前(2019)可以在主流浏览器中的版本是1.0。  
WebGL 1.0 由OpenGL ES 2.0而来。  
借助于HTML5的canvas元素，可以来制作3D的图形。  
而Shaders是一种用类C语言编制的函数，用来做视觉特效。  

## 关键术语
Vertex shader: Vertex shaders are programs that describe the traits (position, colors, and so on) of a vertex.  
Fragment shader: The fragment is a WebGL term that you can consider as a kind of pixel (picture element).  

## 创建流程
1. Create shader objects (gl.createShader()).
2. Store the shader programs (to avoid confusion, we refer to them as “source code”) in the shader objects (g.shaderSource()).
3. Compile the shader objects (gl.compileShader()).
4. Create a program object (gl.createProgram()).
5. Attach the shader objects to the program object (gl.attachShader()).
6. Link the program object (gl.linkProgram()).
7. Tell the WebGL system the program object to be used (gl.useProgram()).

## 代码示例
```js
const VSHADER_SOURCE = `
  attribute vec4 a_Position;
  void main() {
    gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
    gl_Position = a_Position;
    gl_PointSize = 10.0;
  }
`;
const FSHADER_SOURCE = `
  void main() {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
  }
`;

export function drawPoint() {
  const canvas = document.getElementById("example");
  if (!(canvas instanceof HTMLCanvasElement)) {
    return;
  }
  /** @type {WebGLRenderingContext} */
  const gl = canvas.getContext("webgl");

  const shader_vs = createShader(gl, VSHADER_SOURCE, gl.VERTEX_SHADER);
  const shader_frag = createShader(gl, FSHADER_SOURCE, gl.FRAGMENT_SHADER);

  // Create shader program consisting of shader pair
  const program = gl.createProgram();

  let ret = gl.getProgramInfoLog(program);

  if (ret != "") console.log(ret);

  // Attach shaders to the program; these methods do not have a return value
  gl.attachShader(program, shader_vs);
  gl.attachShader(program, shader_frag);

  gl.linkProgram(program);

  // Check the link status
  const linked = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (!linked) {
    // something went wrong with the link
    console.error(gl.getProgramInfoLog(program));
  }

  gl.useProgram(program);

  // get the storage location of attribute variable
  const a_Position = gl.getAttribLocation(program, "a_Position");
  if (a_Position < 0) {
    console.log("Failed to get the storage location of a_Position");
    return;
  }

  gl.vertexAttrib3f(a_Position, 0.0, 0.0, 0.0);

  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);

  gl.drawArrays(gl.POINTS, 0, 1);
}

function createShader(gl, sourceCode, type) {
  // Compiles either a shader of type gl.VERTEX_SHADER or gl.FRAGMENT_SHADER
  let shader = gl.createShader(type);
  gl.shaderSource(shader, sourceCode);
  gl.compileShader(shader);

  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    let info = gl.getShaderInfoLog(shader);
    throw "Could not compile WebGL program. \n\n" + info;
  }
  return shader;
}

document.addEventListener("DOMContentLoaded", event => {
  drawPoint();
});
```
