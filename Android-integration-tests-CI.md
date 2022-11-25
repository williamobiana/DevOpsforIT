# Running Android integration tests in our CI

For android apps containing a lot of features and uses an emulator as part of its build concept, running CI unittest would require a diffrent approach. 

The goal we want our unittest job to accomplish are the following:
* Install SDKmanager
* Start and run an emmulator
* Gradle connectedAndriodTest

```
image: openjdk:11-jdk

variables:
  ANDROID_COMPILE_SDK: "30"
  ANDROID_BUILD_TOOLS: "30.0.3"
  SDK_TOOLS: "8512546" # modify with latest version from https://developer.android.com/studio/#command-tools
  EMULATOR_VERSION: "24"
  
before_script:
  - apt-get --quiet update --yes
  - wget --quiet --output-document=android-sdk.zip https://dl.google.com/android/repository/commandlinetools-linux-${SDK_TOOLS}_latest.zip
  - unzip -q android-sdk.zip -d android-sdk-linux
  - rm android-sdk.zip

  # The hierarchy of the files have been changed, so you will have to re-arrange it

  # ANDROID_HOME is deprecated, we will use ANDROID_SDK_ROOT
  - export ANDROID_SDK_ROOT=$PWD/android-sdk-linux

  # the current hierarchy is android-sdk-linux/cmdline-tools/tools (but the depenencies are not in the tools folder)
  # move the necessary dependencies into the tools folder
  - mv $ANDROID_SDK_ROOT/cmdline-tools/{lib,bin,source.properties,NOTICE.txt} $ANDROID_SDK_ROOT/cmdline-tools/tools
  
  # create a latest folder that will have the updated version of the dependencies 
  - mkdir $ANDROID_SDK_ROOT/cmdline-tools/latest  

  # define PATH for all executable files we will run (skdmanger, avdmanager, abd, emulator)
  - export PATH=$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/cmdline-tools/tools/bin:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_SDK_ROOT/emulator

  # sdkmanager is located in the tools/bin folder which we have specified in our PATH $ANDROID_SDK_ROOT/cmdline-tools/tools/bin
  - sdkmanager --sdk_root=${ANDROID_SDK_ROOT} --update > update.log
  - echo y | sdkmanager --sdk_root=${NDROID_SDK_ROOT} "platforms;android-${ANDROID_COMPILE_SDK}" "build-tools;${ANDROID_BUILD_TOOLS}" "extras;google;m2repository" "extras;android;m2repository" > installPlatform.log

  - chmod +x ./gradlew 

stages:
  - test
  
unitTests:
  stage: test
  script:
    - ./gradlew test

instrumentedTests:
  stage: test
  script:
    - apt-get --quiet update --yes
    - apt-get --quiet install --yes libx11-dev libpulse0 libgl1 libnss3 libxcomposite-dev libxcursor1 libasound2
    - wget --quiet --output-document=android-wait-for-emulator https://raw.githubusercontent.com/travis-ci/travis-cookbooks/0f497eb71291b52a703143c5cd63a217c8766dc9/community-cookbooks/android-sdk/files/default/android-wait-for-emulator
    - chmod +x android-wait-for-emulator
    - sdkmanager --update > update.log
    - sdkmanager "platform-tools" "emulator" "system-images;android-${EMULATOR_VERSION};default;armeabi-v7a"  > installEmulator.log
    
    # create the emulator
    - echo no | avdmanager create avd -n test -k "system-images;android-${EMULATOR_VERSION};default;armeabi-v7a"
    
    # for some reasons the emulator doesnt start automatically, so we have to start it
    - abd start-server
    
    # run without in the background without window and audio
    - emulator -avd test -no-window -no-audio &
    - ./android-wait-for-emulator
    - adb shell input keyevent 82
    - ./gradlew connectedCheck      #or ./gradlew connectedAndroidTest
```

We installed the sdkmanager in the `before script` section

We installed the emulator and ran the gradlew command in the `script` section of our `unitTest job` and `intrumentedTest job` 