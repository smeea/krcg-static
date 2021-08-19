# KRCG Static

[![Validation](https://github.com/lionel-panhaleux/krcg-static/workflows/Validation/badge.svg)](https://github.com/lionel-panhaleux/krcg-static/actions)
[![Python version](https://img.shields.io/badge/python-3.8-blue)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/License-MIT-blue)](https://opensource.org/licenses/MIT)
[![Code Style](https://img.shields.io/badge/code%20style-black-black)](https://github.com/psf/black)

KRCG static V:tES files generation, using
the VEKN [official card texts](http://www.vekn.net/card-lists),
the [Tournament Winning Deck Archive (TWDA)](http://www.vekn.fr/decks/twd.htm) and
[KRCG](https://github.com/lionel-panhaleux/krcg) rulings list.

This software is used to generate the static website [static.krcg.org](https://static.krcg.org).

Portions of the materials are the copyrights and trademarks of Paradox Interactive AB,
and are used with permission. All rights reserved.
For more information please visit [white-wolf.com](http://www.white-wolf.com).

![Dark Pack](dark-pack.png)

## Contribute

**Contributions are welcome !**

This is an offspring of the [KRCG](https://github.com/lionel-panhaleux/krcg)
python package, so please refer to that repository for issues, discussions
and contributions guidelines.

## Update the KRCG static website

-   Copy the translated card images in the `i18n_cards` folder, under their respective
    ISO 639-1 language code folder (ie. `i18n_cards/fr`)

-   Launch the `make static` command

The command build the static site into the `build` folder with the `krcg-static build`
command, then uses `rsync` over `ssh` to push it to the `static.krcg.org` server.
Note you need ssh access to the `krcg.org` server to deploy.

## Fontforge & Imagemagick foo

Use [Homebrew](https://brew.sh) on OSX to install a few graphical tools:

```
brew install imagemagick inkscape potrace
```

[Fontforge](https://fontforge.org) can be used to build and extract fonts.
Use `File > Execute Script` to execute the following `FF` script:

```txt
SelectWorthOutputting(); foreach Export("svg"); endloop;
```

Convert SVG to PNG with a transparent background:

```shell
for f in svg/**/*(.); do g=${f#svg/}; g=png/${g%.svg}.png; convert -background none $f $g; done
```

Convert transparent PNG to white bordered PNG:

```shell
for f in png/icon/*(.); \
do g=png_wb/${f#png/}; \
convert $f \( \
    +clone -alpha extract -morphology edgeout octagon:15 -bordercolor black \
    -border 100 -morphology close Octagon:50 -shave 100 \
    -level 50% -transparent black \) \
-compose DstOver -composite $g \
; done
```

Convert transparent square PNG to white bordered PNG:

```shell
for f in png/icon/*(.); \
do g=png_wb/${f#png/}; \
convert -background none -size 896x896 xc:white \
    \( $f -trim +repage -resize 768x768 \) \
    -geometry +64+64 -composite $g \
; done
```

Note as seen on real cards, inferior disciplines are roughly 15% bigger squares than superior disciplines.
This is because when one rotates the square by 45° the width is √2 = 41% larger,
so compensation is required for visual consistency.
If the base square (inferior) is `896x896`, the superior square is `896/1.15` = `780x780`.
And if the inner inferior icon is `768` in width (for a `64` white border),
then the superior icon width needs to be `768/1.15 * √2` = `944x944`.

Convert transparent losange (sup) PNG to white bordered PNG:

```shell
for f in png/icon/*(.); \
do g=png_wb/${f#png/}; \
convert -background none -size 780x780 xc:white -rotate 45 \
    \( $f -trim +repage -resize 944x944 \) \
    -geometry +82+82 -composite $g \
; done
```

Add transparent border (`128px`):

```shell
for f in png_wb/icon/*(.); do \
convert -background none $f -bordercolor none -border 128x128 $f \
; done
```

Generate a multisize `.ico` file:

```shell
convert -background none vtes.svg \
\( -clone 0 -resize 16x16 \) \
\( -clone 0 -resize 32x32 \) \
\( -clone 0 -resize 64x64 \) \
\( -clone 0 -resize 128x128 \) \
\( -clone 0 -resize 256x256 \) \
-delete 0 -alpha on -compress Zip vtes.ico     
```
