# nvidia-musl

**NOTE:** This was just a small experiment to see if I could get NVIDIA drivers working on musl, real usage is _NOT_ advised and there could be possibilities of undefined behaviour due to the inherent hackiness of all this.

Some hacks to run proprietary NVIDIA drivers on a pure musl system.

* Only two symbols required by the drivers are missing from gcompat so we [patch them in](https://github.com/git-bruh/nvidia-musl/blob/master/repo/gcompat/patches/nvidia.patch). The `_IO_2_1_stdout_` symbol is just a NULL stub so if the driver tries to actually make use of it, it will crash but it doesn't seem to use it at all in my experience so this will do for now.

* `libglvnd` is required by the drivers so we build mesa with glvnd support and build `libglvnd` itself without support for `tls` (`-Dtls=disabled`) to prevent the `initial-exec TLS resolves to dynamic definition in ...` error.

* All the driver libraries like `libnvidia-eglcore.so` have to be explicitly "linked" against gcompat with `patchelf` to avoid `LD_PRELOAD`-ing gcompat (`patchelf --add-needed libgcompat.so.0 libnvidia-xyz.so`).

* X11 will not work (Neither will anything that needs `libnvidia-tls.so`) and no NVIDIA-specific X11 libs should be installed (See the `nvidia` package).

* The `repo/` directory is a KISS repository that contains packages containing the above hacks.

It should be obvious but don't report driver related bugs to anyone if you're running such a setup.
