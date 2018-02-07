# Linear rendering overview

The Unity Editor allows you to work in traditional gamma color space as well as linear color space. While gamma color space is the historically standard format, linear color space rendering gives more precise results. 

For further reading, see documentation on: 

* [Linear or gamma workflow](LinearRendering-LinearOrGammaWorkflow) for information on selecting to work in linear or gamma color space.
* [Gamma Textures with linear rendering](LinearRendering-GammaTextures) for information on gamma Textures in a linear workflow.
* [Linear Textures](LinearRendering-LinearTextures) for information on working with linear Textures.

## Linear and gamma color space

The human eye doesn't have a linear response to light intensity. We see some brightnesses of light more easily than others - a gradient that proceeds in a linear fashion from black to white would not look like a linear gradient to our eyes. 

![Left: A linear gradient. Right: How our eyes perceive that gradient. Note where the borders (which are exactly mid-grey) merge with the gradient in each case](../uploads/Main/LinearLighting-LinearGradients.png)

For historical reasons, monitors and displays have the same characteristic. Sending a monitor a linear signal results in something that looks like the gradient to the right in the illustration above, and simply looks wrong to our eyes. To compensate for this, a corrected signal is sent to make sure the monitor shows an image that looks natural. This correction is known as gamma correction. 

The reason both gamma and linear color spaces exist is because lighting calculations should be done in linear space in order to be mathematically correct, but the result should be presented in gamma space to look correct to our eyes.

When calculating lighting on older hardware restricted to 8 bits per channel for the framebuffer format, using a gamma curve provides more precision in the human-perceivable range. More bits are used in the range where the human eye is the most sensitive.

Even though monitors today are digital, they still take a gamma-encoded signal as input. Image files and video files are explicitly encoded to be in gamma space (meaning they carry gamma-encoded values, not linear intensities). This is the standard; everything is in gamma space. 

The accepted standard for gamma space is called sRGB (see [Wikipedia](https://en.wikipedia.org/wiki/SRGB)). This standard defines a mapping to linear space that allows our eyes to make the most of the 8 bits per channel of precision. Below is a diagram of this mapping.

![Image courtesy of Wikimedia. License: public domain](../uploads/Main/LinearRenderingMapping.png)

Linear rendering refers to the process of rendering a Scene with all inputs linear - that is to say, not gamma corrected for viewing with human eyes or for output to a display.