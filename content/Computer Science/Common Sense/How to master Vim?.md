First install vim, then open your terminal and run:

```bash
vimtutor
```

Finish the tutorial, great! Now you have enough knowledge to use vim well in IDE with extension. But I am sure that is not what you want, right? You want to be a PRO, who uses only text editors and gets rid of modern powerful IDE.

Then I think we should dive into buffers or maybe try lazyvim to bring more power.

# Buffers

In vim, every file you opened is called a buffer, which is an exact copy of the file. We can open new files inside vim use command: `:e filename`. You will find that the new buffer just replaces the previous buffer, and you can find it! No! I haven't write my changes. Don't worry, that buffer is not closed.

You can use command `:ls` which is the same as `:buffers` to check what buffers are still opened.

Then you can use command `:b filename` to go back to it. Also there is an id for each buffer, you can use that to navigate too.

There are also relative moving methods like `:bn` and `:bp` where `n` and `p` stands for next and previous.

To view multiple buffers at the same time, we can use commands: `:sb` to split buffer horizontally, for vertical split, we can use `:vsp`.

To navigate between appearing buffers, we can use `Ctrl+w` and `h,j,k,l` to move around or `Ctrl+w w` to move to the next window.
