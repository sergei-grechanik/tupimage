# Tupimage

`tupimage` is a utility for uploading and displaying terminal images using the
Unicode placeholder extension of the kitty graphics protocol. The Unicode
placeholder extension allows placing images using a special placeholder
character, which makes this approach suitable for text-based applications like
tmux and vim, which know nothing about the graphics protocol.

Currently the Unicode placeholder extension is supported only by the following
terminals:
- [this fork of Kitty](https://github.com/sergei-grechanik/kitty/tree/unicode-placeholders)
- [this fork of st](https://github.com/sergei-grechanik/st/tree/graphics)

![displaying images in tmux](./tupimage-tmux.gif)

## Installation

Just copy `tupimage` somewhere on your PATH. The script is written in bash (uses
bash-specific features) and requires some utilities, like bc for computing the
best number of rows/columns and ImageMagick for reading image info and for image
conversion.

In the future I'll probably rewrite it in python since it became too complex for
a bash script.

## Usage

Just pass the file name to tupimage:

    tupimage image_file

It will automatically compute the optimal number of rows and columns, choose the
best uploading method, and show the uploading progress. The number of rows or
columns may be specified manually, the image will be resized to fit the box
while preserving the aspect ratio:

    # Fit the 20x10 box.
    tupimage image_file -r 10 -c 20

    # The optimal number of columns will be computed automatically.
    tupimage image_file -r 10

### Help

Run `tupimage -h` to display a brief help and `tupimage --help` to see all
available options and environment variables.

### Automatically computing the number of rows/columns

When the number of rows and columns are not provided, tupimage tries to compute
the best values automatically using the formula:

    columns = round(image_width_in_inches  / TUPIMAGE_COLS_PER_INCH)
    rows    = round(image_height_in_inches / TUPIMAGE_ROWS_PER_INCH)

`TUPIMAGE_COLS_PER_INCH` and `TUPIMAGE_ROWS_PER_INCH` are environment variables
that are supposed to be set in `.bashrc` or a similar place. They can be
floating point. One way to compute them is to run the command
`tupimage --get-cells-per-inch`, which will divide the value of `Xft.dpi` by
cell dimensions in pixels queried from the terminal. You can even include
something along these lines in your `.bashrc` (at your own risk):

```shell
if [ -z "$TUPIMAGE_COLS_PER_INCH" ] || [ -z "$TUPIMAGE_ROWS_PER_INCH" ]; then
    cols_and_rows_per_inch="$(tupimage --get-cells-per-inch)"
    export TUPIMAGE_COLS_PER_INCH="$(cut -d ' ' -f 1 <<< "$cols_and_rows_per_inch")"
    export TUPIMAGE_ROWS_PER_INCH="$(cut -d ' ' -f 2 <<< "$cols_and_rows_per_inch")"
    # Uncomment to display images as close as possible to its pixel size:
    # export TUPIMAGE_OVERRIDE_PPI="$(cut -d ' ' -f 3 <<< "$cols_and_rows_per_inch")"
fi
```

And you may also want to include this into your `.tmux.conf`:

    set -ga update-environment "TUPIMAGE_COLS_PER_INCH TUPIMAGE_ROWS_PER_INCH TUPIMAGE_OVERRIDE_PPI"

Note that currently there is no way to display the image in its true size
(either in terms of pixels or inches), but this might be implemented in the
future (TODO).

###  Image resolution (PPI/DPI)

By default tupimage respects the PPI (pixels per inch) of an image. This is not
always desirable (most applications, including web browsers, don't do that). The
PPI of an image can be ignored by overriding it with another PPI value using
either the option `--override-ppi` or the variable `TUPIMAGE_OVERRIDE_PPI`. If
you want to display images as close as possible to their pixel size, this value
should be the same as the one used to compute the number of cols and rows per
inch (i.e. the value of `Xft.dpi` if you use `--get-cells-per-inch`, see the
previous section).
