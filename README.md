# Working From a Theme in Hugo

This repository illustrates how you might choose a Hugo theme and modify its example site to build a site of your own.  We use the theme [`personal web`](.https://github.com/bjacquemet/personal-web).

## Installation

Clone the repository, with the theme as a submodule:

```
git clone --recursive https://github.com/bjacquemet/personal-web.git
```

We need to begin from a specific commit of the theme, so also run:

```
cd themes/personal-web
git checkout bfdce3a65
cd ..
cd ..
```

## Usage

Study the development of the theme by checking out specific branches in numerical order, e.g.:

```
git checkout 01-example-site
```