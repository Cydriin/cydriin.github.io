## `sed` - Stream Editor
```bash
# Substitute all occurrences of pattern with replacement in file.txt
sed 's/pattern/replacement/g' file.txt

# Print only the lines matching pattern in file.txt
sed -n '/pattern/p' file.txt

# Add prefix- to the start of each line in file.txt
sed 's/^/prefix-/' file.txt

# Delete lines containing pattern in file.txt
sed '/pattern/d' file.txt

# Replace only the first occurrence of pattern in each line
sed 's/pattern/replacement/' file.txt
```

## `awk` - Pattern Scanning and Processing Language
```bash
# Print lines matching pattern in file.txt
awk '/pattern/ {print $0}' file.txt

# Print the first and third columns of file.txt
awk '{print $1, $3}' file.txt

# Sum the values in the first column of file.txt
awk '{sum += $1} END {print sum}' file.txt

# Print lines where the value in the second column is greater than 10
awk '$2 > 10' file.txt
```

## `xargs` - Build and Execute Command Lines from Standard Input
```bash
# Run echo command for each line in file.txt
cat file.txt | xargs echo

# Remove files listed in file.txt
cat file.txt | xargs rm

# Run mkdir for each directory listed in file.txt
cat file.txt | xargs -I {} mkdir {}
```

## `grep` - Global Regular Expression Print
```bash
# Find lines containing pattern in file.txt
grep 'pattern' file.txt

# Find lines containing pattern recursively in a directory
grep -r 'pattern' /path/to/directory

# Find lines that do not contain pattern
grep -v 'pattern' file.txt

# Find lines with whole word pattern
grep -w 'pattern' file.txt
```

## `cut` - Remove Sections from Each Line of Files
```bash
# Extract the first field (assuming fields are separated by :)
cut -d ':' -f 1 file.txt

# Extract the first and third fields
cut -d ':' -f 1,3 file.txt

# Extract characters from position 1 to 5
cut -c 1-5 file.txt
```

## `sort` - Sort Lines of Text Files
```bash
# Sort lines in file.txt in ascending order
sort file.txt

# Sort lines in file.txt in descending order
sort -r file.txt

# Sort lines numerically
sort -n file.txt
```

## `uniq` - Report or Omit Repeated Lines
```bash
# Remove duplicate lines in file.txt (file should be sorted first)
uniq file.txt

# Count occurrences of each line
uniq -c file.txt

# Only print duplicate lines
uniq -d file.txt
```

## Using nvim in Conjunction
```bash
:%s/.$//  # removes the dot from the end of each line
:%! sort -u  # removes duplicates
:%!xargs -n1 -I{} sh -c 'echo {} | base64 -d'  # will take each line (that's base64 encoded) and decode it back into vim
```