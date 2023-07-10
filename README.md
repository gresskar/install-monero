# Monero stuff

## 1) Install Visual Studio

Download Visual Studio Community from [here](https://visualstudio.microsoft.com/)

During setup, make sure these are checked:
- Workloads -> Desktop development with C++
- Individual components -> MSVC v??? - VS 20?? C++ x64/x86 build tool (Latest)
- Individual components -> C++ CMake tools for Windows
- Individual components -> C++/CLI support for v??? build tools (Latest)
- Individual components -> Git for Windows

## 2) Monero

*I'm using pre-built binaries; monero must be built with MSYS2 and I don't want to deal with that rn*

### 2.1) Download and unzip monero:

```
Invoke-WebRequest -Uri "https://downloads.getmonero.org/cli/win64" -OutFile "~\Downloads\monero.zip"
Expand-Archive -LiteralPath "~\Downloads\monero.zip" -DestinationPath "~\Downloads\monero\"
```

### 2.2) Install monero (***as admin***):

```
Move-Item -Path "~\Downloads\monero\monero-x86_64-w64-mingw32-v*\" "C:\Program Files\monero\"
```

### 2.3) Open firewall (***as admin***):

```
New-NetFirewallRule -DisplayName "monerod (tcp)" -Direction Inbound -Action Allow -EdgeTraversalPolicy Allow -Protocol TCP -LocalPort 18080 -Program "C:\Program Files\monero\monerod.exe"
New-NetFirewallRule -DisplayName "monerod (udp)" -Direction Inbound -Action Allow -EdgeTraversalPolicy Allow -Protocol UDP -LocalPort 18080 -Program "C:\Program Files\monero\monerod.exe"
```

### 2.4) Create monero data directory:

```
New-Item -Path "D:\" -Name "Monero" -ItemType "directory"
```

### 2.5) Run monero:

```
& "C:\Program Files\monero\monerod.exe" --data-dir "D:\Monero\" --zmq-pub=tcp://127.0.0.1:18083 --no-igd --disable-dns-checkpoints --enable-dns-blocklist
```

### (optional) 2.6) Create service

```
$params = @{
  Name = "Monero"
  BinaryPathName = "C:\Program Files\monero\monerod.exe --data-dir "D:\Monero\" --zmq-pub=tcp://127.0.0.1:18083 --no-igd --disable-dns-checkpoints --enable-dns-blocklist"
  DependsOn = "NetLogon"
  DisplayName = "Monero"
  StartupType = "Delayed"
  Description = "Monero"
}
New-Service @params
```

## 3) P2Pool

### 3.1) Clone the p2pool repository

Visual Studio -> Clone a repository -> https://github.com/SChernykh/p2pool.git

### 3.2) Compile p2pool:

```
cd ~\source\repos\p2pool\
mkdir .\build\ ; cd .\build\
& "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe" -G "Visual Studio 17 2022" -A x64
```

This will generate the `~\source\repos\p2pool\build\p2pool.sln` file. Open it in Visual Studio, change from Debug to Release and build.

### 3.3) Install p2pool (***as admin***):

```
Copy-Item -Path "~\source\repos\p2pool\build\Release\" -Destination "C:\Program Files\p2pool\" -Recurse
```

### 3.4) Open firewall (***as admin***):

```
New-NetFirewallRule -DisplayName "p2pool (tcp)" -Direction Inbound -Action Allow -EdgeTraversalPolicy Allow -Protocol TCP -LocalPort 37888,37889 -Program "C:\Program Files\monero\monerod.exe"
New-NetFirewallRule -DisplayName "p2pool (udp)" -Direction Inbound -Action Allow -EdgeTraversalPolicy Allow -Protocol UDP -LocalPort 37888,37889 -Program "C:\Program Files\monero\monerod.exe"
```

### 3.5) Run p2pool:

```
& "C:\Program Files\p2pool\p2pool.exe" --host 127.0.0.1 --wallet YOUR_WALLET_ADDRESS
```

### (optional) 3.6) Create service

```
$params = @{
  Name = "P2Pool"
  BinaryPathName = "C:\Program Files\p2pool\p2pool.exe --host 127.0.0.1 --wallet YOUR_WALLET_ADDRESS"
  DependsOn = "NetLogon,Monero"
  DisplayName = "P2Pool"
  StartupType = "Delayed"
  Description = "P2Pool"
}
New-Service @params
```

## 4) xmrig

### 4.1) Clone the xmrig repositories

Visual Studio -> Clone a repository -> https://github.com/SChernykh/xmrig.git

Visual Studio -> Clone a repository -> https://github.com/xmrig/xmrig-deps.git

### 4.2) Disable donation

Edit `~\source\repos\xmrig\src\donate.h` so it looks like this:

```
#ifndef XMRIG_DONATE_H
#define XMRIG_DONATE_H

constexpr const int kDefaultDonateLevel = 0;
constexpr const int kMinimumDonateLevel = 0;

#endif // XMRIG_DONATE_H
```

### 4.3) Compile xmrig

```
cd ~\source\repos\xmrig\
mkdir .\build\ ; cd .\build\
& "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe" .. -G "Visual Studio 17 2022" -A x64 -DWITH_CN_LITE=OFF -DWITH_CN_HEAVY=OFF -DWITH_CN_PICO=OFF -DWITH_CN_FEMTO=OFF -DWITH_RANDOMX=ON -DWITH_ARGON2=OFF -DWITH_KAWPOW=OFF -DWITH_GHOSTRIDER=OFF -DWITH_HWLOC=ON -DWITH_LIBCPUID=OFF -DWITH_HTTP=OFF -DWITH_TLS=OFF -DWITH_ASM=ON -DWITH_EMBEDDED_CONFIG=OFF -DWITH_OPENCL=OFF -DWITH_CUDA=OFF -DWITH_NVML=OFF -DWITH_MSR=ON -DWITH_ADL=OFF -DWITH_PROFILING=OFF -DWITH_SSE4_1=ON -DWITH_BENCHMARK=ON -DWITH_SECURE_JIT=OFF -DWITH_DMI=ON -DWITH_DEBUG_LOG=OFF -DHWLOC_DEBUG=OFF -DCMAKE_BUILD_TYPE=Release -DXMRIG_DEPS=~\source\repos\xmrig-deps\msvc2019\x64\
& "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe" --build . --config Release
```

### 4.4) Install xmrig (***as admin***):

```
Copy-Item -Path "~\source\repos\xmrig\build\Release\" -Destination "C:\Program Files\xmrig\" -Recurse
```

### 4.5) Run xmrig:

```
& "C:\Program Files\xmrig\xmrig.exe" -o 127.0.0.1:3333
```

### (optional) 4.6) Create service

```
$params = @{
  Name = "XMRig"
  BinaryPathName = "C:\Program Files\xmrig\xmrig.exe -o 127.0.0.1:3333"
  DependsOn = "NetLogon,Monero,P2Pool"
  DisplayName = "XMRig"
  StartupType = "Delayed"
  Description = "XMRig"
}
New-Service @params
```

You can track your progress here:
- P2Pool Observer: [clear net](https://p2pool.observer/), [tor](http://p2pool2giz2r5cpqicajwoazjcxkfujxswtk3jolfk2ubilhrkqam2id.onion/)
- P2Pool Observer (Mini): [clear net](https://mini.p2pool.observer/), [tor](http://p2pmin25k4ei5bp3l6bpyoap6ogevrc35c3hcfue7zfetjpbhhshxdqd.onion/)
