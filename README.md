# ASSIGMENT 3 - GrafKom A
Daffa Saskara - NRP
5025211249

## Task
 - Draw a 2D Square
 - Show another side of the 3D Cube
   
## Solution(s)
### Task 1
![2d](https://github.com/daffasas/assignment3-computer-graphics/assets/88588446/4fc2be1f-cbac-4099-b7de-2d392e11a18a)
<br>
<br>
Kotak tersebut dibuat dengan menggambar dua segitiga individu pada koordinat yang tepat sehingga mereka menyatu menjadi satu dan terlihat seperti sebuah kotak tunggal. Berikut potongan kode dari proses penggambaran dan penempatan tersebut.
```javascript
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Square in WebGL</title>
    
    <script>
      "use strict";

      const vertexShaderSource =
        "attribute vec3 a_coords;\n" +
        "attribute vec3 a_color;\n" +
        "varying vec3 v_color;\n" +
        "void main() {\n" +
        "   gl_Position = vec4(a_coords, 1.0);\n" +
        "   v_color = a_color;\n" +
        "}\n";

      const fragmentShaderSource =
        "precision mediump float;\n" +
        "varying vec3 v_color;\n" +
        "void main() {\n" +
        "   gl_FragColor = vec4(v_color, 1.0);\n" +
        "}\n";

      let gl;
      let attributeCoords;
      let bufferCoords;
      let attributeColor;
      let bufferColor;

      function draw() {
        gl.clearColor(0, 0, 0, 1);
        gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

        let vertices = new Float32Array([
          // Define the vertices for the square
          -0.5,
          -0.5,
          0.0, // Vertex 0
          0.5,
          -0.5,
          0.0, // Vertex 1
          0.5,
          0.5,
          0.0, // Vertex 2
          -0.5,
          -0.5,
          0.0, // Vertex 3 (Shared with Vertex 0)
          0.5,
          0.5,
          0.0, // Vertex 4 (Shared with Vertex 2)
          -0.5,
          0.5,
          0.0, // Vertex 5
        ]);

        let colors = new Float32Array([
          // Define colors for the vertices (RGB)
          1,
          0,
          0, // Red
          0,
          1,
          0, // Green
          0,
          0,
          1, // Blue
          1,
          0,
          0, // Red (Shared with Vertex 0)
          0,
          0,
          1, // Blue (Shared with Vertex 2)
          0,
          1,
          0, // Green
        ]);

        gl.bindBuffer(gl.ARRAY_BUFFER, bufferCoords);
        gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
        gl.vertexAttribPointer(attributeCoords, 3, gl.FLOAT, false, 0, 0);
        gl.enableVertexAttribArray(attributeCoords);

        gl.bindBuffer(gl.ARRAY_BUFFER, bufferColor);
        gl.bufferData(gl.ARRAY_BUFFER, colors, gl.STATIC_DRAW);
        gl.vertexAttribPointer(attributeColor, 3, gl.FLOAT, false, 0, 0);
        gl.enableVertexAttribArray(attributeColor);

        // Triangle 1
        gl.drawArrays(gl.TRIANGLES, 0, 3);

        // Triangle 2
        gl.drawArrays(gl.TRIANGLES, 3, 3);
      }

      function createProgram(gl, vertexShaderSource, fragmentShaderSource) {
        let vsh = gl.createShader(gl.VERTEX_SHADER);
        gl.shaderSource(vsh, vertexShaderSource);
        gl.compileShader(vsh);
        if (!gl.getShaderParameter(vsh, gl.COMPILE_STATUS)) {
          throw new Error(
            "Error in vertex shader: " + gl.getShaderInfoLog(vsh)
          );
        }
        let fsh = gl.createShader(gl.FRAGMENT_SHADER);
        gl.shaderSource(fsh, fragmentShaderSource);
        gl.compileShader(fsh);
        if (!gl.getShaderParameter(fsh, gl.COMPILE_STATUS)) {
          throw new Error(
            "Error in fragment shader: " + gl.getShaderInfoLog(fsh)
          );
        }
        let prog = gl.createProgram();
        gl.attachShader(prog, vsh);
        gl.attachShader(prog, fsh);
        gl.linkProgram(prog);
        if (!gl.getProgramParameter(prog, gl.LINK_STATUS)) {
          throw new Error(
            "Link error in program: " + gl.getProgramInfoLog(prog)
          );
        }
        return prog;
      }

      function initGL() {
        let prog = createProgram(gl, vertexShaderSource, fragmentShaderSource);
        gl.useProgram(prog);

        bufferCoords = gl.createBuffer();
        bufferColor = gl.createBuffer();

        attributeCoords = gl.getAttribLocation(prog, "a_coords");
        attributeColor = gl.getAttribLocation(prog, "a_color");
      }

      function init() {
        try {
          let canvas = document.getElementById("webglcanvas");
          gl = canvas.getContext("webgl");
          if (!gl) {
            throw "Browser does not support WebGL";
          }
        } catch (e) {
          document.getElementById("canvas-holder").innerHTML =
            "<p>Sorry, could not get a WebGL graphics context.</p>";
          return;
        }
        try {
          initGL();
        } catch (e) {
          document.getElementById("canvas-holder").innerHTML =
            "<p>Sorry, could not initialize the WebGL graphics context: " +
            e.message +
            "</p>";
          return;
        }
        draw();
      }

```

## Task 2
![cubeperf](https://github.com/daffasas/assignment3-computer-graphics/assets/88588446/43bbafc6-6c35-4e7b-a09f-1192ad803305)
<br><br><br>
Berikut adalah Kubus 3D yang di ubah warnanya
<br><br><br><br><br>
Kode lengkap:
```javascript
<!DOCTYPE html>
<meta charset="UTF-8" />
<html>
  <head>
    <title>Rotating Cube with glMatrix and WebGL</title>
    <style>
      body {
        background-color: #eeeeee;
      }
    </style>

    <!--
     A first example of using glMatrix to implement
     projection and modelview transformations.  Also
     an example of using <script> tags to hold the
     source code for the shader program.
-->

    <script type="x-shader/x-vertex" id="vshader-source">
      attribute vec3 a_coords;
      void main() {
          vec4 coords = vec4(a_coords,1.0);
          gl_Position = coords;
      }
    </script>

    <script type="x-shader/x-fragment" id="fshader-source">
      #ifdef GL_FRAGMENT_PRECISION_HIGH
         precision highp float;
      #else
         precision mediump float;
      #endif
      uniform vec4 color;
      void main() {
          gl_FragColor = color;
      }
    </script>

    <script src="gl-matrix-min.js"></script>

    <script>
      "use strict";

      let gl; // The webgl context.

      let a_coords_loc; // Location of the a_coords attribute variable in the shader program.
      let a_coords_buffer; // Buffer to hold the values for a_coords.
      let u_color; // Location of the uniform specifying a color for the primitive.

      /* Draws a WebGL primitive.  The first parameter must be one of the constants
       * that specify primitives:  gl.POINTS, gl.LINES, gl.LINE_LOOP, gl.LINE_STRIP,
       * gl.TRIANGLES, gl.TRIANGLE_STRIP, gl.TRIANGLE_FAN.  The second parameter must
       * be an array of 4 numbers in the range 0.0 to 1.0, giving the RGBA color of
       * the color of the primitive.  The third parameter must be an array of numbers.
       * The length of the array must be a multiple of 3.  Each triple of numbers provides
       * xyz-coords for one vertex for the primitive.  This assumes that u_color is the
       * location of a color uniform in the shader program, a_coords_loc is the location of
       * the coords attribute, and a_coords_buffer is a VBO for the coords attribute.
       */
      function drawPrimitive(primitiveType, color, vertices) {
        gl.enableVertexAttribArray(a_coords_loc);
        gl.bindBuffer(gl.ARRAY_BUFFER, a_coords_buffer);
        gl.bufferData(
          gl.ARRAY_BUFFER,
          new Float32Array(vertices),
          gl.STREAM_DRAW
        );
        gl.uniform4fv(u_color, color);
        gl.vertexAttribPointer(a_coords_loc, 3, gl.FLOAT, false, 0, 0);
        gl.drawArrays(primitiveType, 0, vertices.length / 3);
      }

      /* Draws a colored cube, along with a set of coordinate axes.
       * (Note that the use of the above drawPrimitive function is not an efficient
       * way to draw with WebGL.  Here, the geometry is so simple that it doesn't matter.)
       */
      
        function draw() {
        gl.clearColor(0, 0, 0, 1);
        gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

        drawPrimitive(gl.TRIANGLE_FAN, [1, 0.5, 0, 1], [-0.5, 0.3, -0.5, 0, 0.1, -0.5, 0.5, 0.3, -0.5, 0, 0.5, 0.4]); // front roof (orange)
        //drawPrimitive(gl.TRIANGLE_FAN, [0.5, 1, 0, 1], [0, -0.4, 0, 0.5, -0.2, 0, 0.5, 0.3, 0, 0, 0.1, 0]); // front right side (lime green)
        drawPrimitive(gl.TRIANGLE_FAN, [1, 0.84, 0, 1], [-0.5, -0.2, -0.5, -0.5, 0.3, -0.5, 0, 0.1, -0.5, 0, -0.4, -0.5]); // front left side (gold)
        drawPrimitive(gl.TRIANGLE_FAN, [0, 1, 1, 1], [-0.5, 0.3, 0.5, -0.5, -0.2, 0.5, 0, 0, 0.5, 0, 0.5, 0.4]); //back left side (cyan)
        drawPrimitive(gl.TRIANGLE_FAN, [1, 0, 1, 1], [0.5, 0.3, 0.5, 0.5, -0.2, 0.5, 0, 0, 0.5, 0, 0.5, 0.4]); //back right side (magenta)
        drawPrimitive(gl.TRIANGLE_FAN, [0, 0.5, 0.5, 1], [-0.5, -0.2, 0.5, 0, -0, 0.5, 0.5, -0.2, 0.5, 0, -0.4, 0.4] ); //base (teal) 
        }
        
        

      /* Initialize the WebGL context.  Called from init() */
      function initGL() {
        let prog = createProgram(gl, "vshader-source", "fshader-source");
        gl.useProgram(prog);
        a_coords_loc = gl.getAttribLocation(prog, "a_coords");
        u_color = gl.getUniformLocation(prog, "color");
        a_coords_buffer = gl.createBuffer();
        gl.enable(gl.DEPTH_TEST); //memastikan objek yang didepan kamera tidak tertumpuk
      }

      /* Creates a program for use in the WebGL context gl, and returns the
       * identifier for that program.  If an error occurs while compiling or
       * linking the program, an exception of type Error is thrown.  The error
       * string contains the compilation or linking error.  If no error occurs,
       * the program identifier is the return value of the function.
       *    The second and third parameters are the id attributes for <script>
       * elements that contain the source code for the vertex and fragment
       * shaders.
       */
      function createProgram(gl, vertexShaderID, fragmentShaderID) {
        function getTextContent(elementID) {
          // This nested function retrieves the text content of an
          // element on the web page.  It is used here to get the shader
          // source code from the script elements that contain it.
          let element = document.getElementById(elementID);
          let node = element.firstChild;
          let str = "";
          while (node) {
            if (node.nodeType === 3)
              // this is a text node
              str += node.textContent;
            node = node.nextSibling;
          }
          return str;
        }
        let vertexShaderSource, fragmentShaderSource;
        try {
          vertexShaderSource = getTextContent(vertexShaderID);
          fragmentShaderSource = getTextContent(fragmentShaderID);
        } catch (e) {
          throw new Error(
            "Error: Could not get shader source code from script elements."
          );
        }
        let vsh = gl.createShader(gl.VERTEX_SHADER);
        gl.shaderSource(vsh, vertexShaderSource);
        gl.compileShader(vsh);
        if (!gl.getShaderParameter(vsh, gl.COMPILE_STATUS)) {
          throw new Error(
            "Error in vertex shader:  " + gl.getShaderInfoLog(vsh)
          );
        }
        let fsh = gl.createShader(gl.FRAGMENT_SHADER);
        gl.shaderSource(fsh, fragmentShaderSource);
        gl.compileShader(fsh);
        if (!gl.getShaderParameter(fsh, gl.COMPILE_STATUS)) {
          throw new Error(
            "Error in fragment shader:  " + gl.getShaderInfoLog(fsh)
          );
        }
        let prog = gl.createProgram();
        gl.attachShader(prog, vsh);
        gl.attachShader(prog, fsh);
        gl.linkProgram(prog);
        if (!gl.getProgramParameter(prog, gl.LINK_STATUS)) {
          throw new Error(
            "Link error in program:  " + gl.getProgramInfoLog(prog)
          );
        }
        return prog;
      }

      /**
       * initialization function that will be called when the page has loaded
       */
      function init() {
        try {
          let canvas = document.getElementById("webglcanvas");
          gl = canvas.getContext("webgl");
          if (!gl) {
            throw "Browser does not support WebGL";
          }
        } catch (e) {
          document.getElementById("canvas-holder").innerHTML =
            "<p>Sorry, could not get a WebGL graphics context.</p>";
          return;
        }
        try {
          initGL(); // initialize the WebGL graphics context
        } catch (e) {
          document.getElementById("canvas-holder").innerHTML =
            "<p>Sorry, could not initialize the WebGL graphics context: " +
            e.message +
            "</p>";
          return;
        }

             draw();
        }

      window.onload = init;
```

- Kubus tanpa command
```javascript
drawPrimitive(gl.TRIANGLE_FAN, [1, 0.5, 0, 1], [-0.5, 0.3, -0.5, 0, 0.1, -0.5, 0.5, 0.3, -0.5, 0, 0.5, 0.4]); // front roof (orange)
        //drawPrimitive(gl.TRIANGLE_FAN, [0.5, 1, 0, 1], [0, -0.4, 0, 0.5, -0.2, 0, 0.5, 0.3, 0, 0, 0.1, 0]); // front right side (lime green)
        drawPrimitive(gl.TRIANGLE_FAN, [1, 0.84, 0, 1], [-0.5, -0.2, -0.5, -0.5, 0.3, -0.5, 0, 0.1, -0.5, 0, -0.4, -0.5]); // front left side (gold)
        drawPrimitive(gl.TRIANGLE_FAN, [0, 1, 1, 1], [-0.5, 0.3, 0.5, -0.5, -0.2, 0.5, 0, 0, 0.5, 0, 0.5, 0.4]); //back left side (cyan)
        drawPrimitive(gl.TRIANGLE_FAN, [1, 0, 1, 1], [0.5, 0.3, 0.5, 0.5, -0.2, 0.5, 0, 0, 0.5, 0, 0.5, 0.4]); //back right side (magenta)
        drawPrimitive(gl.TRIANGLE_FAN, [0, 0.5, 0.5, 1], [-0.5, -0.2, 0.5, 0, -0, 0.5, 0.5, -0.2, 0.5, 0, -0.4, 0.4] ); //base (teal) 
        }
```

![cube2](https://github.com/daffasas/assignment3-computer-graphics/assets/88588446/8be12010-5f5a-40b3-87f6-a7be4fa38bd6)

anda bisa meng-edit atau menghabus fungsi lainnya untuk "mengupas kubus" nya
