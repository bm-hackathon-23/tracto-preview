# Hackhathon topic - Improving Tractography preview for better data exploration

## Context

On the dataportal (see [NA216 dataset](https://dataportal.brainminds.jp/marmoset-mri-na216)), tractography data can be inspected with an online previewer leaveraging [XTk](https://github.com/xtk/X) javascript library.
In practice, the rendered data often looks like a dense ball of fibers, and that's not so easy to explore the fibers details, especially for the inner regions which are  obscured by outer regions.

![current tracto viewer](docs/images/tracto-preview-current.png)


Indeed, with Xtk it's only possible:

* to change the opacity of all fibers (of a trk file) at once, 
* to hide some of them depending on their length (by setting min and max fibers length filter values).


## Proposal

It would be good to somehow be able to only view fibers inside a user-defined region of the space.
(Tanaka-sensei proposal "three othogonal guide 'planes' of some thickness. Center and thickness can be specified in the UI with sliders").

Ideally by continuing using Xtk  which is currently used by the online preview.

### Goal

Extending tractography viewer to only display fibers in a user-defined volume (rectangular cuboid aligned to axis).

It means cropping the fiber rendering outside of the user defined volume, as figured in the schema below:
![tracto viewer goal](docs/images/tracto-preview-goal.png)


## Implementation

### Analysis of existing code

* XTk is webGL powered; It defines a couple of generic shaders (in [`shader.js`](https://github.com/xtk/X/blob/master/visualization/shaders.js)) that can handle rendering of NifTI volumes, tractography fibers and more.
<br/>(see [Webgl fundamentals](https://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html) for explanations about Webgl shaders)

* The shaders already includes mechanisms to adjust rendering of loaded objects depending on the value of user-defined criteria.
For example, in case of tractography data, some individual fibers may be masked depending on their overall length.

* Fiber files in TrackVis format (`.trk` ) are parsed by [`parserTRK.js`](https://github.com/xtk/X/blob/master/io/parserTRK.js). Each fiber segment is assigned a few _scalars_, including the overall length of the fiber they belong to. This enables the shader to discard the rendering of segments depending the length of their fiber.
<br/>(For fibers, WebGLRendering mode is `LINES`, to draw lines segment between pairs of vertices. GLSL discard keyword is used the shaders to hide fiber segments.)

### Tentative

In a first approach, we could probably obtain the desired effect of volume cropping by simply preventing the rendering of fiber segments that extend outside the user-defined volume of interest (similar logic as length based fiber masking described above).

This would work if fibers are composed of many short-length line segments (which seems a reasonable assumption since in practice fibers tend not to have long straight stretches).
Because in this first approximation the fibers would not be strictly cut at the volume edges (but rather at the boundaries of the neighbouring segment completely contained in the cuboid), the volume boundaries might not appear neat.

Moreover, we have to expect that when users define very thin volume (compared to segment length), the number of rendered fibers might become artificially very low.

We'll need to asses whether these artefacts are acceptable in the context of an online preview.

### Required changes

Basically :

* provide a UI to allow users to define their volume of interrest (cuboid) [use sliders with [dat](https://github.com/dataarts/dat.gui)],
* send the cuboid definition to the shader [by means of newly defined `UNIFORM`s],
* extend vertex shader GLSL code to discard any segment extending outside of the cuboid (at least one of its vertice is outside).

## Demos

* `demos/demo-before/` contains a standalone demo similar to the current preview available on Brain/MINDS data portal ([NA216 dataset](https://dataportal.brainminds.jp/marmoset-mri-na216)).

References:

* XTk Lesson **#06: Connectivity** http://lessons.goxtk.com/06/
* XTk Demo **FiberAtlas** http://demos.goxtk.com/brainfibers/

### How to run on your machine

#### Prerequisites

* git
* `npm` installed on your machine

#### 1. clone this repo

```sh
git clone https://github.com/bm-hackathon-23/tracto-preview
```

#### 2. Install small http server (run once)

Modern web-browsers don't allow loading local resources; Thus, a little http-server is needed for the demo to be able to load data.

```sh
cd tracto-preview/demos/
npm install
```

#### 3. Start http server

Change current dir to the directory of the demo you want to run.

For example:
```sh
cd tracto-preview/demos/demo-before
node_modules/http-server/bin/http-server . -p 8000
```

#### 4. Load http://127.0.0.1:8000/ in your web-browser.

#### 5. Stop http server

Press `[Ctrl]-c` to stop the server.
