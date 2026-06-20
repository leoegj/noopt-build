name: Build noopt.ko for Xiaomi 13 Fuxi

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/ylarod/ddk-min:android13-5.15-20260313
      options: --privileged

    steps:
      - name: Clone NoOpt source
        run: |
          git clone --depth=1 https://github.com/Yon8/NoOpt.git /tmp/noopt

      - name: Build noopt.ko
        run: |
          cd /tmp/noopt/kernel
          make KDIR=/workspace/kernel CONFIG_KSU=m CC=clang \
               CFLAGS_MODULE="-Wno-error" \
               -j$(nproc) 2>&1 | tail -30
          if [ ! -f noopt.ko ]; then
            echo "Build failed!"
            exit 1
          fi
          ls -la noopt.ko

      - name: Patch vermagic
        run: |
          cd /tmp/noopt/kernel
          python3 -c "
data = bytearray(open('noopt.ko','rb').read())
old = b'vermagic=5.15.202-android13-5.15.202_r00-dirty'
new = b'vermagic=5.15.178-android13-8-00021-g6f2f96be86b9-ab13729987 SMP preempt mod_unload modversions aarch64'
idx = data.find(old)
if idx >= 0:
    data[idx:idx+len(new)] = new + b'\x00'*(len(old)-len(new))
    open('noopt.ko','wb').write(data)
    print('Patched OK')
else:
    print('Pattern not found')
"
          strings noopt.ko | grep vermagic || true

      - name: Upload noopt.ko
        uses: actions/upload-artifact@v4
        with:
          name: noopt-kernel-module-xiaomi13-fuxi
          path: /tmp/noopt/kernel/noopt.ko
