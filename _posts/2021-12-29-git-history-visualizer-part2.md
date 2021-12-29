---
title: Building git-history-visualizer - part 2
---

# Drawing the graph

The next step was to find ways to draw a graph of the files and directories. I decided to use [D3.js](https://d3js.org/)
. I explored the examples and made a quick proof of concept of a file hierarchy using the tree layout:

![D3 tree layout](/assets/git_history_visualizer/tree.png)

One more example that is even closer to what I had in mind is
the [Force-Directed Tree](https://observablehq.com/@d3/force-directed-tree).

![D3 force-directed tree](/assets/git_history_visualizer/force_directed_tree.png)

Based on the example I started to implement the layout for a hard-coded file structure. Then I added some buttons to
simulate adding and removing the nodes in the graph.

![First tree](/assets/git_history_visualizer/d3_early.gif)

# Feeding the git history into the graph

The graph expects a hierarchical data structure that represents the directory for rendering the UI:

```json
{
  "name": "root",
  "children": [
    {
      "name": "a",
      "children": [
        {
          "name": "b.js"
        },
        {
          "name": "c.js"
        }
      ]
    }
  ]
}
```

The Git log would return the files as flat list, for example `['a/b.js', 'a/c.js']`. To convert the list into the tree
structure I created the `tree-builder` component. It uses the reduce function to generate the expected output (credit
goes to StackOverflow / MDN):

```typescript
export const parseFiles = (paths: string[]) => {
    const root: Tree = {name: "root", children: []};
    root.children = paths.reduce((r: Tree[], path: string) => {
        path.split('/')
            .reduce((acc: Tree[] | undefined, currentName) => {
                let temp = acc?.find(t => t.name === currentName);
                if (!temp) {
                    acc?.push(temp = {name: currentName, children: []});
                }
                return temp?.children;
            }, r);
        return r;
    }, [])
    return root
}
```

With that it was time to put the different pieces together. The main function would get the changes from git, render the
initial graph and then step through the changes in the history to update the graph.

```javascript
import {ForceDirectedGraph} from "./d3/force-directed-graph";
import {getChanges} from "./git/git-client";
import * as LightningFS from "@isomorphic-git/lightning-fs";
import http from "isomorphic-git/http/web";
import {parseCommit} from "./d3/tree-builder";

(async function () {
    const fs = new LightningFS('fs')
    const commits = await getChanges(fs, http);
    const initialTree = parseCommit(commits[0]);
    const graph = new ForceDirectedGraph(initialTree)
    for (let i = 1; i < commits.length; i++) {
        const commit = commits[i];
        for (const file of commit.files) {
            await new Promise(r => setTimeout(r, 500));
            if (file.type === 'add') {
                graph.add(file.path)
            } else if (file.type === 'delete') {
                graph.remove(file.path)
            }
        }
    }
})();
```

This is what the app looked like at this stage:

![Stepping through changes](/assets/git_history_visualizer/d3_main.gif)

# Improvements

With the basic functionality in place it was time to make some structural improvements. First I
converted `force-directred-graph.ts` to a class and improve the API it exposes.

Then I noticed that there are certain events that failed to update the graph as expected. The reason was that the 'add'
event of empty directories were filtered out in the git component. This was easy to fix, thanks to the existing unit
tests.

Another shortcoming was that in the nested file structure I no longer had the fill file path. This was a problem because
I needed a unique ID to find the nodes that I wanted to add/update/delete. This was also a rather small change to add
int he `parseFiles` function.

Next I added the proper handling of adding new nodes to the graph and added test cases that verify the graph logic.
The 'modify' was also still missing, so I decided to do a simple color fade of the node when there is an update event.
Turned out special characters as CSS Ids are tricky but there is `CSS.escape` to solve that.

```typescript
   function modify(path: string) {
    const cssId = CSS.escape(`root/${path}`);
    this.nodeContainer.select(`#${cssId}`)
        .transition()
        .duration(1000)
        .attr("fill", "red")
        .attr("stroke", "red")
        .transition() // transition back to normal
        .delay(1000)
        .duration(1000)
        .attr("fill", (d: any) => d.data.name.includes('.') ? "#0099ff" : "#000")
        .attr("stroke", (d: any) => d.data.name.includes('.') ? "#0099ff" : "#000");
}

```

To streamline the coding style I added [ESLint](https://eslint.org/) and configured the settings according
to [TypeScript ESLint](https://typescript-eslint.io/) and ran `eslint --fix` for the initial cleanup. After that only
few linting issues were left that required manual intervention.

Directories as black, files in blue. Commits to existing files appear flashing red.
![Part 2](/assets/git_history_visualizer/d3_steps.gif)

# Up next

At this point the basic functionality of my initial plan was mostly done. Check
out [part 3](2021-12-29-git-history-visualizer-part3.md) to find out about additional ideas and features that started to
appear.
