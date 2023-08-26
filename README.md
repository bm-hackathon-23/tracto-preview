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

* The shaders already includes mechanisms to adjust rendering of loaded objects depending on the value of user-defined criteria.
For example, in case of tractography data, some individual fibers may be masked depending on their overall length.

* Fiber files in `.trk` format are parsed by [`parserTRK.js`](https://github.com/xtk/X/blob/master/io/parserTRK.js). Each fiber segment is assigned a few _scalars_, including the overall length of the fiber they belong to. This enables the shader to discard the rendering of segments depending the length of their fiber.

### Tentative

In a first approach, we could probably obtain the desired effect of volume cropping by simply preventing the rendering of fiber segments that extend outside the user-defined volume of interest (similar logic as length based fiber masking described above).

This would work if fibers are composed of many short-length line segments (which seems a reasonable assumption since in practice fibers tend not to have long straight stretches).
Because in this first approximation the fibers would not be strictly cut at the volume edges (but rather at their next contained segments boundaries), the volume boundaries might not appear neat.
Moreover, we have to expect that when users define very thin volume, the number of rendered fibers might become artificially very low; We'll need to asses whether these artefact are acceptable in the context of an online preview.


