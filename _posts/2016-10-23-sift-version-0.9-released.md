---
layout: post
title: Sift Version 0.9 Released
---

A new version of sift is now available for download: [https://sift-tool.org/download](https://sift-tool.org/download)

## New Options

sift 0.9 supports several new options:


<dl>
<dt>--conf</dt>
<dd markdown="1">
The `--conf` option can be used to specify an additional config file. Settings configured in that file override settings from global/local config files.

This is especially useful in combination with the `--no-conf` option to use a specific configuration in scripts.
</dd>

<dt>--field-sep</dt>
<dd markdown="1">
The `--field-sep` option allows to use a custom field separator, this can be useful when the output of sift is processed by another script.

<pre>
<b>$ sift PM_SUSPEND linux-kernel -n --field-sep '|'</b>
linux-kernel/kernel/cpu.c|644|	case PM_SUSPEND_PREPARE:
...
</pre>
</dd>

<dt>--byte-offset</dt>
<dd markdown="1">
This option allows to show the byte offset of the matching line. If this options is combined with `--only-matching`, it shows the offset of the match.
</dd>

<dt>Support for custom types</dt>
<dd markdown="1">
sift supports types to limit searches to specific file types for quite some time now:
<pre>
# only search in perl files (*.pl, *.pm, *.pod, *.t
# or a perl shebang on the first line):
<b>$ sift -t perl pattern</b>
# Exclude html and xml files: 
<b>$ sift -T html,xml pattern</b>
</pre>
As of version 0.9, custom types can be created in addition to the built in types. 

The following example creates a type `script` that searches in *.pl, *.py and *.rb files:

<pre>
<b>sift --add-type 'script=*.pl,*.py,*.rb' --write-config</b>
</pre>

Without the option `--write-config` the created type would only be valid for the current call to sift.

The option `--list-types` can be used to list all builtin and custom types. It also shows some sample commands to create custom types.
</dd>
</dl>

---

## Performance improvements and better cross platform support

Sift is written in Go, and many things can be implemented with a good performance in Go. There are some limitations though - e.g. iterating over an array/slice includes bounds check to detect out of range errors. This can have a massive performance impact on tight loops.

Regarding sift, this affected the counting of newlines (only used when sift shows line numbers for matches) and a lowercase conversion routine. Up to sift 0.8, these routines were implemented in pure Go and alternatively in C. The C version performed better, but it had some overhead per function call (using cgo needs to translate between Go and C) and external dependencies (a C compiler).

As of version 0.9, the cgo dependency is removed. In addition to the pure Go implementations, sift now includes optimized assembly routines using SIMD instructions for 64-bit x86 processors.
The assembly implementation gives a 20x speedup compared to the pure Go implementation on CPUs I tested. In practice this means that for processing very large files with line numbers turned on, runtime is cut in half.

