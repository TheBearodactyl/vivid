# vivid 🌈
A simple-to-use `cpp` color library

- color space **conversions**
- perceptual color **interpolation**
- popular and custom **color maps**
- **xterm** names and **ansi** codes
- ansi **escape sequences** and **html** encoding

```cpp
using namespace tq;

//  create and interpolate colors
col_t c1 = rgb::fromName( "indianred" );
col_t c2 = rgb::fromHsl( { 0.f, 0.4f, 0.5f } );

col_t interp = rgb::lerp( c1, c2, 0.5f );
std::string hex = hex::fromRgb( interp );

//  quick access to popular colormaps for data visualization
auto cmap = ColorMap::loadDefault( ColorMap::Viridis );
col_t mid = cmap.at( 0.5f );

//  ansi and html encodings
std::cout << tq::ansi::fg( 163 ) << "woah!!" << tq::ansi::reset;
fout << tq::html::bg( "#abc123" ) << "styled background color" << tq::html::close;
```

##  Content

<!-- TOC depthFrom:2 depthTo:2 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Content](#content)
- [Motivation](#motivation)
- [Get Started](#get-started)
- [Dependencies](#dependencies)
- [Color Spaces](#color-spaces)
- [Interpolation](#interpolation)
- [Color Maps](#color-maps)
- [Encodings](#encodings)

<!-- /TOC -->


## Motivation

Things we create should be beautiful. Be it a console log message or a real-time volumetric data plot. I'm working with colors quite often, but found the available means to work with them lacking. Especially if you want to just get stuff (he said stuff) done and your ideas down in code. Over time, I gathered all the little snippets and helpers I had created, and thus this project was born.

`vivid` allows you to quickly create, lookup and convert colors. It provides perceptual color interpolation, easy access to color names, ascii escape codes and to some of the great data visualization color palettes out there.


## Get Started

```bash
git clone git@github.com:gurki/vivid.git
git submodule update --init
```

This repository comes with support for ~~both~~ `Qt` (_vivid.pri_) ~~and `cmake` (_CMakeList_)~~ projects.
You can start by simply opening up `examples/qmake/vivid.pro` in `Qt Creator`.


## Dependencies

`vivid`  depends on a small number of header-only libraries, which are included as submodules.

- Nlohmann's great [Json for Modern C++](https://github.com/nlohmann/json) to load color names and color maps
- [OpenGL Mathematics (GLM)](https://github.com/g-truc/glm) for vector type and operations


## Color Spaces

`vivid` uses different types to represent colors, based on their attributed color space. The default `rgb`-space uses `glm::vec<3, float>`, which is accessible via `tq::col_t`. For `RRGGBB` byte-typed colors, `glm::vec<3, uint8_t>` or `tq::col8_t` is used.

The library has an extensive set of direct conversion routines. Additionally, there's a bunch of shortcuts for multi-step conversions. I haven't included all possible conversions. Rather, the functions that are currently included have been of use to me at least at once upon a time.

Conversions are built in a functional way, where colors get passed through converters, yielding new colors in different spaces. The user must (to some degree) ensure the integrity of the data passed into a converting function. E.g. `tq::xyz::fromLab` indeed assumes, that it is handed a `tq::col_t`, whose data encodes a valid `lab` color representation.

### Native Conversions

The following native conversions are currently available.

    hcl ← lab
    hex ← rgb8, index
    hsl ← rgb, index
    hsv ← rgb
    index ← rgb8, name
    lab ← xyz, hcl
    name ← index
    rgb ← rgb8, hsv, hsl, xyz
    rgb32 ← rgb, hex
    rgb8 ← rgb, rgb32, index
    xyz ← lab, rgb


## Interpolation

```cpp
//  pseudo-code to generate the images in this section
for ( auto& pixel : image ) {
    const float t = pixel.x / image.width;
    const auto col = tq::rgb::lerpHcl( c1, c2, t );
    image.setColor( pixel, col );
}
```

Color interpolation is an interesting topic. What should the color halfway in-between <span style="color:rgb(178, 76, 76)">red</span> and <span style="color:rgb(25, 153, 102)">green</span> look like? There is a great article introducing this topic by Grego Aisch [^1]. In order to do a perceptually linear transition from one color to another, we can't simply linearly interpolate two _RGB_-vectors. Rather, we move to a more suitable color space, interpolate there, and then move back again. Namely, we use the _CIE L\*C\*h_ space, also known as _HCL_, which matches the human visual system rather well. There are more suitable color spaces nowadays to do so, but _HCL_ has a nice balance between complexity (code and computation) and outcome.

Compare the images in the table below to get an idea of interpolating in different color spaces.

Color Space   | Linear Interpolation
--------------|-------------------------------------------------------------------
RGB           | ![lerp-rgb](docs/images/interpolations/lerpRgb.png)
HCL           | ![lerp-cielch](docs/images/interpolations/lerpHcl.png)
HSV           | ![lerp-hsv](docs/images/interpolations/lerpHsv.png)
HSL (Clamped) | ![lerp-hsl-clamped](docs/images/interpolations/lerpHslClamped.png)

[\^1] [Grego Aisch (2011) - How To Avoid Equidistant HSV Colors](https://www.vis4.net/blog/2011/12/avoid-equidistant-hsv-colors/)


## Color Maps

`vivid` comes with a set of pre-defined color maps, which I conveniently gathered under one umbrella. Thanks to the awesome community out there for their great work! [\^2,\^3]

As shown in the example in the beginning, it's quick and easy to query colors from a certain color map. You can also create your own maps by simply loading an according _\*.json_ file.

```cpp
//  loading a custom color map
auto cmap = ColorMap::loadFromFile( VIVID_ROOT_PATH "res/colormaps/mycolormap.json" );
auto mid = cmap.at( 0.5f );
```

Name        | Image
------------|----------------------------------------------------
Inferno     | ![inferno](docs/images/colormaps/inferno.png)
Magma       | ![magma](docs/images/colormaps/magma.png)
Plasma      | ![plasma](docs/images/colormaps/plasma.png)
Viridis     | ![viridis](docs/images/colormaps/viridis.png)
Vivid       | ![vivid](docs/images/colormaps/vivid.png)
Rainbow     | ![vivid](docs/images/colormaps/rainbow.png)
Hsl         | ![hsl](docs/images/colormaps/hsl.png)
Hsl Pastel  | ![hsl-pastel](docs/images/colormaps/hsl-pastel.png)
Blue-Yellow | ![vivid](docs/images/colormaps/blue-yellow.png)
Cool-Warm   | ![vivid](docs/images/colormaps/cool-warm.png)  |  

[\^2] [Stefan & Nathaniel - MPL Colormaps](http://bids.github.io/colormap/) <br>
[\^3] [SciVisColor](https://sciviscolor.org/)


## Encodings

`vivid` provides encodings for **ansi** escape codes (pretty console <3) and **html** using spans.

### ANSI

You can colorize console messages very simply using the `tq::ansi` helpers.

```cpp
std::cout << tq::ansi::fg( 136 ) << "and tada, colorized font" << tq::ansi::reset;
```

To get an overview of all color codes or quickly check if your console has 8-bit color support, you can call `printColorTable()` (shoutout to Gawin [^4] for the layout idea).

![colortable](docs/images/console/colortable.png)

Escape codes can also be used in conjunction with `ColorMaps` to create some joyful effects.

```cpp
const auto rainbowMap = tq::ColorMap::fromPreset( tq::ColorMap::PresetRainbow );
const std::string text = "How can you tell? - Raaaaaaiiiinbooooooowwws.";
std::cout << tq::ansi::colorize( text, rainbowMap ) << std::endl;
```

![rainbows](docs/images/console/rainbow-text.png)


### HTML

One of my side projects is a tagged logging system, where one of the sinks goes to html. This has become very handy.

```cpp
auto col = tq::rgb8::fromName( "LightSteelBlue" );
fout << tq::html::fg( col ) << "colorized html text!" << tq::html::close;
//  <span style='color:rgb(175, 175, 255)'>colorized html text!</span>
```

[^4] [Gawin's xterm color demo](https://github.com/gawin/bash-colors-256)
