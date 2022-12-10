# Troubleshooting android instrumented test on Gitlab Ci

## Gitlab runner
While it is possible to execute unit tests on a shared runner on Gitlab's CI, instrumented tests require hardware acceleration which is not available on shared runners presently. Thus, the only choice is to configure your own machine as a runner for Gitlab to use in running CI/CD.

Ensure docker is installed [here](https://docs.docker.com/engine/install/ubuntu/)

Set up your Gitlab runner [here](https://docs.gitlab.com/runner/install/linux-manually.html)

## System images for sdkmanager
Depending on the build spec on your local android studio, you can use the specifc system image for the emulator on gitlab CI from this [list](https://gist.github.com/alvr/8db356880447d2c4bbe948ea92d22c23)

```
# example
sdkmanager "platform-tools" "emulator" "system-images;android-33;google_apis_playstore;x86_64"
```

## Running the emulator
Ensure the emulator is running without graphic interface, audio, boot animation, snapshot & it should wipe all data it may have stored during its boot sequence in the background.

```
# example
emulator -avd test -no-window -no-audio -no-boot-anim -no-snapshot -wipe-data &
```

## Start the server
Some times the server doesnt start after the has been created and run, in this case you can simply start the server with the command

```
adb start-server
```
## Review the "wait-for-emulator-script"
The default [script](https://raw.githubusercontent.com/travis-ci/travis-cookbooks/0f497eb71291b52a703143c5cd63a217c8766dc9/community-cookbooks/android-sdk/files/default/android-wait-for-emulator) is sufficent without change, but sometimes the test might timeout before it gets a chance to run. Thus you may want to modify the timeout in seconds
