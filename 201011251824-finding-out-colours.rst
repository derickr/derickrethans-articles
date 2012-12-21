Analysing Colours in an Image
=============================

.. articleMetaData::
   :Where: London, UK
   :Date: 2010-11-25 18:24 Europe/London
   :Tags: blog, php, opensource

For a project_ that I am working on, I had to figure out which colours are
primarily used in an image so that it can be indexed for sorting. Obviously,
there is only a small finite set of colours that we are interested in.  This
article shows on how I went about analysing an image to see which of those
colours were primarily used in the image.

First of all, I had to define a palette that I wanted to index on. Then
I needed to check the image to see which colours existed, and map those to
my finite set of colours. Then we could count them, and select the ones
that were used most.

.. _project: http://plumwillow.com

Picking the palette of colours wasn't very simple especially because
pink and purple are difficult to differentiate, as well as orange and brown.
It required a little bit of effort, but we settled on the following colours:

.. image:: /images/content/pw-colours.png

Some of the colour names have multiple colour values attached to them; this was
so that we can tune the analysing algorithm a little bit to provide better
results.

In order to match the colours from the image to the palette, I needed to
create a mapping algorithm. It's difficult to do this with RGB values because
they don't have the natural properties that we humans use for colours. However,
the HSV model works a lot better. A HSV colour 'wheel' looks like:

.. image:: /images/content/hsv-normal.png

On the X-axis we have the hue_ — a value that describes *which* colour it is on
a scale from 0 to 360 (°). On The Y-axis we have both the saturation_ — a value
that describes the intensity of a colour and value_ — the value describing the
lightness of a colour. The respective six blocks on the y-axis correspond with
saturation values of 0.0, 0.2, 0.4, 0.6, 0.8 and 1.0, and each of the ten bands
of each block with the value values of 0.0, 0.1, 0.2 ... 0.9 and 1.0.

.. _hue: http://en.wikipedia.org/wiki/Hue
.. _value: http://en.wikipedia.org/wiki/Lightness_(color)
.. _saturation: http://en.wikipedia.org/wiki/Saturation_(color_theory)

Then I cheated and used `The Gimp`_ to change this image of the colour wheel
to an indexed image, using the palette from above. The option for that is:
Image -> Mode -> Indexed -> "Use Custom Palette" and then pick your palette.
Don't forget to change it back to RGB before you save it though.  The resulting
image then looks like:

.. image:: /images/content/hsv-normal-palette.png

The script to draw this palette is::

    <?php
    require 'colortools.php';

    $img = imagecreatetruecolor( 720, 720 );

    $hFactor = 2;
    $hStep = 15;
    $vStep = 0.10;
    $vFactor = 120;

    for ( $h = 0; $h < 360; $h += $hStep )
    {
        for ( $s = 0; $s < 1.01; $s += 0.2 ) 
        {   
            for ( $v = 0; $v < 1.01; $v += $vStep )
            {   
                colorComponent::hsvToRgb( $h, $s, $v, $r, $g, $b );
                $c = imagecolorallocate( $img, $r * 255, $g * 255, $b * 255 );
                imagefilledrectangle( 
                    $img,
                    $h * $hFactor, $s * 600 + $v * $vFactor, 
                    ($h + $hStep) * $hFactor - 1, $s * 600 + ($v + $vFactor) - 1,
                    $c 
                );  
            }
        }   
    }       
            
    imagepng( $img, $argv[1] ); 
    ?>

.. _`The Gimp`: http://gimp.org

Each square's coordinates represent a mapping between (rounded) H, S and V 
values. With a little script, we can generate a mapping table::

    <?php
    function colorToRgb($color, &$r, &$g, &$b)
    {
        $r = ($color >> 16) & 0xFF;
        $g = ($color >> 8) & 0xFF;
        $b = $color & 0xFF;
    }

    $img = imagecreatefrompng( '/tmp/hsv-normal-palette.png' );

    $hFactor = 2;
    $vFactor = 12;

    echo "<" . "?php\nclass colorMap {\n\tpublic \$map;\n\n";
    echo "\tfunction __construct() {\n\t\t\$this->map = array(\n";

    for ( $h = 0; $h < 360; $h += 15 )
    {
        for ( $s = 0; $s < 5; $s += 1 )
        {
            for ( $v = 0; $v < 10; $v += 1 )
            {
                $c = imagecolorat( $img, $h * $hFactor, $s * 120 + $v * $vFactor );
                colorToRgb($c, $r, $g, $b);

                printf( "\t\t\t'x%03x%01x%01x' => '#%02x%02x%02x',\n",
                    $h, $s, $v, $r, $g, $b
                );
            }
        }
    }

    echo "\t\t);\t}\n}\n";
    ?>

It would be better if we had the algorithm to convert from a random HSV value
to a HSV value matching the closest colour in the palette. That's something for
a next stage though.

The mapping table is used to find out which colours of our palette
are actually part of the image. To sample all pixels in an image would take
too much time, so we restrict ourselves to 200x200 samples, making 40000
pixels.

For each sampled pixel we:

- find the RGB values
- convert the RGB values to HSV (unless it's transparent, then we skip it)
- normalize the HSV values by rounding the values
- map the rounded value with our mapping key to produce the RGB hex value
  from our palette
- count how much of each of the RGB hex values we've gotten
- create an array where the number of pixels normalised to each RBG hex value
  is larger than 5%

The HSV-to-normalized-key algorithm looks like::

    $key = sprintf('x%03x%01x%01x',
        max(0, min(345, floor( $h / 15 ) * 15)),
        min(4, floor($s * 5)),
        min(9, floor($v * 10))
    );

To visualise this process, I've created a before and after image:

.. image:: /images/content/phonebooth-normal.png

Sadly, as you can see, our typical red phone box isn't quite detected as being red, 
but rather black and grey. For some reason the conversion from an arbitrary HSV value
to a palette colour with The Gimp isn't quite as good as we've hoped. So instead of
drawing a correct HSV colour wheel, I decided to boost the colours a little bit away
from grey by modifying the S and V values. This caused the following change in
the palette-drawing script::

    $sR = 1 - ( (1-$s) * (1-$s) );
    $vR = 1 - ( (1-$v) * (1-$v) * (1-$v) * (1-$v) );
    colorComponent::hsvToRgb( $h, $sR, $vR, $r, $g, $b );

In a diagram, this looks like:

.. image:: /images/content/hsv-corrections.png

We use this new algorithm, to redraw a new HSV wheel:

.. image:: /images/content/hsv-boost.png

And with The Gimp create a new palette:

.. image:: /images/content/hsv-boost-palette.png

This new palette we then scan to create the color map out off. With this
new color map we re-analyse the image to count our primary image colours. The
result is then:

.. image:: /images/content/phonebooth-boost.png

Which tells us that red was the most prevalent colour with over 50% of the sampled
pixels (13916 + 7822 out of 40000), grey being second with 35% and black with 19%.
Brown, orange and green also make up more than 5% of the total amount of
sampled pixels.
