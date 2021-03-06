# html-stitcher

- [html-stitcher](#html-stitcher)
  - [Introduction](#introduction)
  - [How Does It Work?](#how-does-it-work)
  - [Command Line Usage](#command-line-usage)
    - [Single File Input](#single-file-input)
    - [Directory Input](#directory-input)
  - [Features](#features)
    - [Parameter Substitution](#parameter-substitution)
    - [Inner Substitution](#inner-substitution)

## Introduction

`html-stitcher` is a CLI build tool for compiling HTML files from re-usable components. It is designed for decomposing large HTML files into smaller chunks to make them easier to work with, improving readability and providing the opportunity to extract repeated HTML blocks into re-usable components.

Here's an example to illustrate the basic principle:

![Hello world](doc/media/hello-world.png?raw=true "Hello world")

We start with an input file known as a *root* file, named `index.html`. The input file includes the `<welcome></welcome>` element which tells `html-stitcher` to substitute the element with the output of compiling the `welcome` *partial* defined in `welcome.html`. The final output generated by `html-stitcher` is simply the result of merging both files together.

## How Does It Work?

`html-stitcher` works with HTML files, which are categorized into *root* files and *partial* files:

* Root files act as entrypoints for `html-stitcher` to process and produce output files for. Root files are always processed and always result in a corresponding output file.
* Partial files are treated the same as root files, but the output is instead included in the output file for the root file that included it. Partial files can also reference other partial files. Partial files are only processed when included by another file.
* Each output file can therefore be thought of as being composed from a tree of HTML files, with one root file and any number of descendant partial files.

Any file can include a partial in its generated output by including a `<name></name>` element where `name` is the file name, preceding the first dot, of the partial to include.

## Command Line Usage

`html-stitcher` requires an input path, which can either be the path to a single *root* file or a directory. Depending on whether a file or directory is specified, there are some slight differences in how `html-stitcher`'s option flags are interpreted and used.

### Single File Input

Minimum usage:

    html-stitcher path/to/root/file.html

Full usage:

    html-stitcher path/to/root/file.html -o path/to/output/file.html --partial-files **/*.html

By default, `html-stitcher` will write the output of the compiled file directly to the shell. The output from the run can then be piped directly into another program. The output can instead be directly written to a file using the optional `-o/--output` option.

`html-stitcher` will attempt to identify *partial* files by searching the same directory as the input file using the glob pattern specified by the `--partial-files` option. By default this option is set to `**/*.html` but it can be changed as necessary.

### Directory Input

Minimum usage:

    html-stitcher path/to/src/directory -o path/to/dist/directory

Full usage:

    html-stitcher path/to/src/directory -o path/to/dist/directory --root-files **/*[!partial].html --partial-files **/*.html

The `--output` (`-o`) option is always required for an input directory and must be an existing *directory*. `html-stitcher` will write all compiled files to this path, maintaining the same directory path as the *root* file had relative to the input directory. For example, the output from `/{input}/dir1/dir2/file.html` will be written to `/{output}/dir1/dir2/file.html`.

It is also important that *root* files and *partial* files have some kind of distinction in their names or paths so that `html-stitcher` can distinguish between them. `html-stitcher` uses the glob patterns specified by the `--root-files` and `--partial-files` options to manage this. By default, the *root* file glob is set to `**/*[!.partial].html` and the *partial* file glob is set to `**/*.html`. These defaults allow a convention of *root* file names ending in `.html` and partial file names ending in `.partial.html` to be used, but these patterns can be changed as desired.

## Features

### Parameter Substitution

Partial elements can be can be parameterised by adding additional attributes to the element's start tag:

```html
<!-- index.html -->
<h2>Largest Cities by Population</h2>
<ol>
    <!-- Each city has a `name`, `population` and `color` parameter -->
    <city name="Tokyo" population="37,468,000" color="red"></city>
    <city name="Delhi" population="28,514,000" color="blue"></city>
    <city name="Shanghai" population="25,582,000" color="green"></city>
</ol>
```

Parameters in the partial file are referenced by name using the `${var}` format:

```html
<!-- city.partial.html -->
<li>
    <b style="color: ${color}">${name}</b>
    <p>Population: ${population}</p>
</li>
```

During compilation, each `${parameter}` found in the partial's HTML is substituted with the value from the including element.
In this example, the first list item has `${name}` replaced with `Tokyo` and `${color}` replaced with `red`:

```html
<!-- output.html -->
<h2>Largest Cities by Population</h2>
<ol>
    <li>
        <b style="color: red">Tokyo</b>
        <p>Population: 37,468,000</p>
    </li>
    <li>
        <b style="color: blue">Delhi</b>
        <p>Population: 28,514,000</p>
    </li>
    <li>
        <b style="color: green">Shanghai</b>
        <p>Population: 25,582,000</p>
    </li>
</ol>
```

### Inner Substitution

Inner substitution is an extension of parameter substitution, only the parameter name is `inner` and the parameter value is the inner HTML content of the partial element.

```html
<!-- index.html -->
<div>
    <card>The quick brown fox jumps over the lazy dog</card>
    <card>The five boxing wizards jump quickly</card>
    <card>How vexingly quick daft zebras jump</card>
</div>
```

The inner HTML is referenced with `${inner}`:

```html
<!-- card.partial.html -->
<div>
    <p>${inner}</p>
</div>
```

As with parameter substitution, any reference to `${inner}` is replaced with the content of the including element:

```html
<div>
    <div>
        <p>The quick brown fox jumps over the lazy dog</p>
    </div>
    <div>
        <p>The five boxing wizards jump quickly</p>
    </div>
    <div>
        <p>How vexingly quick daft zebras jump</p>
    </div>
</div>
```
