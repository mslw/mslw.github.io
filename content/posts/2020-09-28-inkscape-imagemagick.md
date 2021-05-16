---
layout: post
title: "Inkscape + ImageMagick: a powerful combination"
author: Michał Szczepanik
tags: ['software', 'tips & tricks']
---

I can't recommend Inkscape enough, either for assembling figures or for building posters. However, Inkscape on its own does not export to tiff. Luckily, this can be accomplished with ImageMagick. Here's a short recipe, and a sort of laudation for both programs.

Recently I had to rework a figure with a schematic of an experimental design. The first version was put together in PowerPoint, but we somehow lost the ungrouped source file, so I had to start from scratch. Two pieces of free software made the job easy: Inkscape and ImageMagick.

It's easy to start with an A4 page, to have a sense of proportions in print (the graphics are vector, so scaling is not much of an issue anyway). Before saving, it might be useful to limit the page size to content and margins, by going to File → Document Properties (Shift+Ctrl+D) → Page Tab → Resize page to content.

Following that, the conversion can be performed as follows (an updated invocation, based on this [gist](https://gist.github.com/matsen/4263955) by matsen). It goes from svg through png to tif, but there is little apparent loss of quality, if any:

```
inkscape --export-filename new.png --export-dpi 300 file.svg
convert -compress LZW -alpha remove new.png new.tif
```

As a side note, clipping to content and adding margins could theoretically be incorporated in the export call, as the CLI has `--export-margin` and `--export-area-drawing` options, but as of the day of writing they are unsupported; see [this issue](https://gitlab.com/inkscape/inkscape/-/issues/1142).

Tried with Inkscape 1.0 and ImageMagick 6.9.11-22.
