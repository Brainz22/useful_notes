**Yanking (copying a range of lines):**

`81gg11yy`: cursor moves to line 81 and copies the next 11 lines.

**Putting (pasting):**

`P`: paste before cursor.

`p`: paste after cursor.

**Gives total number of lines:**

`:echo line('$')`: it shows the total number of lines on the command line inside vim.

**Deleting extra blank spaces at the end of lines:**

The following command deletes any trailing whitespace at the end of each line. If no trailing whitespace is found no change occurs, and the e flag means no error is displayed.

`:%s/\s\+$//e`
