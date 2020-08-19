---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "NGLVIEW"
subtitle: ""
summary: "NGLVIEW basic example for my website with a few example"
authors: []
tags: [visualize]
categories: []
date: 2020-08-15T09:25:09+08:00
lastmod: 2020-08-15T09:25:09+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

projects: []
---

## NGLVIEW Show PDB Structure

- show raw pdb structure with different representation

```html
  </style>
  <script src="/js/ngl.js"></script>
  <script>
    document.addEventListener("DOMContentLoaded", function () {
      var stage = new NGL.Stage("viewport", {backgroundColor: "black"});
      stage.loadFile("rcsb://1cfd").then(function(component) {
        // component.addRepresentation("cartoon", {colorScheme: "sstruc"});
        component.addRepresentation("cartoon", {color: "gray", opacity: 0.7});
        component.addRepresentation("ball+stick", {sele: "12-13"});
        component.autoView("12-13");
        })
      stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
      <!-- stage.keyControls.add("r", NGL.KeyActions.autoView); -->
    });
  </script>

  <div id="viewport" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div>
```

  </style>
  <meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">

  <script src="/js/ngl.js"></script>
  <script>
    document.addEventListener("DOMContentLoaded", function () {
      var stage = new NGL.Stage("viewport", {backgroundColor: "black"});
      stage.loadFile("rcsb://1cfd").then(function(component) {
        // component.addRepresentation("cartoon", {colorScheme: "sstruc"});
        component.addRepresentation("cartoon", {color: "gray", opacity: 0.7});
        component.addRepresentation("ball+stick", {sele: "12-13"});
        component.autoView("12-13");
        })
      stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
      <!-- stage.keyControls.add("r", NGL.KeyActions.autoView); -->
    });
  </script>

  <div id="viewport" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div>


- show two structures and align

```html
  <script>
    document.addEventListener("DOMContentLoaded", function () {
      var stage = new NGL.Stage("viewport2", {backgroundColor: "black"});
      stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
      Promise.all([
        stage.loadFile('rcsb://1cll').then(function(o) {
          o.addRepresentation('cartoon', {color: 'lightgreen'})
          o.autoView()
          return o;
        }),
        stage.loadFile('rcsb://1cfd').then(function(o) {
          o.addRepresentation('cartoon', {color: 'tomato'})
          o.autoView()
          return o;
        })
      ]).then(function (ol) {
        var s1 = ol[0].structure
        var s2 = ol[1].structure
        NGL.superpose(s1, s2, true, "1-63.CA", "1-63.CA")
        ol[ 0 ].updateRepresentations({ position: true })
      })
    });
  </script>
  <div id="viewport2" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div>
```

  <script>
    document.addEventListener("DOMContentLoaded", function () {
      var stage = new NGL.Stage("viewport2", {backgroundColor: "black"});
      stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
      Promise.all([
        stage.loadFile('rcsb://1cll').then(function(o) {
          o.addRepresentation('cartoon', {color: 'lightgreen'})
          o.autoView()
          return o;
        }),
        stage.loadFile('rcsb://1cfd').then(function(o) {
          o.addRepresentation('cartoon', {color: 'tomato'})
          o.autoView()
          return o;
        })
      ]).then(function (ol) {
        var s1 = ol[0].structure
        var s2 = ol[1].structure
        NGL.superpose(s1, s2, true, "1-63.CA", "1-63.CA")
        ol[ 0 ].updateRepresentations({ position: true })
      })
    });
  </script>
  <div id="viewport2" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div>

- add a pdb INPUT for showing the structure and double click for a full screen

