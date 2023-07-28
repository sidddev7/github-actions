### For building a react native app from the github actions and build the app from fastlane and upload to playstore for different tracks

- save the google api key to github secrets environments in base64 encoding
- Decode the google api key secret inside the workflow itself and store it in **tmp** directory that is created for workflow by github actions
- reference the path to that tmp json file so that it can be used for  authentication with google servers

## Github workflow file
```javascript
name: Engaged Ai Android Builder

on:
  push:
    branches: [patch/fastlane-android]

jobs:
  build:
    runs-on: ubuntu-22.04
    environment: Staging
    env:
      GOOGLE_API_KEY: ${{secrets.GOOGLE_PLAY_STORE_API_KEY}} // this is the base64 encoded api key stored in github secrets
    strategy:
      matrix:
        node-version: [18.12.1]
    steps:
      - uses: actions/checkout@v3
        with:
          ref: patch/fastlane-android
      - name: Use Nodejs ${{matrix.node-version}}
        uses: actions/setup-node@v2.5.2
        with:
          node-version: ${{matrix.node-version}}
      - run: npm i --force
      - name: Cache Gradle
        uses: actions/cache@v1
        with:   
          path: ~/.gradle/caches/
          key: cache-clean-gradle-${{ matrix.os }}-${{ matrix.jdk }}
      - name: Cache Gradle Wrapper
        uses: actions/cache@v1
        with:
          path: ~/.gradle/wrapper/
          key: cache-clean-wrapper-${{ matrix.os }}-${{ matrix.jdk }}
      - run: |
          echo  "$GOOGLE_API_KEY" | base64 --decode > /tmp/google-credentials.json
      - uses: ruby/setup-ruby@v1.150.0
        with:
          ruby-version: "3.0"
          bundler-cache: true
      - uses: maierj/fastlane-action@v3.0.0
        with:
          lane: "android build"

```
