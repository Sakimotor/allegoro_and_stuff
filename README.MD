# Notes about Cleopatra Fortune and Allegro

Well, we want to port Cleopatra Fortune over DOS, and Allegro is a good starting point for doing that, let's see what we need to handle: 


## Allegro: Initializing the game

Any game in allegro can be initialized by writing the following at the beginning of the main:

```c
#include <allegro.h>

void main {

allegro_init();
install_timer();
install_keyboard();
...
}
```

We [initialize the graphics](https://liballeg.org/stabledocs/en/alleg008.html) with the following excerpt:

```c
/* we open a fullscreen application, and fallback to safe/text mode if it fails */
if (set_gfx_mode(GFX_AUTODETECT, w, h, 0, 0) != 0) {
      if (set_gfx_mode(GFX_SAFE, w, h, 0, 0) != 0) {
	 set_gfx_mode(GFX_TEXT, 0, 0, 0, 0);
	 allegro_message("Unable to set any graphic mode\n%s\n", allegro_error);
	 return 1;
      }
```

## Allegro: Loading Bitmaps

Allegro can easily [load a bitmap](https://liballeg.org/stabledocs/en/alleg010.html) (BMP, LBM, PCX, TGA) into memory:

```c
	  BITMAP *bmp;
      PALETTE palette;
      
	  /*the bitmap's color palette will be stored in the "palette" argument */
      bmp = load_bitmap("image.pcx", palette);
      if (!bmp)
         abort_on_error("Couldn't load image.pcx!");
      ...
	  /*we remove the bitmap from memory once we're done using it */
      destroy_bitmap(bmp);

```

Displaying it is then done by ["blitting"](https://liballeg.org/stabledocs/en/alleg014.html#blit) our bitmap into the actual screen

```c
       /* Blit src on the screen (a general BITMAP* variable that represents the display in memory. */
      blit(bmp, screen, 0, 0, 0, 0, bmp->w, bmp->h);
```

## Allegro: Loading a Palette

If we have a variable that stores a color palette in memory, we can set the screen's palette to it anytime:
```c	  
	  
	/* set the color palette, we'll use a custom one: default ones are desktop_palette (atari st palette), and default_palette (VGA DOS palette ?) */
   set_palette(pal);
```

Programs like [Fixpal](https://web.archive.org/web/20120426214537/http://gradha.sdf-eu.org/palopt/palopt.html) can be used to generate a single palette out of a whole collection of bitmaps, then saved to its own bitmap.

In order to use the generated palette.bmp file, we use the `load_bitmap` method mentioned above to store it into a variable.


## Allegro: Loading Fonts

[A font can be loaded](https://liballeg.org/stabledocs/en/alleg017.html) from a bitmap file, a .GRX format .fnt file or BIOS-format .fnt files

The [load_font](https://liballeg.org/stabledocs/en/alleg017.html#load_font) instruction can load any of the aforementioned types automatically. It stores the font palette in a variable passed as the second argument, if such an argument is given.

```c
FONT *myfont;
      PALETTE palette;
      myfont = load_font("my_font.pcx", palette, NULL);
      if (!myfont)
         abort_on_error("Couldn't load font!");
      ...
      destroy_font(myfont);
```

Allegro-compatible fonts can be made by using the [Allegro Font Editor](http://members.allegro.cc/miran/fonted.html)

The program seemingly runs on any Windows NT OS, and Windows 10. 
If a general palette has been made, as mentioned in the **Loading a Palette** section, it can be loaded by going to `Palette > Load` 
![image](https://i.imgur.com/HpLluAh.png)

A font of any size can be created by going to `File > New`, at which point we will be prompted to enter the width and height that our font is going to get.

![img](https://i.imgur.com/YDuoDTT.png)

Then, we can click into the palette on the right of the application (left-click is foreground color, right-click is background color) and start drawing our font, pixel by pixel. By setting a custom text in the **Preview Text** section, we can see how our font looks in a real situation.

![img](https://i.imgur.com/yqYB5ju.png)

Once our font is all made, we can save it in any of the formats supported by Allegro (pcx, bmp, tga, lbm) and use it in our games by going to `File > Save as`.

![img](https://i.imgur.com/ab8irpe.png)

## Allegro: Displaying Text

We can [display text](https://liballeg.org/stabledocs/en/alleg018.html) using the `textout` or `textprintf` routines. If a font was previously loaded into memory, it can be used here.
 
```c
/*SCREEN_W and SCREEN_H are macros that give us the current screen width and height as int.
white is the foreground color of the text, black is the backgroun color. */
textout_ex(screen, myfont, "This is my own pretty font!", SCREEN_W / 2, SCREEN_H / 2, white, black);
```

## Allegro: Datafiles

Allegro supports a special archive format known as **datafile**. Datafiles are created by a tool called [grabber](https://www.allegro.cc/depot/Grabber/), we can find its' manual on [this page](https://www.allegro.cc/manual/4/tools/grabber/). A complete guide is also available [here](https://liballeg.org/docs/ovehk_stl_tutorial_en/tut2.html).
The benefit of using the program is that it can compress most of the game's data in a way that would optimize our data, optimal for DOS games. It does so for MIDIs, bitmaps, palettes and most filetypes (raw files like custom binary data or wav samples can hardly be compressed but with bitmaps and fonts I've seen **up to 80%** of compression, going from 80 to 20kb !)

We call any element inside a Datafile an **Object**. 
Each object is given an index, and going through all of the objects in a Datafile works the same as going through an array. Example: if the Datafile contains a **FONT** object followed by a **BITMAP** object, then `datafile[0]` would be the font, and `datafile[1]` would be the bitmap.
Once loaded, an object's data is accessed by reading `object` 's  `dat` attribute.

Datafiles end with the `.dat` extension, and have a few routines that allow them to be loaded in Allegro.  

First of all, to **load the file in memory**, we call the [`load_datafile`](https://liballeg.org/stabledocs/en/alleg031.html#load_datafile) function with the filename as an argument. 

```c
/* Load the resources for our game. */
      DATAFILE *dat = load_datafile("game.dat");
      if (!dat)
         abort_on_error("Couldn't load sound resources!");
      /* Use resources. */
      ...
      /* Destroy them when we don't need them any more. */
      unload_datafile(dat);
```

When a datafile is loaded, every object we put inside can be retrieved the following way (here we put a BITMAP as the first object in the datafile:

```c
data=load_datafile("tutorial.dat");
BITMAP* image = (BITMAP*) data[0].dat;
```

Any object in a datafile can be given a name. In case we kept the name of our objects, we can retrieve it by using the [`find_datafile_object`](https://liballeg.org/stabledocs/en/alleg031.html#find_datafile_object) function. Example, let's say that we have a "STUFF" object somewhere in our datafile. We can then load it the following way:

```c
char*  stuff = "STUFF";
      DATAFILE *dat, *obj;
      ...
      
      obj = find_datafile_object(dat, stuff);
      if (!obj)
         abort_on_error("That stuff doesn't exist!");
```

We can also use [`load_datafile_object`](https://liballeg.org/stabledocs/en/alleg031.html#load_datafile_object) if we haven't loaded the whole datafile in memory yet and that we only want to retrieve a specific object:

```c
/* Load only the music from the datafile. */
      music_object = load_datafile_object("datafile.dat",
                                          "MUSIC");
      /* Play it and wait a moment for it. */
      play_midi(music_object->dat);
      ...
      /* Destroy unneeded music. */
      unload_datafile_object(music_object);
```

Finally, there is a [`load_dat_font`](https://liballeg.org/stabledocs/en/alleg017.html#load_dat_font) function that allows us to locate and easily load a font object from a datafile, as well as its related palette:

```c
FONT *f;
      PALETTE pal;
      char *names[img] = { "FONT_1_DATA", "FONT_1_PALETTE" }
      f = load_dat_font("fonts.dat", pal, names);
```

The **names** list works the following way: The first string in this list is the name of the font. You can pass NULL here to just load the first font found in the datafile. The second string can be used to specify the name of the palette associated with the font. This is only returned if the pal parameter is not NULL. If you pass NULL for the name of the palette, the last palette found before the font was found is returned. You can also pass NULL for param, which is treated as if you had passed NULL for both strings separately. In this case, the function will simply load the first font it finds from the datafile and the palette that precedes it. 


Let's take a closer look into the **grabber** software that actually creates our datafiles.
First time we open the software, we end up on a menu where a new "datafile"/archive  project is already open. 
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872574478749696/image.png?ex=66ac5819&is=66ab0699&hm=ec185929040606b6b170e2ebc5efe0f7d7b5fd222a97277914cff99560cc809c&)
By clicking  `Object > New` (or by right-clicking the white space),  we can add data into our datafile. Today, we're interested into making a colored font work, so we're going to add it. 
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872574725947402/image.png?ex=66ac5819&is=66ab0699&hm=daea804d12844411bc266f6f5713c9b24552fc4d93b95166a78fca3ac545d60a&)
Out of convenience, we can add the palette before the font, but we can do those steps in any order ! First, let's create a new `Palette`.

You might notice that when we create a new object, it doesn't ask we for a path, only for a name. That's because here we only declare the existence of a palette in our datafile. 
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872574981931110/image.png?ex=66ac5819&is=66ab0699&hm=f90e0550593a2b4dfdf967c13e8eaabc67a6432bfa5446feceb6d97b26bd01e5&)
To actually load our bitmap and derive a palette out of it, we need to press `OK` then right-click the newly-created declaration, 
 ![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872575346966578/image.png?ex=66ac5819&is=66ab0699&hm=af76a109a3330eb6aa02111241d48310405c887498a21308843fc133496c4281&)
 and click `Grab` (yes, that's where the software name comes from ! ), which will open an old-school file explorer where we'll be asked to load a font-compatible file.
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872575699157024/image.png?ex=66ac5819&is=66ab0699&hm=5337bdb1eb9b52a2a348eeaa573581ff6c01590e1986889ec29de41a0e22316a&)
 Click "OK" on the file we want to use, and *voila*, our own palette, ready to be used :D
 ![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872576093294652/image.png?ex=66ac581a&is=66ab069a&hm=48fcaa7fd871bfd1474894c100a66ca66c1035e3eb8f8cde3adc047e39d1acb5&)

Reproduce the same exact steps with the `Font` as a new  type,
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872576437485588/image.png?ex=66ac581a&is=66ab069a&hm=6ca46a2dbb2c52808c509a0351cd7ed78ccbf9341ce67c21f9e98b06b8ce7859&) 
and we end up with a cool-looking clairfont that will load in our gamefiles as long as we write ![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872576730955787/image.png?ex=66ac581a&is=66ab069a&hm=692f45c8220e1c6b3629d0bdf45417607c66f8b78f446b415ab72a59448868f2&) 

Enable `Global Compression` in the top-right corner of the app, then go to `File > Save stripped > Strip grabber information`. 
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872577057984632/image.png?ex=66ac581a&is=66ab069a&hm=b268eb4be9dc4fc600c4a3d89c63d7b295fec719e755e67266ce7626cb2aec9e&)
We should end up with a file 4x smaller than if we used the bitmap as-is, yay
![img](https://cdn.discordapp.com/attachments/1255497743103823913/1267872577452507236/image.png?ex=66ac581a&is=66ab069a&hm=c519ff5ce006d1ef78ed228289f0f4ce855c96458331f44fc01944855bfa253e&)


## Cleopatra: Score 

The Scoring logic needs to be reverse-engineered (since no one wrote about it somehow), we'll use the PS1 version for the sake of convenience.



