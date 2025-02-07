# FlameGraph

生成火焰图分为 3 步

1. 捕获堆栈，可使用 `perf record -F 1000 -a -g -p` 和 `perf script > out.perf`

2. 折叠堆栈，可使用 stackcollapse-perf.pl，如 `./stackcollapse-perf.pl out.perf > out.folded`

3. 调用 flamegraph.pl

```sh
./flamegraph.pl [options] infile > outfile.svg

--title TEXT     # change title text
--subtitle TEXT  # second level title (optional)
--width NUM      # width of image (default 1200)
--height NUM     # height of each frame (default 16)
--minwidth NUM   # omit smaller functions. In pixels or use "%" for 
                 # percentage of time (default 0.1 pixels)
--fonttype FONT  # font type (default "Verdana")
--fontsize NUM   # font size (default 12)
--countname TEXT # count type label (default "samples")
--nametype TEXT  # name type label (default "Function:")
--colors PALETTE # set color palette. choices are: hot (default), mem,
                 # io, wakeup, chain, java, js, perl, red, green, blue,
                 # aqua, yellow, purple, orange
--bgcolors COLOR # set background colors. gradient choices are yellow
                 # (default), blue, green, grey; flat colors use "#rrggbb"
--hash           # colors are keyed by function name hash
--cp             # use consistent palette (palette.map)
--reverse        # generate stack-reversed flame graph
--inverted       # icicle graph
--flamechart     # produce a flame chart (sort by time, do not merge stacks)
--negate         # switch differential hues (blue<->red)
--notes TEXT     # add notes comment in SVG (for debugging)
--help           # this message

./flamegraph.pl --title="Flame Graph: malloc()" trace.txt > graph.svg
```