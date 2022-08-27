# Writing Simple-SAM

The [SAM file format](https://en.wikipedia.org/wiki/SAM_(file_format)) covered in class is, together with the [BAM format](https://en.wikipedia.org/wiki/Binary_Alignment_Map) what practically all readmapping tools use these days to describe where patterns (approxmiately) occur in genomes.

We use a slightly simpler file format in GSA, which we will call Simple-SAM, that contains only the information relevant to the algorithms we implement. This format consists of zero or more tab-separated columns, each line having five columns. The columns contain:

 * Column 1: The name of the string we searched for (the pattern or read).
 * Column 2: The name of the reference string we searched in (the chromosome).
 * Column 3: The position where the pattern was found (indexing from 1).
 * Column 4: The CIGAR describing how to edit the reference into the read.
 * Column 5: The read string.

If you already know the read, since column one has its name, you don't need column 5, but we always include it since these are usually short and by having it at hand we can avoid looking up strings in large files of reads in downstream analysis.

The CIGAR is always the length of the pattern followed by `M` where there are no edits, and at this point in the class we do not worry about approximative matching. We will get to that later.

An example can look like this:

```
read1	chr1	2	3M	iss
read2	chr2	12	3M	mis
read4	chr2	17	6M	ssippi
```

The first line says that read `read1` is found at the second position in the chromosome called `chr1`. The read is `iss` and since it is 3 long and we only work with exact matches, the cigar is `3M`.

The second line says that read `read2`, which is the string `mis`, is found at position 12 in `chr2`.

The last line says that `read4`, the string `ssippi`, is found at index 17 in `chr2`.

This seems simple enough, and it is, but my experience is that more than half of the first projects are rejected because the output format is wrong, rather than there being anything wrong with the algorithms you implement, so I thought it would be a good idea to have an exercise where you make sure that you can output match results in this format.

The main problems are almost always one of these four:

 * The columns must be tab-separated, so space or spaces won't cut it. A space and a tab is not the same character, and some tools are picky about that.
 * The order of the columns are wrong (typically switching read and reference names).
 * The CIGAR is wrong (I don't know how that can happen, but it does happen a lot).
 * The position is zero-indexed rather than one-indexed (which is understandable if you are a computer scientist and using any none-R language developed within the last 50 years, but nevertheless wrong for SAM).

## Getting the output right

This exercise is all about getting the output format right, so we won't be implementing any search algorithm at all. You just need to read an input file with four tab-separated values per line

 * Column 1: The name of the chromosome
 * Column 2: The name of the read
 * Column 3: The actual string that is the read
 * Column 4: The position of the match (zero-indexed)

For each line, you output the corresponding Simple-SAM line.

The data from above, in this format, would look like:

```
chr1    read1   iss 1
chr2    read2   mis 11
chr2    read4   ssippi  16
```

When you debug your program, if that is necessary, you will want to know `diff` and `tr`. If you don't, you learn how they work before you do the exercise. (Running `man diff` and `man tr` will do the job).

If you want to know the lines that differ between two files, and what the lines are, you run `diff`:

```sh
diff right.sam wrong-order.sam
1,3c1,3
< read1	chr1	2	3M	iss
< read2	chr2	12	3M	mis
< read4	chr2	17	6M	ssippi
---
> read1	chr1	3M  2	iss
> read2	chr2	3M  12	mis
> read4	chr2	6M  17	ssippi
```

This tells me that there are three lines in each file that does not appear in the other, and if I examine the files I can see that the information is there but that the third and fourth columns are swapped. The position should go before the CIGAR, so I have to fix that.

It is harder to figure out with invisible characters, since a sequence of space can look a lot like a tab, but this is where `tr` comes in handy. It will replace one character with another, given a sequence of "from" and "to" characters, so you can use it to give tabs one visual appearance and spaces another:

```sh
diff right.sam space.sam | tr " \t" ".‣"
1,3c1,3
<.read1‣chr1‣2‣3M‣iss
<.read2‣chr2‣12‣3M‣mis
<.read4‣chr2‣17‣6M‣ssippi
---
>.read1.chr1....2.......3M......iss
>.read2.chr2....12......3M......mis
>.read4.chr2....17......6M......ssippi
```

(the first space on each line, '.', is not an error in the SAM file but `diff` adding a space that `tr` changes; you could run `tr` first and avoid that, but I'm smart enough not to get bothered by it).

I have provided you with a template program, `tosam`, and you just need to output the data you get in Simple-SAM format. After that, expect you to get it right in the coming projects. If not, I will weep for humanity, and you don't want that, do you?

