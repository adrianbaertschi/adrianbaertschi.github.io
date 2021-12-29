---
title: Building git-history-visualizer - part 1
---

# Intro

For the majority of my software engineering career I was focusing on (Java) backend development. I got some exposure to
the web frontend world through Angular and React, but not enough to be confident in this vast ecosystem. So I decided to
brush up my skills by starting a small pet project. In the past I played around with this really cool project
called [Gource](https://gource.io/), a tool that generates an animated tree of a source control repository. The idea was
born to re-create something similar for browser. Gource is [Open Source](https://github.com/acaudwell/Gource/) and
written mostly in C++.

Example repo visualisation by Gource:

![Gource example](https://thumbs.gfycat.com/EuphoricConfusedBrahmancow-size_restricted.gif)

In general, I imagined my app will have three parts:

- Interacting with a Git repository (clone it, read the history)
- Drawing a tree / graph of the directories and files
- Animating the graph with changes from the Git commits

# Walking through the Git history

As a first step I looked into JavaScript libraries for Git operations. Some googling lead me to the impressive
[isomorphic-git](https://isomorphic-git.org/en/) project, which has nice docs, is in active development supports all the
features that are needed.

The API is easy to use, and it is possible to run it in the browser or on Node.js.

```javascript
// Node.js example
const path = require('path')
const git = require('isomorphic-git')
const http = require('isomorphic-git/http/node')
const fs = require('fs')

const dir = path.join(process.cwd(), 'test-clone')
git.clone({fs, http, dir, url: 'https://github.com/adrianbaertschi/mars-rover'})
    .then(git.log({fs, dir}))
    .then(console.log)
```

The idea is to read the commits and convert them into a data structure that then later will be fed into the graph
visualisation.

For the got log entries I needed something similar to the output of `git log --name-status --reverse` which prints each
commit in this format:

```
commit 26d3c8f6ed2951dd81e55b6a8b78ba6933fc2ae8
Author: adrianbaertschi <adrian.baertschi@zuehlke.com>
Date:   Fri May 28 23:44:04 2021 +0800

    Minor cleanups, use dummy data for now

D       index.html
M       src/explore-d3.ts
M       webpack.config.js
```

(Gource is actually
[running git log](https://github.com/acaudwell/Gource/blob/06174a1ba57b9a7b7e87cd244b4be09b8f46fd3b/src/formats/git.cpp#L91)
directly and then parsing the output)

To have this information I needed to walk through the git history and compare each commit with the previous one. The
result should be which files are affected by the commit and how (add/modify/delete). Luckily there
is [this snippet](https://isomorphic-git.org/docs/en/snippets#git-diff-name-status-commithash1-commithash2)
which I used as the base for my implementation. It utilises the [walk command](https://isomorphic-git.org/docs/en/walk)
of isomorphic-git. This idea originally comes from a project that had the same idea as me :-):
[https://github.com/kpj/GitViz](https://github.com/kpj/GitViz)

# Making it testable

To make the future development easier, I started to add simple test cases using [Jest](https://jestjs.io/) for the
conversion logic. The tests clone a demo repo and verify that the commits and file information are evaluated in the
right format.

```javascript
import {getChanges} from "./explore-git";

const fs = require('fs')
const http = require('isomorphic-git/http/node')
const path = require('path')
const dir = path.join(process.cwd(), 'test-clone')

test('mars-rover repo first commit lists files', async () => {
    const changes = await getChanges(fs, http, dir);
    const firstCommit = changes[0];
    expect(firstCommit.commit).toBe('1c416b90e6dffb31ea657976be5961fb04d1c5fc')
    const expectedFiles = [
        {path: ".gitignore", type: "add"},
        {path: "README.md", type: "add"},
        {path: "build.gradle", type: "add"},
        {path: "gradle/wrapper/gradle-wrapper.jar", type: "add"},
        {path: "gradle/wrapper/gradle-wrapper.properties", type: "add"},
        {path: "gradlew", type: "add"},
        {path: "gradlew.bat", type: "add"},
        {path: "settings.gradle", type: "add"},
        {path: "src/main/java/Direction.java", type: "add"},
        {path: "src/main/java/Plateau.java", type: "add"},
        {path: "src/main/java/Rover.java", type: "add"},
        {path: "src/test/java/PlateauTest.java", type: "add"},
        {path: "src/test/java/RoverTest.java", type: "add"}
    ]
    expect(firstCommit.files).toStrictEqual(expectedFiles);
})
```

At this stage the git implementation was still written for Node.js only, so I needed to change it to be executable in
the browser by switching the filesystem from Nodes `fs`
to [lightning-fs](https://github.com/isomorphic-git/lightning-fs). In addition, the http library needs to be changed
from `isomorphic-git/http/node` to `isomorphic-git/http/web`. To keep the code testable I keep `http` and `fs` as
dependencies of my git module, then inject the node version for the tests and the browser versions for the real app.

```javascript
import {getChanges} from "./git/explore-git";
import * as LightningFS from "@isomorphic-git/lightning-fs";
import http from "isomorphic-git/http/web";

const fs = new LightningFS('fs')
getChanges(fs, http).then(value => console.log(value))
```

To further improve the development experience I introduced TypeScript and started to add more and more type information
to the codebase.

Type definitions of the git history result:

```typescript
export interface Commit {
    commit: string
    files: FileChange[]
}

export interface FileChange {
    path: string
    operation: FileOperation
    isDirectory: boolean
}

export type FileOperation = 'ADD' | 'REMOVE' | 'MODIFY'
```

One tricky parts of the git conversions turn out to be file and directory renaming, which are modelled as two operations
(first remove old, then add new file).

# Up next

With this I felt I have all the data needed to start building the visualisation check
out [part 2]({% post_url 2021-12-29-git-history-visualizer-part2 %}).
