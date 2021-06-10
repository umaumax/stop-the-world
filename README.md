# stop-the-world

`LD_PRELOAD` base library for stopping(send `SIGTSTP`) the program before starting `main()`

## for what?
* lazy `gdb` attach pid
* get pid for `pmap` to see memory mapping
* to set pid to `tasks` file of `cgroup`

## how to run
``` bash
# linux
LD_PRELOAD=./target/debug/libstop_the_world.so ls
# darwin
DYLD_FORCE_FLAT_NAMESPACE=1 DYLD_INSERT_LIBRARIES=./target/debug/libstop_the_world.dylib ls

# if your environment has no output log of background process pid, set below environment variable
STOP_THE_WORLD_PIDFILE=$(tty)

# environment variable
STOP_THE_WORLD_INIT=1 # (default is enable)
STOP_THE_WORLD_TERM=0 # (default is disable)
STOP_THE_WORLD_PIDFILE="" # (default is no output)
```

## how to send SIGCONT signal
``` bash
fg
# or
kill -SIGCONT $PID
# or
pkill -SIGCONT $PNAME
```

## FYI

* some rust program: NG(no sleep)
* some c++ program: OK
* `kill -SIGSEGV` to `sleep`: OK

``` cpp
#include <chrono>
#include <csignal>
#include <thread>

void segv_handler(int sig) {
  std::this_thread::sleep_for(std::chrono::seconds(3600 * 24));
}

namespace internal {
struct Segvstop {
  Segvstop() {
    struct sigaction sa;
    sa.sa_flags   = 0;
    sa.sa_handler = segv_handler;
    sigemptyset(&sa.sa_mask);
    sigaction(SIGSEGV, &sa, nullptr);
  }
  ~Segvstop() {}
};

static Segvstop segvstop;
}  // namespace internal
```

----

[gdb上で再現しないセグフォをgdbで追跡する \- Qiita]( https://qiita.com/s417-lama/items/daea1a029f58d71358ef )

darwin
``` bash
g++ -shared -fPIC -std=c++11 segvstop.cpp -o libsegvstop.dylib
DYLD_FORCE_FLAT_NAMESPACE=1 DYLD_INSERT_LIBRARIES=./libsegvstop.dylib XXX
```
