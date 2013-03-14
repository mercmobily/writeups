# Basic CSS

## Declare a stylesheet

### EXTERNAL:

    <head>
      <link rel="stylesheet" type="text/css" href="mystyle.css" />
    </head>

### INTERNAL:

    <head>
      <style type="text/css">
        hr {
          color:sienna;
        }
      </style>
    </head>

### INLINE:
    <p style="color:sienna;margin-left:20px">This is a paragraph.</p>

## Safe fonts and colours

 * Safe fonts: http://www.w3schools.com/cssref/css_websafe_fonts.asp
 * Colour names: http://www.w3schools.com/cssref/css_colornames.asp (`aqua`, `black`, `blue`, `fuchsia`, `gray`, `grey`, `green`, `lime`, `maroon`, `navy`, `olive`, `purple`, `red`, `silver`, `teal`, `white`, `yellow`)


## POSITIONING PARAMETERS: STATIC, RELATIVE, ABSOLUTE, FIXED

### STATIC AND RELATIVE POSITIONING:
 * Default, or position:static;
 * IF static,   top, bottom, left, right are NOT obeyed!
 * IF relative, top, bottom, left, right are obeyed!
   Remember: top, bottom, left, right will shift the block in that direction

 * block: (display:block)
   * Elements are placed vertically one after the other
   * margin:top and margin:bottom can be used to set margins

 * inline: (display:inline)
   * to be used for simple text only!
   * Elements placed horizontally, one after the other
   * A set of inline boxes are enclosed in a "line box", a set of inline boxes might be split into several "line boxes" (see: multi-lines)
   * The height of the line box can be set with line-height

 * inline-block (display: inline-block)
   *  Makes a block element inline (will display one after the other), but preserves block capabilities such as setting width and height, top and bottom margins, paddings etc.
   * Important when turning existing block elements to inline, or browser might assign 0 height if they contain no text for example

Code examples:

     /* Static: parameters top, bottom, left, right are NOT obeyed */
     .something {
       position: static;
       margin-top: 5em;      /* Note: in overlapping margins, the margin will be the greater of the two  */
       margin-bottom: 10em;
     }

     /* relative: parameters top, bottom, left, right ARE obeyed */
     .something {
       position: relative;
       margin: 5em;
       border: 1px solid #f00;
     }



### ABSOLUTE AND FIXED POSITIONING:
 * Both called "absolute" really
 * IF absolute, the "0x0" coordinate is the containing element which is either relative, absolute or fixed. SO,
   a "relative" element will be container block of absolute descendants, wherever they are in hiherarchy.
 * IF fixed, the "0x0" coordinate is the top left corner of the browser
 * Here top, bottom, left, right are used to specify distance from each edge of element to the corresponding edge of containing block. 
 * Can specify dimensions of the box using width and height (if not, the element will shrink-wrap to fit parent's contents).


## ALIGN TEXT CENTRE AND RIGHT:

### CENTRE

    .center {
      margin-left:auto;
      margin-right:auto;
      width:70%;
      background-color:#b0e0e6;
    }

### RIGHT

    .right {
      position:absolute;
      right:0px;
      width:300px;
      background-color:#b0e0e6;
    }




