Playstation3: Project Settings
==============================


Unity for PS3 comes with additional project settings.

###Build Settings: General

1. [Build Type](#BuildType)

###Player Settings: Publishing Settings

1. [Title Config](#TitleConfig)
1. [DLC Config](#DLCConfig)
1. [Thumbnail (PNG 320x176)](#Thumbnail)
1. [Background (PNG 1980x1080)](#Background)
1. [Sound (ATRAC)](#Sound)
1. [Trophy Package](#Trophy)
1. [Title Settings](#TitleSettings)

<a name="BuildType"></a>
Build type
----------

PC Hosted

1. Creates a package to be launched from the PC
1. Option to link with development player ( for player connection, profiler and full TTY output)

PSN Submission

1. Creates a submission ready PKG file
1. Links with retail libs ( no development player, profiler or TTY output)


Disc Submission

1. Creates a package that can be used to create BluRay images (with ps3gen)
1. Links with retail libs ( no development player, profiler or TTY output)

<a name="TitleConfig"></a>
Title Config
------------

This is a __parameter description file__ describing the specifications of the Title, and is required to generate the .sfo system file. Details of how to create the file can be found in the __Disc Image Generator__ documentation : (https://ps3.scedev.net/projects/ps3gen) in the __Generator_Tools-Users_Guide_e.pdf__ > __Appendix C__ > __Parameter Description File (*.sfx) Specifications__.

<a name="DLCConfig"></a>
DLC Config
----------

This is a text file that specifies the parameters required for creating the NPDRM package. More information is available here: https://ps3.scedev.net/docs/ps3-en,NP_DRM-Package_Requirements,Creating_the_NPDRM_Package/1/

<a name="Thumbnail"></a>
Thumbnail (PNG 320x176)
-----------------------

 This icon is used by all content types. It's a 32-bit or 24-bit PNG, with dimension 320 x 176 pixels.
More information available here: https://ps3.scedev.net/docs/ps3-en,Content_Information-Specifications,Still_Image_Icon/1

<a name="Background"></a>
Background (PNG 1980x1080)
--------------------------

 The content information background image is an image file that is displayed when the focus moves to the content.
More information available here: https://ps3.scedev.net/docs/ps3-en,Content_Information-Specifications,Content_Information_Background_Image/

<a name="Sound"></a>
Sound (ATRAC)
-------------

An audio file that is played back when the focus moves to the content.

Format: 

* ATRAC3plus™
* Sampling frequency: 48 kHz
* Channel: 1ch or 2ch
* Peak volume: less than -12dB recommended

More information available here: https://ps3.scedev.net/docs/ps3-en,Content_Information-Specifications,BGM/#Audio_Format

<a name="Trophy"></a>
Trophy Package
--------------

Trophy Pack file with the trophy information for the game. The file can be generated with the Trophy Pack File Utility.
You will have to fill in the required Trophy Communication ID and Trophy Communication Signature given to you by SCE.

More information available here: https://ps3.scedev.net/docs/ps3-en,Trophy_System-Overview/

<a name="TitleSettings"></a>
Title settings
--------------

* **Trial mode**: Specifies whether the title is a trial or a full game.
The amount of memory reserved for MONO and for loading System Modules (SPRX) at runtime. The game scripts & mono runtime will allocate from this pool.
_Note: If the memory setting is too low you will see an error similar to this in the TTY console: prx.c:sys_prx_load_module:Error: res(id)=0x80010004, in which case the solution is to increase the System Modules Reserve value_
* **Max Save Game size (KB)**: Specifies the maximum HDD space in KB required by the title when no save games are present (Part of TRC R224)
