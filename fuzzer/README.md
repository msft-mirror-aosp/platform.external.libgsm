# Fuzzer for libgsm decoder

## Plugin Design Considerations
The fuzzer plugin for GSM is designed based on the understanding of the
codec and tries to achieve the following:

##### Maximize code coverage
The configuration parameters are not hardcoded, but instead selected based on
incoming data. This ensures more code paths are reached by the fuzzer.

GSM supports the following options:
1. Verbosity (parameter name: `GSM_VERBOSE`)
2. Fastness (parameter name: `GSM_OPT_FAST`)

| Parameter| Valid Values| Configured Value|
|------------- |-------------| ----- |
| `GSM_VERBOSE` | 0. `Verbosity disabled` 1. `Verbosity enabled` | Bit 0 (LSB) of 1st byte of data. |
| `GSM_OPT_FAST`   | 0. `Regular algorithm will be used` 1. `Faster version of the algorithm will be used`  | Bit 1 (LSB) of 1st byte of data. |

This also ensures that the plugin is always deterministic for any given input.

##### Maximize utilization of input data
The plugin feeds the entire input data to the codec using a loop.
 * If the size of media file is less than 65 bytes, (65 - size) bytes are padded and then sent to
 the decoder so that decode happens for all the input files, regardless of their size.
 GSM decoder consumes alternating frames of 33 and 32 bytes, so 65 bytes of input are needed.
 * If the decode operation was successful, the input is advanced by the frame size
 * If the decode operation was un-successful, the input is still advanced by frame size so
that the fuzzer can proceed to feed the next frame.

This ensures that the plugin tolerates any kind of input (empty, huge,
malformed, etc) and doesnt `exit()` on any input and thereby increasing the
chance of identifying vulnerabilities.

## Build

This describes steps to build gsm_dec_fuzzer binary.

### Android

#### Steps to build
Build the fuzzer
```
  $ mm -j$(nproc) gsm_dec_fuzzer
```

#### Steps to run
Create a directory CORPUS_DIR and copy some gsm files to that folder
Push this directory to device.

To run on device
```
  $ adb sync data
  $ adb shell /data/fuzz/arm64/gsm_dec_fuzzer/gsm_dec_fuzzer CORPUS_DIR
```
To run on host
```
  $ $ANDROID_HOST_OUT/fuzz/x86_64/gsm_dec_fuzzer/gsm_dec_fuzzer CORPUS_DIR
```

## References:
 * http://llvm.org/docs/LibFuzzer.html
 * https://github.com/google/oss-fuzz
