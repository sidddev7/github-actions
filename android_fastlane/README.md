# For building a react native app from the github actions and build the app from fastlane and upload to playstore for different tracks

- Get your app ready to be deployed on Google play store

## Setup Fastlane in macos / Windows / Linux by following steps mentioned [here](https://docs.fastlane.tools/getting-started/ios/setup/)

- run **[Fastlane init](https://docs.fastlane.tools/getting-started/ios/setup/#:~:text=directory%20and%20run-,fastlane%20init,-Note%20that%20if)** inside your root directory
- Create your custom Lanes inside the Fastfile as you require for different operating systems like android and ios, check [this](https://github.com/sidddev7/fastlane#readme)
- Create Github actions workflow files as you require specifying the trigger branches, required runner image, jobs that you need to do, check [this](https://github.com/sidddev7/github-actions#readme)

## Setup needed to be done on Github repository
- Create environments you need, see [here](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- save the **google api key** to github secrets environments in base64 encoding

## Required Steps for github actions to have

- Decode the google api key secret inside the workflow itself, and store it in **tmp** directory that is created for workflow by github actions.
- reference the path to that tmp json file so that it can be used for authentication with google servers.
- Also reference the aab file or apk file generated from the github actions to the Fastfile Lane explicitly as shown in Fastfile code.

## Example Github workflow file
```javascript
name: Name your runner

on:
  push:
    branches: [**tirgger_branch_name**]

jobs:
  build:
    runs-on: ubuntu-22.04
    environment: **environment you want to use**
    env:
      GOOGLE_API_KEY: ${{secrets.GOOGLE_PLAY_STORE_API_KEY}} // this is the base64 encoded api key stored in github secrets
    strategy:
      matrix:
        node-version: [18.12.1]
    steps:
      - uses: actions/checkout@v3
        with:
          ref: **checkout_branch_name**
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

## Example Fastlane

- For fastlane we need to create a aab or apk file before requesting upload_to_play_store


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
        package_name:"com.example.mobile"
      )
  end
  lane :build_apk do
    gradle(task: 'assemble', build_type: 'Release', project_dir: 'android/')
  end

end
```

