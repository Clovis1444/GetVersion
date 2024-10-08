# Table of contents

-   [Table of contents](#table-of-contents)
-   [About](#about)
-   [Inputs](#inputs)
-   [Outputs](#outputs)
-   [Example usages](#example-usages)

# About

This github action allows you to get project version from almost any type of file.

## Features

-   Outputs project version from file.
-   Outputs whether a tag with the current version already exists.

## Credits

This action uses the following actions:

-   [actions/checkout](https://github.com/actions/checkout)
-   [mukunku/tag-exists-action](https://github.com/mukunku/tag-exists-action)

# Inputs

-   [file_to_read](#file_to_readrequired)
-   [line_pattern](#line_patternrequired)
-   [version_pattern](#version_pattern)
-   [custom_command](#custom_command)
-   [repo](#repo)
-   [checkout](#checkout)
-   [tag_prefix](#tag_prefix)
-   [tag_postfix](#tag_postfix)

## `file_to_read`[REQUIRED]

File that contains project version.

## `line_pattern`[REQUIRED]

RegEx pattern to a line, that contains project version.

## `version_pattern`

RegEx pattern to a line, that contains project version.

**Default value**: `[0-9]\+\.[0-9]\+\.[0-9]\+`

## `custom_command`

Execute your own command to determine project version. When using this the following inputs are ignored: `file_to_read`, `line_pattern`, `version_pattern`.

**Default value**: ` `

## `repo`

Repository name with owner. If you are using this you should mark [`checkout`](#checkout) option as **true**.

**Default value**: `${{ github.repository }}`

## `checkout`

Whether or not to checkout the file.

**Default value**: `false`

## `tag_prefix`

Tag prefix to search for existing tags.

**Default value**: `v`

## `tag_postfix`

Tag postfix to search for existing tags.

**Default value**: ` `

# Outputs

-   [project_version](#project_version)
-   [exists](#exists)

## `project_version`

Project version.

## `exists`

Returns true if tag with the current version already exists.

# Example usages

All examples presented use `Clovis1444/GetVersion@main` action. `@main` stands for _last commit_. It is highly recommended to use the **latest release version** of the action.

For example, if the **last release version** is **_v0.0.12_** use `Clovis1444/GetVersion@v0.0.12`.

-   [Minimalistic use case](#minimalistic-use-case)
-   [Check if tag exists](#check-if-tag-exists)
-   [Custom version pattern](#custom-version-pattern)
-   [Custom command](#custom-command)
-   [Checkout from other repo](#checkout-from-other-repo)
-   [Search for tag with prefix and postfix](#search-for-tag-with-prefix-and-postfix)

## Minimalistic use case

### CMakeLists.txt:

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(init VERSION 0.1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -Weffc++ -Wno-sign-compare -Wconversion -Wsign-conversion -pedantic-errors")

include(CTest)
enable_testing()

add_executable(init src/main.cpp)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

### Usage

```yaml
- uses: Clovis1444/GetVersion@main
  id: get_version
  with:
      file_to_read: CMakeLists.txt
      line_pattern: project(init VERSION
```

### Output

**steps.get_version.outputs.project_version**: `0.1.0`

## Check if tag exists

### Usage

```yaml
- uses: Clovis1444/GetVersion@main
  id: get_version
  with:
      file_to_read: CMakeLists.txt
      line_pattern: project(init VERSION

- name: Indicate if tag exists
  if: ${{ steps.get_version.outputs.exists }} == true
  run: echo Tag v${{ steps.get_version.outputs.project_version }} already exists
```

## Custom version pattern

### package.json

```json
{
	"name": "my_project",
	"version": "228.0.1.12",
	"description": "My cool project",
	"main": "src/index.js",
	"scripts": {
		"test": "echo \"Error: no test specified\" && exit 1"
	},
	"keywords": [],
	"author": "Clovis1444",
	"license": "ISC",
	"dependencies": {
		"@actions/core": "^1.10.1",
		"@actions/github": "^6.0.0"
	}
}
```

### Usage

```yaml
- uses: Clovis1444/GetVersion@main
  id: get_version
  with:
      file_to_read: package.json
      line_pattern: '"version":'
      version_pattern: '[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+'
```

### Output

**steps.get_version.outputs.project_version**: `228.0.1.12`

## Custom command

### Cargo.toml

```toml
[package]
name = "winapi_app"
version = "2.2.3"
edition = "2021"

[build-dependencies]
winres = "0.1.12"

[dependencies]
mslnk = "0.1.8"

[dependencies.windows]
version = "0.54.0"
features = [
    "Win32_Graphics_Gdi",
    "Win32_System_LibraryLoader",
    "Win32_System_DataExchange",
    "Win32_UI_WindowsAndMessaging",
    "Win32_Foundation",
    "Win32_UI_Input_KeyboardAndMouse",
    "Win32_UI_Shell",
    "Win32_Media_Audio",
]

```

### Usage

```yaml
- uses: Clovis1444/GetVersion@main
  id: get_version
  with:
      file_to_read: ""
      line_pattern: ""
      custom_command: grep -A 2 '\[package\]' Cargo.toml | awk -F\" '/version/ {print $2}'
```

### Output

**steps.get_version.outputs.project_version**: `2.2.3`

## Checkout from other repo

### Usage

```yaml
- uses: Clovis1444/GetVersion@main
  id: get_version
  with:
      file_to_read: tests/samples/CMakeLists.txt
      line_pattern: project(init VERSION
      checkout: true
      repo: Clovis1444/GetVersion
```

## Search for tag with prefix and postfix

Assuming you search for a tag with the following pattern: `v<project_version>-beta`.

### Usage

```yaml
- uses: Clovis1444/GetVersion@main
  id: get_version
  with:
      file_to_read: CMakeLists.txt
      line_pattern: project(init VERSION
      tag_prefix: v
      tag_postfix: -beta
```
