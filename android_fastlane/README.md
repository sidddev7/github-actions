### For building a react native app from the github actions and build the app from fastlane and upload to playstore for different tracks

- save the google api key to github secrets environments in base64 encoding
- Decode the google api key secret inside the workflow itself and store it in **tmp** directory that is created for workflow by github actions
- reference the path to that tmp json file so that it can be used for  authentication with google servers
- Also reference the aab file or apk file generated from the github actions to the Fastfile Lane explicitly as shown in Fastfile code

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

## Fastlane related Code

- For fastlane we need to create a aab or apk file before requesting upload_to_play_store
- 
### Appfile
```ruby
json_key_file("/tmp/google-credentials.json") # change it to the file name you created at time of decoding the base64 github secret
```

### Fastfile
```ruby
fastlane_version '2.213.0'

platform :android do
  #android lanes
  desc 'Build the Android application.'
  lane :build do
      gradle(task:"clean bundle",project_dir:"android/",build_type: 'Release',)
      upload_to_play_store(
        track:'beta',
        release_status: 'draft',
        aab:'./android/app/build/outputs/bundle/release/app-release.aab',  #this can be changed to apk if needed, look at the directory where your app is built by gradle
        package_name:"io.engagedai.mobileapp"
      )
  end
  lane :build_apk do
    gradle(task: 'assemble', build_type: 'Release', project_dir: 'android/')
  end

end
```

