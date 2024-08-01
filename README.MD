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

## Allegro: Displaying Text

We can [display text](https://liballeg.org/stabledocs/en/alleg018.html) using the `textout` or `textprintf` routines. If a font was previously loaded into memory, it can be used here.
 
```c
/*SCREEN_W and SCREEN_H are macros that give us the current screen width and height as int.
white is the foreground color of the text, black is the backgroun color. */
textout_ex(screen, myfont, "This is my own pretty font!", SCREEN_W / 2, SCREEN_H / 2, white, black);
```

## Allegro: Datafiles

Allegro, while not bundled by default with it, 


## Cleopatra: Score 

The Scoring logic needs to be reverse-engineered (since no one wrote about it somehow), we'll use the PS1 version for the sake of convenience.



