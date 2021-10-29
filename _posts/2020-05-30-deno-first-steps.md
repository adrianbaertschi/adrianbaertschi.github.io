---
title: First steps with Deno 1.0
---
The [announcement of Deno 1.0](https://deno.land/v1) has received a lot of attention in the Webdev community. Time to
try out the JS/TS runtime from the creator of Node.js.

Like Node.js, Deno makes it possible to run JavaScript and TypeScript outside of a web browser. It tries to avoid
the [design mistakes in Node](https://www.youtube.com/watch?v=M3BM9TB-8yA) and ships in a single executable binary.

# Installation (Window 10)

Following the docs from <https://deno.land>, I used this Powershell command to download and install:

`iwr https://deno.land/x/install/install.ps1 -useb | iex`

This will download the latest deno.exe into C:\Users\\\<user>\\.deno\bin and setup the PATH so its ready to use in the
command line.

Visual Studio Code and IntelliJ IDEA both provide plugins to bring support into the IDE. The IDEA Plugin even has
support for debugging.

# Hello World

Simply create a text file and start writing some JavaScript or TypeScript. For example: `console.log("Hello Deno");` and
run it with `deno helloWorld.ts` which will print the text to the console.

# Make a HTTP Request

The following program reads the first argument which is passed in, makes a request to the url, reads the response and
prints it to the console.

```javascript
const url = Deno.args[0];
const res = await fetch(url);
const body = new Uint8Array(await res.arrayBuffer());
await Deno.stdout.write(body);
```

Run it with `deno run --allow-net=www.google.com first_steps.ts https://www.google.com`
**Important**: You have to give Deno explicit permissions to access the network, the same applies
to [other resources](https://deno.land/manual/getting_started/permissions).

# Reading a file

Example to read a number of files and print the contents to the console:

```javascript
const filenames = Deno.args;
for (const filename of filenames) {
  const file = await Deno.open(filename);
  await Deno.copy(file, Deno.stdout);
  file.close();
}
```

Run it with `deno run --allow-read files.ts curl.ts`

See more examples in the [Deno Manual](https://deno.land/manual/examples)

# Standard modules

See <https://deno.land/std> for a list of standard modules and how to use them. This example imports the uuid library.

```javascript
import { v4 } from "https://deno.land/std/uuid/mod.ts";
const myUUID = v4.generate();
const isValid = v4.validate(myUUID);
console.log(myUUID, isValid);
```

The first time you run this script with `deno run` wit will download the required dependencies. Dependencies and other
files are cached in C:\Users\<user>\\AppData\Local\deno (run `deno info` to display the paths)

# Using third party libraries

In addition, there is a growing directory of [third party modules](https://deno.land/x). that can be used by referencing
by url:

```javascript
import { camelCase } from "https://deno.land/x/case/mod.ts";
const camel = camelCase("hello world")
console.log(camel)
```

# Tooling

Deno ships with a number of tools for debugging, running tests, formatting etc.
See [Built-in Tooling](https://deno.land/manual/tools).

# Conclusion

Deno looks like a good alternative to Node if you want to run simple scripts or apps without having to deal with the
complexity and quirks of the Node platform. If you know JS/TS then getting started is quick and easy. As stated in the
manual, make sure you have some understanding about async/await. The developer experience is really nice, I especially
liked the simple setup of just running a single binary (I used VS Code and Git Bash to run it).
