## Welcome to the LEAP Hand C++ SDK

In order to use the LEAP Hand with C++ sdk please read the following information.

#### Install
- Build DynamixelSDK on $(YOUR_PLATFORM) {linux32, linux64, macos} (default: linux64) and leap_hand control

```bash
  cd ~/LEAP_Hand_API/cpp
  mkdir build && cd build
  cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=</where/to/install>
  cmake --build . --target run_make_dynamixel_sdk -j$(nproc)
  cmake --build . --target test_leap_hand -j$(nproc)
```

#### Usage 

Please execute the following commands to run the test.

```bash
cd ~/LEAP_Hand_API/cpp/build/
./test_leap_hand
```