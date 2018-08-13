# Grid Parser

I found this [grid](https://static.tjctf.org/ecdb2ff56241299271bf44268880e46a304f50d212ae05dab586e3843ad59d50_movies.grid) while doing some parsing. Something about it just doesn't seem right though..

## What do we got here?

Opening the file with a hex editor quickly shows us it's some kind of zip file. Opening it with 7zip reveals a familiar structure: a _rels and a xl folder as well as a Content_Types.xml file. It's clear this is some kind of spreadsheet in oooxml format. Sadly, simply renamin the extension to xlsx doesn't make Excel love it more and we can't open it. But we get a pretty good of the structure fo the file by browsing the structure.

## The solve
I shamefully admit I spent _way_ too much solving this one. I reconstructed the file by creating a new excel book and replacing the contents with the challenge file contents and was rewarded with a very nice in-depth database of the marvel cinematic univers movies. Hwoever a quick glance didn't reveal anything out of the ordinary. 

I went back to the original zip and found a png file in the ```xl/media``` folder which, unsurprisingly, revealed itself to be a zip as well (advantage of using 7zip: just keep clicking, it will recognize embedded archive files), containing a single, encrypted ```flag.txt``` file. I extracted/split the png and realised that the image displayed to crudely drawn asterisks.

## DIY jacktheripper

There have to be a lot of better ways to solve this but I decided to quickly write my own brute forcing script, under the assumption that the password really was 2 chars (it actually works relatively well for up to 4 chars). It found it after a few seconds revealing the flag.

tjctf{n0t_5u5_4t_4LL_r1gHt?}
