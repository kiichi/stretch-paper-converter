## Totally Unofficial Rendering Experiment of StretchPaper on Browser

Attempt to render StretchPaper App's drawing data on web browser (this is an awesome app if you have invested on Apple pencil 2 and iPad Pro, Good-Bye Dry Erase Board). This web conversion project is Totally Experimental and Unofficial.



- [Go to Demo Page](https://kiichi.github.io/stretch-paper-converter/)


## Screenshots

![Very simple Drawing][https://github.com/kiichi/stretch-paper-converter/blob/main/compare.png?raw=true]


## References


Full Credits and respects to the app develoepr + all those library + article authors:

- [Stretchpaper — The infinite canvas for iPad.](http://www.stretchpaper.com/)
- [javascript - Convert base64 string to ArrayBuffer - Stack Overflow](https://stackoverflow.com/questions/21797299/convert-base64-string-to-arraybuffer)
- [msgpack/spec.md at master · msgpack/msgpack](https://github.com/msgpack/msgpack/blob/master/spec.md) 
- [joeferner/node-bplist-parser: Binary plist parser.](https://github.com/joeferner/node-bplist-parser)
- [binary plist parser for Javascript (non-Node) - PIYO - Tech & Life -](https://blog.piyo.tech/posts/2014-05-08-203906/)
- [pi-chan/bplist-parser: Convert binary plist to JSON.](https://github.com/pi-chan/bplist-parser)
- [SVG.js v3.0 | Home](https://svgjs.com/docs/3.0/)
- [Smooth a Svg path with cubic bezier curves | by François Romain | Medium](https://medium.com/@francoisromain/smooth-a-svg-path-with-cubic-bezier-curves-e37b49d46c74)



## Process 

(my notes)

1. Export StretchPaper raw data (.stretch)
2. This is Base64 MessagePack format and it contains another binary plist contents for points
3. Load the .stretch file using MessagePack
4. Split by the new line char
5. Decode Base64 each line
6. Parse the line in MessagePack format.
7. There are points Uint16 chunk at offset 105. next byte is the length.
8. Decode Base64 for the next n bytes.
9. Get binary array into binary plist parser
10. Get array of array which contains points data
11. Each points consists of 5 values, and they repeats. [x, y, (z or inc counter), (pressure or stroke width), (0.25 something not sure)...]
12. Translate them into SVG. Using line / polyline function for now. For smoothing effect, Bezier in path should be ideal (?) [Smooth a Svg path with cubic bezier curves | by François Romain | Medium](https://medium.com/@francoisromain/smooth-a-svg-path-with-cubic-bezier-curves-e37b49d46c74)

## Other Notes

Maybe just use SigPad? 


My thought process:

1. looks like base64 lines
1. decoded
1. Got hint "MessagePack"
1. library didn't work but got "marker" information for each chunk
1. simple format, so parse it out manually from the library soruce code
1. look for useful chunk. found a string sequence "points"
1. looks like another base64 chunk
1. decoded, then found "bplist" sequence
1. it make sense to have Uint16 (xcode is all 16bits), iOS info.plist format probably quick and easy way to save from NSData like serialization
1. found bplist-parser library, but it's written in Node.js
1. Tried browserify but no luck since Buffer object is native to OS
1. Found ported browser version 
1. Tested import and figured it out to pass ArrayBuffer instead of native Node.js Buffer
1. Got structured json (array) of floats. 
1. Saw the patter that every 5 elements seems to be repeating. Assume some structure start with x,y, etc... 
1. Search for svg rendering library
1. Kind ok drawing using svg.line funciton. polyline should be faster but can not figure out dynamic width for each part
1. figured out command + - either collapse or keep them so that we can do redo on web


### Notes about command
- Offset = 1 is command. + means added points and - means erased object(s)
- If it's added, the GUID is stored in Offset 5 - 41.
- If it's -, then one eraser stroken can erase multiple GUID(s)
- Need collapse those before render, or it might be fun to do undo - redo stack on browser?

## TODO
- Parse other parts like meta information 
- figure out smooth curve with proper thickness
- collapse original lines so that subtract erased lines
