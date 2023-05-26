## Github Actions

- Github actions is used for automating several task based on the trigger points defined in the workflows file

- github actions provide support for several tech stacks ranging from node, python, ruby on rails and more

- lets see how to configure the Node.js workflow file

name: Node.js Github actions Workflow setup

```javascript
on:
  push:
    branches: [`Mention the branches you want github to look for the push actions`]

jobs: //Define the jobs to be done when this event is triggered
    build:
        runs-on: ubuntu-latest | macos-latest | windows-latest | or your own VM

        strategy:
            matrix:
                node-version: [18.x] //or any other node version you want
        
        steps:
            - uses: actions/checkout@v3 // this helps in managing the git branches
              with:
                ref: // branchname you want to checkout to
            - name : Use Node.js ${{matrix.node-version}} // this is the name shown as the process in the batch
              uses: actions/setup-node@v2.5.2
              with:
                node-version: ${{matrix.node-version}}
            - run: npm i // install all the dependencies required
            /*
                other jobs to be done can be listed here
            */

```
