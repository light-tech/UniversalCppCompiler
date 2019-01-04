Project File Specification
==========================

The project description file `project.json` is a simple human-readable [JSON]() file. An example can be found in our [Getting Started](https://github.com/light-tech/DevMaxGettingStarted) repository.

Project
-------

The whole file is a JSON object of "type" `PROJECT` consisting of the following members
```JSON
PROJECT {
"version": INT,
"name": STRING,
"build_commands": BUILD_COMMAND_MAP,
"build_definitions": ARRAY of BUILD_DEFINITION
}
```

Build commands
--------------

`BUILD_COMMAND_MAP` is a JSON object treated as an association (i.e. a dictionary) from `STRING` to `BUILD_COMMAND` object which are of the following structure:
```JSON
BUILD_COMMAND {
"action": ONE OF "interpret" OR "compile" OR "link",
"args": ARRAY OF STRINGS,
"sys_include_dir": ARRAY OF STRINGS,
"include_dir": ARRAY OF STRINGS,
"sys_lib_dir": ARRAY OF STRINGS
}
```
The interpretation of the other members are determined by the `"action"` key. In any case, the fields `sys_include_dir`, `include_dir` and `sys_lib_dir` are RELATIVE paths to be expanded to absolute pathh by DevMax: Each relative path in `sys_include_dir` are to be prepended with the path to `C++Include`and similarly, `sys_lib_dir` with the path to `C++Lib` folder. (These folders are in the same place as `C++Projects` folder.) The paths in `include_dir` are, on the other hand, treated relative to the project folder. Needless to say, `sys_lib_dir` is only considered when the action is `link` whereas the `*include_dir` are only considered for the other two actions.
The member `args` consists of the command line arguments (switches) you usually pass to clang compiler or linker.

Build definition
----------------

A `BUILD_DEFINITION` object consists of a name and an array of `BUILD_STEP`.
```JSON
BUILD_STEP {
"command" : Name of a BUILD_COMMAND defined in parent PROJECT object,
"inputs": ARRAY of STRING,
"output": STRING
}
```
When executing a `BUILD_DEFINITION`, DevMax further adds the `inputs` and `output` to the command after prepending these relative paths with the path to the project.

Example
-------

Let us take a simple example. Assuming `$ProjectDir` refers to where this project is located, for example `<...>/C++Projects/ExampleProject`. Then `$SysIncludeDir := $ProjectDir../../C++Include` and `$SysLibDir := $ProjectDir../../C++Lib`.

```JSON
{
"version":1,
"name":"Example project",
"build_commands": {
	"CompileC++" : {
		"action":"compile",
		"args":["-fms-extensions", "-fms-compatibility", "-x", "c++", "-std=c++14", "-w"],
		"sys_include_dir":["ucrt", "msvc"],
		"include_dir":[""],
		"comment":"The empty string in include_dir has the effect of adding the project folder to non-system header search path. For the record, any field not-interpreted by DevMax can be used to add comment like this."
	},
	"MakeExe" : {
		"action":"link",
		"args":["/defaultlib:msvcrt.lib", "/subsystem:Console"],
		"sys_lib_dir":["msvc", "winsdk"]
	},
},
"build_definitions": [
	{
		"name":"Build program",
		"build_steps": [
			{
				"command":"CompileC++",
				"inputs":["src/Source.cpp"]
			},
			{
				"command":"MakeExe",
				"inputs":["src/Source.o"],
				"output":"Source.exe"
			}
		]
	}
]
}
```

Running the build definition `"Build program"` has the effect similar to that of running two fairly long commands:
```
clang -isystem $SysIncludeDir/ucrt -isystem $SysIncludeDir/msvc -I $ProjectDir -fms-extensions -fms-compatibility -x c++ -std=c++14 -w -c $ProjectDir/src/Source.cpp -o $ProjectDir/src/Source.o

link /libpath:$SysLibDir/msvc /libpath:$SysLibDir/winsdk /defaultlib:msvcrt.lib /subsystem:Console $ProjectDir/src/Source.o /out:$ProjectDir/Source.exe
```
It is safer than writing these commands if you need to quote arguments correctly.