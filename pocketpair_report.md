Subject: Dedicated Server (Linux) — intermittent SIGSEGV inside the engine itself, no mods/third-party libraries involved

Hi,

I run a Linux dedicated Palworld server and have been chasing an intermittent crash over the past few days. I want to report it directly, because after ruling out everything on my end, the evidence points at the engine itself rather than anything in my setup.

**What's happening:**
The server occasionally crashes with a segmentation fault (SIGSEGV, process exit code 139) during normal operation — not tied to any specific player action I've been able to identify. Restarts sometimes recover cleanly and sometimes don't, requiring a manual restart.

**Why I'm confident this isn't caused by mods or my configuration:**
I run a small set of Lua mods via a community UE4SS Linux port, and I initially suspected that layer. I captured a core dump of the crash and analyzed it with gdb against the server binary. The full stack trace resolves entirely within the main game executable's own address range — none of the modding library, none of any other third-party component. Relevant frames:

```
#0  ___pthread_mutex_lock (mutex=0x1430) at ./nptl/pthread_mutex_lock.c:80
#1  0x00000000074e3313 in ?? ()
#2  0x00000000072defe0 in ?? ()
#3  0x0000000006be54f8 in ?? ()
#4  0x0000000007a1df37 in ?? ()
#5  0x0000000007b57a86 in ?? ()
#6  0x000000000a2ffcee in ?? ()
#7  0x000000000a2f717b in ?? ()
#8  0x000000000a2f5842 in ?? ()
#9  0x000000000a2f5d98 in ?? ()
#10 0x000000000a2ee7c2 in ?? ()
#11 0x000000000a2ed999 in ?? ()
#12 0x000000000a5ee618 in ?? ()
#13 0x000000000a5ece70 in ?? ()
#14 0x0000000004e27363 in ?? ()
#15 0x000000000a621bad in ?? ()
#16 0x0000000004cb7b15 in ?? ()
#17 0x000000000a556748 in ?? ()
#18 0x000000000a39f2c4 in ?? ()
#19 0x00000000043b87c3 in _start ()
```

The shipping build is stripped, so I can't name the exact function beyond frame #0 — but `mutex=0x1430` is an implausibly small value for a real mutex object, which is consistent with a corrupted or uninitialized pointer being dereferenced as a lock (i.e. a race condition somewhere in the engine's threading, not a logic bug a mod could trigger).

**Not an isolated case:** a friend who runs a separate Palworld server — vanilla, no mods, no UE4SS or any third-party tooling at all — has independently described the same kind of behavior: occasional crashes, and restarts that sometimes don't recover. That's consistent with this being a pre-existing issue in the dedicated server build itself, not something specific to my setup.

**Server details:**
- Platform: Linux dedicated server (`PalServer-Linux-Shipping`), x86_64
- Launch flags: `-publiclobby -useperfthreads -NoAsyncLoadingThread -UseMultithreadForDS -port=8211 -publicport=8211 -players=32 -rcon`



Is this a known issue on your end? Appreciate any guidance, and happy to provide anything else that would help track it down.

Thanks,
[Your name / server name]
