- [Bash Basic](#bash-basic)
  - [Some tricks](#some-tricks)
    - [Check if a file exist](#check-if-a-file-exist)
    - [Check first character of a string](#check-first-character-of-a-string)
    - [Remove `space` from a string](#remove-space-from-a-string)
    - [find all find with specific extension of directory and its subdirectory](#find-all-find-with-specific-extension-of-directory-and-its-subdirectory)
    - [Parse file with shell script](#parse-file-with-shell-script)
    - [declare a array](#declare-a-array)
    - [check if a string/line exist in a file](#check-if-a-stringline-exist-in-a-file)
    - [replace whole line in of a file that match a pattern](#replace-whole-line-in-of-a-file-that-match-a-pattern)
    - [Use Bearer Token for authetication](#use-bearer-token-for-authetication)
      - [wget](#wget)
      - [curl](#curl)
    - [`pipe` Explained](#pipe-explained)

# Bash Basic

## Some tricks

### Check if a file exist

```bash
if [ ! -f /tmp/foo.txt ]; then
    echo "File not found!"
fi
```

> Bash Ref:
> https://tiswww.case.edu/php/chet/bash/bashref.html

### Check first character of a string

**Wildcard:**

```bash
str="/some/directory/file"
if [[ $str == /* ]]; then
  echo 1;
else
  echo 0;
fi
```

**Substring expansion:**

```bash
if [[ ${str:0:1} == "/" ]] ; then
  echo 1;
else
  echo 0;
fi
```

This is going to take a substring of **str** starting at the **0th** character with length **1**.

```ini
  ${parameter:offset}
  ${parameter:offset:length}
          Substring  Expansion.   Expands  to  up to length characters of
          parameter starting at the character specified  by  offset.   If
          length is omitted, expands to the substring of parameter start-
          ing at the character specified by offset.   length  and  offset
          are  arithmetic  expressions (see ARITHMETIC EVALUATION below).
          length must evaluate to a number greater than or equal to zero.
          If  offset  evaluates  to a number less than zero, the value is
          used as an offset from the end of the value of  parameter.   If
          parameter  is  @,  the  result  is length positional parameters
          beginning at offset.  If parameter is an array name indexed  by
          @ or *, the result is the length members of the array beginning
          with ${parameter[offset]}.  A negative offset is taken relative
          to  one  greater than the maximum index of the specified array.
          Note that a negative offset must be separated from the colon by
          at  least  one space to avoid being confused with the :- expan-
          sion.  Substring indexing is zero-based unless  the  positional
          parameters are used, in which case the indexing starts at 1.
```

**Regex:**

```bash
if [[ $str =~ ^/ ]];then
  echo 1;
else
  echo 0;
fi
```

`^` indicates starting with.

Ref: https://medium.com/factory-mind/regex-tutorial-a-simple-cheatsheet-by-examples-649dc1c3f285

### Remove `space` from a string

```bash
str=" This sentence contains leading trailing and intermediate whitespaces "

## Remove leading whitespaces
lstr="$( echo "${str}" | sed -e 's/^[[:space:]]*//')"
echo "$lstr"

## Remove trailing whitespaces
tstr="$( echo "${str}" | sed -e 's/[[:space:]]*$//')"
echo "$tstr"

## Remove all white spaces
astr="$( echo "${str}" | tr -d '[:space:]')"
echo "$astr"
```

Ref: https://unix.stackexchange.com/questions/249869/meaning-of-101/249870

### find all find with specific extension of directory and its subdirectory

```console
find . -type f -name "*.conf"
```

### Parse file with shell script

```sh
#!/bin/sh
set -e

cmd="/usr/local/bin/docker-entrypoint.sh"
configFile="/usr/config/memcached.conf"

# parseConfig parse the config file and convert it to argument to pass to memcached binary
parseConfig() {
    args=""
    while read -r line || [ -n "$line" ]; do
        case $line in
            -*) # for config format -c 500 or --conn-limit=500
                args="$(echo "${args}" "${line}")"
            ;;
            [a-z]*) # for config format conn-limit = 500
                trimmedLine="$(echo "${line}" | tr -d '[:space:]')" # trim all spaces from the line (i.e conn-limit=500)
                param="$(echo "--${trimmedLine}")"                  # append -- in front of trimmedLine (i.e --conn-limit=500)
                args="$(echo "${args}" "${param}")"
            ;;
            \#*) # line start with #
                # commented line, ignore it
            ;;
            *) # invalid format
                echo "\"$line\" is invalid configuration parameter format"
                echo "Use any of the following format\n-c 300\nor\n--conn-limit=300\nor\nconn-limit = 300"
                exit 1
            ;;
        esac
    done <"$configFile"
    cmd="$(echo "${cmd}" "${args}")"
}

# if configFile exist then parse it.
if [ -f "${configFile}" ]; then
    parseConfig
fi
# Now run docker-entrypoint.sh and send the parsed configs as arguments to it
$cmd
```

### declare a array

We can declare array in bash script using,

```bash
declare -a prefixArray
```

Now, we can put data on array at any index. Array is 0 indexed.

```bash
prefixArray[INDEX]=VALUE
prefixArray[0]="this is first element"
```

After using array clear the memory using `unset`,

```bash
unset prefixArray
```

### check if a string/line exist in a file

**Syntax:**

```bash
if grep -Fxq "$WhatToFind" my_list.txt
then
    # code if found
else
    # code if not found
fi
```

Here,

- F: Affects how PATTERN is interpreted (fixed string instead of a regex)
- x: Match whole line
- q: Shhhhh... minimal printing

**Example:**

```bash
if grep -q ^"$prefix:" out.yaml; then
    echo "$prefix found in out.yaml"
else
    echo "Not found"
fi
```

This check if `out.yaml` file has a line starting with `$prefix`.

### replace whole line in of a file that match a pattern

**Syntax:**

```bash
sed -i "/string or regex goes here/c\replace with this" fileName
```

Here, `-i` for replace in place and `c` is for change/replace.

**Example:**

```bash
sed -i "/^$key:/c\\$line" out.yaml
```

This  replace the line that start with `$key` of `out.yaml` file with `$line`.

### Use Bearer Token for authetication

#### wget

**Syntax:**

```bash
wget --header="Authorization: Bearer ${TOKEN}" ${URL}
```

**Example:**

```bash
wget --header="Authorization: Bearer ${TOKEN}" https://sandbox.xill.io/v2/contents/5a184d0207903113023b5aaa/SANDBOX.md --no-check-certificate
```

#### curl

**Syntax:**

```bash
curl -H 'Accept: application/json' -H "Authorization: Bearer ${TOKEN}" ${URL}
```

**Example:**

```bash
curl -H 'Accept: application/json' -H "Authorization: Bearer ${TOKEN}" https://{hostname}/api/myresource
```

### `pipe` Explained

Link: https://brandonwamboldt.ca/how-linux-pipes-work-under-the-hood-1518/