```html
  <script>
    var stage;
    document.addEventListener("DOMContentLoaded", function () {
      stage = new NGL.Stage("viewport3");
      stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
      stage.viewer.container.addEventListener("dblclick", function () {
          stage.toggleFullscreen();
      });
      function handleResize () {
        stage.handleResize();
      }
      document.getElementById("pdbidInput").addEventListener("keydown", function (e) {
        if (e.keyCode === 13) {
          stage.removeAllComponents();
          var url = "rcsb://" + e.target.value + ".mmtf";
          stage.loadFile(url, {defaultRepresentation: true});
          e.target.value = "";
          e.target.blur();
        }
      });
    });
  </script>
  <div id="viewport3" style="width:100%; height:400px;"></div>
  <div style="text-align:center; position:relative; bottom:3em;">
    <div style="display:inline-block;">
      <input id="pdbidInput" style="opacity:0.7; width:4em;"/>
    </div>
  </div>
```

  <script>
    var stage;
    document.addEventListener("DOMContentLoaded", function () {
      stage = new NGL.Stage("viewport3");
      stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
      stage.viewer.container.addEventListener("dblclick", function () {
          stage.toggleFullscreen();
      });
      function handleResize () {
        stage.handleResize();
      }
      document.getElementById("pdbidInput").addEventListener("keydown", function (e) {
        if (e.keyCode === 13) {
          stage.removeAllComponents();
          var url = "rcsb://" + e.target.value + ".mmtf";
          stage.loadFile(url, {defaultRepresentation: true});
          e.target.value = "";
          e.target.blur();
        }
      });
    });
  </script>
  <div id="viewport3" style="width:100%; height:400px;"></div>
  <div style="text-align:center; position:relative; bottom:3em;">
    <div style="display:inline-block;">
      <input id="pdbidInput" style="opacity:0.7; width:4em;"/>
    </div>
  </div>


## Draw custom shape

- show arrow beteen two atoms and zoom

```html
  <script>
    document.addEventListener("DOMContentLoaded", function () {
    var stage = new NGL.Stage("viewport4", {backgroundColor: "white"});
    stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
    stage.loadFile("rcsb://1crn.mmtf").then(function (o){
      o.addRepresentation("cartoon", {opacity: 0.3, depthWrite: false, flatShaded: false, side: "double"})
      const ap1 = o.structure.getAtomProxy();
      const ap2 = o.structure.getAtomProxy();
      var idx = o.structure.getAtomIndices(new NGL.Selection("(10.CA) or (11.CA)"))
      ap1.index = idx[0];
      ap2.index = idx[1];
      var shape = new NGL.Shape('shape', { dashedCylinder: true });
      shape.addArrow(ap1.positionToVector3(), ap2.positionToVector3(), [ 0, 1, 1 ], 0.3)
      var shapeComp = stage.addComponentFromObject( shape );
      o.addRepresentation('ball+stick', {sele:"10, 11"})
      shapeComp.addRepresentation( "buffer" );
      var center = o.getCenter("10, 11")
      var zoom = o.getZoom("10, 11")
      stage.animationControls.zoomMove(center, zoom, 0);
      });
    });
  </script>
  <div id="viewport4" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div>
```

  <script>
    document.addEventListener("DOMContentLoaded", function () {
    var stage = new NGL.Stage("viewport4", {backgroundColor: "white"});
    stage.mouseControls.remove("scroll", NGL.MouseActions.zoomScroll)
    stage.loadFile("rcsb://1crn.mmtf").then(function (o){
      o.addRepresentation("cartoon", {opacity: 0.3, depthWrite: false, flatShaded: false, side: "double"})
      const ap1 = o.structure.getAtomProxy();
      const ap2 = o.structure.getAtomProxy();
      var idx = o.structure.getAtomIndices(new NGL.Selection("(10.CA) or (11.CA)"))
      ap1.index = idx[0];
      ap2.index = idx[1];
      var shape = new NGL.Shape('shape', { dashedCylinder: true });
      shape.addArrow(ap1.positionToVector3(), ap2.positionToVector3(), [ 0, 1, 1 ], 0.3)
      var shapeComp = stage.addComponentFromObject( shape );
      o.addRepresentation('ball+stick', {sele:"10, 11"})
      shapeComp.addRepresentation( "buffer" );
      var center = o.getCenter("10, 11")
      var zoom = o.getZoom("10, 11")
      stage.animationControls.zoomMove(center, zoom, 0);
      });
    });
  </script>
  <div id="viewport4" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div>

<!-- ## Load A trajectory

<script>
  document.addEventListener("DOMContentLoaded", function () {
  var stage = new NGL.Stage("viewport5", {backgroundColor: "white"});
  stage.loadFile('rcsb://2MI7.pdb').then(function (o) {
    NGL.autoLoad('rcsb://2MI7.pdb').then(function (frames) {
      o.addTrajectory(frames, {
        initialFrame: 0,
        deltaTime: 200
      })
      o.addRepresentation('licorice', {scale: 0.5})
      o.addRepresentation('spacefill', {sele: 'not :B'})
      o.addRepresentation('cartoon')
      o.addRepresentation('backbone')
      stage.autoView()})
    })
  })
</script>
<div id="viewport5" style="width:100%; height:400px; margin: 0 auto; overflow:hidden;"> </div> -->