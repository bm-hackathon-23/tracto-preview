# Hackhathon topic - Improving Tractography preview for better data exploration

## Context

On the dataportal (see NA216 dataset at https://dataportal.brainminds.jp/marmoset-mri-na216), tractography data can be inspected with an online previewer leaveraging XTk (https://github.com/xtk/X).
In practice, the rendered data often looks like a dense ball of fibers, and that's not so to explore the fibers details, especially for the inner regions which are  obscured by outer regions.

Indeed, with Xtk it's only possible: 
* to change the opacity of all fibers (of a trk file) at once, 
* to hide some of them depending on their length (by setting min and max fibers length filter values).


## Idea

It would be good to somehow be able to only view fibers inside a user-defined region of the space.
(Tanaka-sensei proposal "three othogonal guide 'planes' of some thickness. Center and thickness can be specified in the UI with sliders")
Ideally by continuing using Xtk  which is currently used by the online preview.



